---
layout: post
title: Low-latency osu! on Linux
---

osu! on Linux is nothing new. It has been running fine under Wine for almost a decade already ([the first guide on the osu! forums](https://osu.ppy.sh/community/forums/topics/14614) goes back to 2009)! Getting it to work perfectly, to meet my top player needs, is another story though.  

## History

osu! has been running nearly flawlessly since the "OpenGL update" that happened [late 2015](https://blog.ppy.sh/post/128331961733/20150904). It was a huge update, as the osu! engine was rewritten to only support OpenGL and drop DirectX support. For some reason, the OpenGL rendering engine of the game never worked on Wine - but fortunately, that osu! update fixed it. What that meant for us Linux users, is that Wine didn't have to convert DirectX calls to OpenGL anymore!  
Since that update, osu! on Linux has been nearly painless to install and play on, compared to other games relying on Wine. Its performance, in terms of frames-per-seconds, has since been identical if not better than on Windows on most hardware.

There has always been a recuring issue though: audio latency.

## Audio Latency

To make it simple, we'll consider two main sources of audio latency: the audio server & the audio source (Wine/the game).

Usually, Linux distros nowadays are shipped with [PulseAudio](https://www.freedesktop.org/wiki/Software/PulseAudio/). It is a very capable, modern audio server providing many functionalities. However, it is also usually shipped with configuration files to meet nearly perfect stability on any kind of systems. It will meet about anyone's expectations, but ours. We are *hardcore Linux rhythm gamers*! (I honestly hope I'll never say that again.)  
That means by default, there's a very noticeable high latency (that is noticeable even on other games, without Wine).

To tackle that, players used to either use another audio server (or straight go with ALSA, as Wine doesn't support JACK anymore) or tweak their PulseAudio config to reduce latency.  
With appropriate configuration (but dozens of headaches), you could eventually get a perfect osu! setup, matching Windows' latency without PulseAudio. However, that also often meant reducing compatibility with other applications needing audio, or not being able to have osu! running alongside another audio software. Despite those downsides, I almost never tried going without PulseAudio - especially as I intended to record/stream my gameplay.  
PulseAudio is never going to be the audio server with the lowest latency by design, but it doesn't mean it can't achieve very low latency either. It also has many functionalities I appreciate as a Twitch streamer (networking, null sinks and loopbacks).

With all of that in mind, I tried optimizing latency elsewhere: inside the Wine PulseAudio driver.

## Inside winepulse.drv

It took quite some time, but I ended up with a patch to winepulse.drv that is stable and capable of providing very low latency. But before we get into results, let's have a look at the patch first. I'll comment it hunk by hunk.

```diff
@@ -69,8 +69,8 @@
     Priority_Preferred
 };
 
-static const REFERENCE_TIME MinimumPeriod = 30000;
-static const REFERENCE_TIME DefaultPeriod = 100000;
+static const REFERENCE_TIME MinimumPeriod = 1000;
+static const REFERENCE_TIME DefaultPeriod = 2000;
 
 static pa_context *pulse_ctx;
 static pa_mainloop *pulse_ml;
```

This one just edits a few constants that used to help quite a ton in previous Wine releases. In recent updates though, their effect has been reduced. But these lower values may still help.

```diff
@@ -406,9 +406,9 @@
     ss.channels = map.channels;
 
     attr.maxlength = -1;
-    attr.tlength = -1;
-    attr.minreq = attr.fragsize = pa_frame_size(&ss);
-    attr.prebuf = 0;
+    attr.minreq = -1;
+    attr.tlength = attr.fragsize = pa_usec_to_bytes(1000, &ss);
+    attr.prebuf = -1;
 
     stream = pa_stream_new(ctx, "format test stream", &ss, &map);
     if (stream)
@@ -417,9 +417,9 @@
         ret = -1;
     else if (render)
         ret = pa_stream_connect_playback(stream, NULL, &attr,
-        PA_STREAM_START_CORKED|PA_STREAM_FIX_RATE|PA_STREAM_FIX_CHANNELS|PA_STREAM_EARLY_REQUESTS, NULL, NULL);
+        PA_STREAM_START_CORKED|PA_STREAM_FIX_RATE|PA_STREAM_FIX_CHANNELS|PA_STREAM_EARLY_REQUESTS|PA_STREAM_ADJUST_LATENCY, NULL, NULL);
     else
-        ret = pa_stream_connect_record(stream, NULL, &attr, PA_STREAM_START_CORKED|PA_STREAM_FIX_RATE|PA_STREAM_FIX_CHANNELS|PA_STREAM_EARLY_REQUESTS);
+        ret = pa_stream_connect_record(stream, NULL, &attr, PA_STREAM_START_CORKED|PA_STREAM_FIX_RATE|PA_STREAM_FIX_CHANNELS|PA_STREAM_EARLY_REQUESTS|PA_STREAM_ADJUST_LATENCY);
     if (ret >= 0) {
         while (pa_mainloop_iterate(ml, 1, &ret) >= 0 &&
                 pa_stream_get_state(stream) == PA_STREAM_CREATING)
```

This one edits flags and attributes to request lower latency from PulseAudio. I just applied suggestions from the [PulseAudio LatencyControl wiki page](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/Developer/Clients/LatencyControl/) here.

```diff
@@ -1767,6 +1767,13 @@
     /* Uh oh, really low latency requested.. */
     if (duration <= 2 * period)
         period /= 2;
+    
+    const char *env = getenv("STAGING_AUDIO_DURATION");
+    if(env) {
+        int val = atoi(env);
+        duration = val;
+        printf("Set audio duration to %d (STAGING_AUDIO_DURATION).\n", val);
+    }
 
     period_bytes = pa_frame_size(&This->ss) * MulDiv(period, This->ss.rate, 10000000);
```

This is the one with the biggest impact. The function I added this slice of code in is `AudioClient_Initialize`. The name is pretty explicit, it initializes the audio client so it is likely to be somewhere we can make adjustments to improve latency. And so I did, and it works.  
To my understanding, `duration` is directly *requested* by the application, but we don't have to hold on to it. That means we can manipulate this value to make "audio slices" whatever length we want. As you may have understood by looking at the code, I made it adjustable at runtime with the `STAGING_AUDIO_DURATION` environment variable. I named it this way as the wine staging patches also use environment variables beginning with `STAGING`. Maybe this could be moved into an entry in the Windows registry, but I figured this was the most simple solution for now.  

## Results of my Progress

It is now time for the results. Testing was done with the patch I described along with a few common PulseAudio configuration adjustments (that I'll give later in the article).

My testing method to measure audio latency is the following:
- edit the `osu!.<user>.cfg` file and set `CustomFrameLimit` to an unrealistically high number (such as `10000`) to achieve the highest possible refresh rate on main menu
- start a microphone recording on Audacity
- set the system, osu! global and music volume to 100% (or whatever is high enough to hear music decently in your mic)
- set osu! effects volume to 0%
- put the microphone in between the speakers and the mouse
- pause/unpause the music and measure the audio latency by calculating the difference between when you hear the click and the music pausing/unpausing.

Let's begin with Windows 10.  
I'm using the default Microsoft drivers for my motherboard's built-in sound card with all enhancements disabled.
<div class="embed">
    <img src="/assets/articles/2018-06-16-osuLinuxAudioLatency/measure-windows.png" alt="Waveform of the latency test on Windows 10"><br>
    <i>Waveform of the latency test on Windows 10</i>
</div>
This is about **74ms** of latency! This is high. Unfortunately, the Windows audio subsystem is hardly configurable so the only way to optimize latency is probably either in osu! or the audio drivers.

We can surely do better...  
Let's see what osu! on Wine Staging 3.10 (without my patch) gives.
<div class="embed">
    <img src="/assets/articles/2018-06-16-osuLinuxAudioLatency/measure-linux.png" alt="Waveform of the latency test on Linux with unpatched winepulse.drv"><br>
    <i>Waveform of the latency test on Linux with unpatched winepulse.drv</i>
</div>
Unfortunately, despite tweaks to my PulseAudio install, we're getting **104ms** of latency. This is way higher than what we experience on Windows, and unplayable to my standards.  
The most common and recent workaround for this was to use the Wine ALSA drivers instead (which works on PulseAudio using default configuration), which somehow provide lower latency despite being theoretically less optimal.

Time for my patch to get in the way.
<div class="embed">
    <img src="/assets/articles/2018-06-16-osuLinuxAudioLatency/measure-linux-patched.png" alt="Waveform of the latency test on Linux with patched winepulse.drv"><br>
    <i>Waveform of the latency test on Linux with patched winepulse.drv</i>
</div>
Now we're talking: **15ms**! This is the result I get with my patch and `STAGING_AUDIO_DURATION` set to `5000` and the PulseAudio settings given below. I'm sure I could actually lower it down to 10ms (with more extreme PulseAudio settings; I can also set `STAGING_AUDIO_DURATION` as low as `2000`). But this is stable and I don't really need to push it down further; I'm very happy with this right now!

## Installing

This is a detailed osu! install guide for Linux users. If you already have a working osu! install, you might want to just read the ["Installing Wine"](#installing-wine) and ["Adjusting Latency"](#adjusting-latency) parts as they probably cover what you're looking for. 

**This isn't a beginner guide.** I'm too lazy to explain everything, so if you're not familiar with Linux, you should consider reading another guide/use automated scripts. You won't benefit from my patch though.

### Graphics Drivers & Desktop Environment

The NVIDIA Open-source drivers (`nouveau`) often offer bad performance. Using proprietary drivers on NVIDIA cards is encouraged. You should look up how to install them on your distribution's wiki. Ubuntu has a tab dedicated to "Additional Drivers" in its "Software & Updates" utility for this purpose.  
As far as I know, the Intel and AMD built-in drivers are fine in most cases.

If you're having performance/lag issues with the desktop environment you're using, I recommend you to switch to the very reliable Xfce (which can run with no compositor, if that's what you're looking for). No matter your choice, I recommend staying with Xorg over Wayland for the time being. Support for Wayland is not very mature yet.  
You can do that later if your current desktop environment doesn't suit your needs.

### Installing Wine

To run osu! on Linux, we need Wine. Not any kind of Wine (hah!), but Wine with our patch applied.  

#### Arch Linux Users

I created an AUR package: [`wine-osu`](https://aur.archlinux.org/packages/wine-osu/). It's latest Wine with Staging patches and my patch applied (I'll keep it up-to-date over time).  
However, as **Wine is very long to compile** (really, it is), I encourage you to use my repository instead:
- Add & sign my key:
```bash
sudo pacman-key --keyserver hkps://hkps.pool.sks-keyservers.net -r C0E7D0CDB72FBE95
sudo pacman-key --keyserver hkps://hkps.pool.sks-keyservers.net --lsign-key C0E7D0CDB72FBE95
```
- Append my repo to your `/etc/pacman.conf`:
```
[thepoon]
Server = https://archrepo.thepoon.fr
Server = https://mirrors.celianvdb.fr/archlinux/thepoon
```
- Update your local pacman database: `sudo pacman -Syu`
- Install `wine-osu`: `sudo pacman -S wine-osu`

You're now set with `wine-osu` installed in `/opt/wine-osu`!

#### Others

Other distributions users have two options.

First one is: build Wine 3.10, with [staging patches](https://github.com/wine-staging/wine-staging) and [my patch](https://aur.archlinux.org/cgit/aur.git/tree/winepulse_latency.patch?h=wine-osu) applied. However, Wine can be very complicated, long and different to build across every distributions, so I won't cover that here. You will have to figure that out by yourself.

Second option is: install Wine Staging and replace `winepulse.drv` with mine. However, this should be considered a workaround as it may not work across every Wine version, PulseAudio versions, distributions, etc. I do not recommend it, but I couldn't find an "easy enough" way to build Wine on other distributions such as Ubuntu, so... that will do for now (let me know if this breaks).  
I tested this method successfully with Ubuntu 18.04 in a VM, I can confirm audio is working but I didn't measure latency.
- Download and install Wine Staging from [WineHQ](https://wiki.winehq.org/Download). Ubuntu 18.04 is apparently still not supported, so you should execute this to add the 17.10 repo instead: `sudo apt-add-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ artful main'`
- Replace `winepulse.drv`: 
```bash
sudo wget -O /opt/wine-staging/lib/wine/winepulse.drv.so https://blog.thepoon.fr/assets/articles/2018-06-16-osuLinuxAudioLatency/32bit/winepulse.drv.so
sudo wget -O /opt/wine-staging/lib64/wine/winepulse.drv.so https://blog.thepoon.fr/assets/articles/2018-06-16-osuLinuxAudioLatency/64bit/winepulse.drv.so
```

### Setting up the `WINEPREFIX`

Very basically, the `WINEPREFIX` is a folder that will be the root of our "Windows install". No, we won't install Windows, but it will be considered the root, and we will install libraries/frameworks that osu! needs to run here.  
We don't have to create it, Wine will take care of that for us. All we need to do is set the path to it in our environment. **That also means you will need to define it every time you open a new shell and want to manipulate your osu! install and start osu!** (unless you define it in your profile, which will make it your default `WINEPREFIX`, which is pointless since it already has a default value of `~/.wine` when undefined).  
We will also set `WINEARCH` to `win32`. That will make this `WINEPREFIX` only able to execute 32-bit apps, which is fine in our case. It is recommended to make your prefix 32-bit only if you don't need 64-bit support, as this is a more stable configuration.
```bash
export WINEPREFIX=~/.wine_osu # This is the path to a hidden folder in your home folder.
export WINEARCH=win32 # Only needed when executing the first command with that WINEPREFIX

# Arch Linux/wine-osu users should uncomment next line
# to update PATH to make sure we're using the right Wine binary
#export PATH=/opt/wine-osu/bin:$PATH
```

- [Install `winetricks`](https://github.com/Winetricks/winetricks#installing). It is included in repositories of most Linux distros.
- Install .NET Framework 4.0: `winetricks dotnet40`  
  Do not install Mono or Gecko.  
  Press next at every step. If you're prompted to reboot your computer, no matter the choice it won't reboot and will be fine.  
  If the process is stuck and the terminal is printing lines ending with "retrying (60 sec)", open a new terminal, set the `WINEPREFIX` variable and execute `wineserver -k`.
- Optional: for some icons to appear, install GDI+: `winetricks gdiplus`.

### Installing osu!

- Make sure the environment variables are correct (you can use `env` to print them)
- Create an empty directory for osu!, and `cd` into it
- Download the osu!installer: `wget https://m1.ppy.sh/r/osu\!install.exe`
- Install osu!: `wine osu\!install.exe`

At this point, you should be able to play osu! already. Congrats! But before getting addicted, you might want to adjust the latency...

### Adjusting latency

First, most desktop users and especially gamers will want a kernel optimized for this kind of workflow. I recommend Ubuntu/Debian users to install the [Liquorix Kernel](https://liquorix.net/#install) and I recommend the [linux-zen Kernel](https://www.archlinux.org/packages/extra/x86_64/linux-zen/) for Arch Linux users.

Next, we will adjust security limits in `/etc/security/limits.conf` to give PulseAudio higher priority:
```bash
echo "@audio - nice -20
@audio - rtprio 99" >> /etc/security/limits.conf
```
You may replace `@audio` by your username if you're not part of the `audio` group (check your groups, with, well, the `groups` command).

Now, for audio latency, we will set a few PulseAudio settings.
```bash
mkdir -p ~/.config/pulse/daemon.conf.d/
echo "high-priority = yes
nice-level = -15

realtime-scheduling = yes
realtime-priority = 50

resample-method = speex-float-0

default-fragments = 2 # Minimum is 2
default-fragment-size-msec = 2 # You can set this to 1, but that will break OBS audio capture." > ~/.config/pulse/daemon.conf.d/better-latency.conf
```

Next, we will want to edit `/etc/pulse/default.pa`. You should find a line similar to this:
```
load-module module-udev-detect tsched=1
```
You should set the `tsched` value to 0. Time-based scheduling (`tsched=1`) guarantees a more consistent audio playback, but makes going down in latency way harder/impossible. Interrupt-based scheduling (`tsched=0`) works way better in these conditions.

You should reboot your system now to apply the newer security limits and PulseAudio settings.

Lastly, let's get my patch into action: start osu! with the lowest `STAGING_AUDIO_DURATION` value you can get osu! stable with. I recommend starting with 10000: `STAGING_AUDIO_DURATION=10000 wine osu\!.exe`

### Start Script

To tidy up your install, I recommend creating a start script, in `~/.local/bin/osu` for example:
```bash
#!/bin/sh
WINEPREFIX=~/.wine_osu
STAGING_AUDIO_DURATION=5000 # As low as you can get osu! stable with

# Arch Linux/wine-osu users should uncomment next line
# for the patch to be effective
#PATH=/opt/wine-osu/bin:$PATH

cd ~/Documents/osu! # Or wherever you installed osu! in
wine osu!.exe "$@"
```

I also recommend creating a "kill" script. osu! may freeze or not start for whatever reason, and that comes in handy sometimes. In `~/.local/bin/osukill` for example:
```bash
#!/bin/sh
WINEPREFIX=~/.wine_osu

wineserver -k
```

Make sure `$HOME/.local/bin` is in your PATH, otherwise do `echo 'PATH="$HOME/.local/bin:$PATH"' >> ~/.profile` and restart your session.  
Don't forget to make these scripts executable (with `chmod +x ~/.local/bin/osu*`) and give them a try!

### Adding osu! to your Applications

Finally, we will add osu! to your Applications menu to start it easily.

First, let's download the osu! icon: `wget -O ~/.local/share/icons/osu\!.png https://w.ppy.sh/c/c9/Logo.png`

Next, open `~/.local/share/applications/osu!.desktop` and paste:
```
[Desktop Entry]
Type=Application
Name=osu!
Icon=osu!.png
Exec=osu %u
StartupWMClass=osu!.exe
Categories=Game;
MimeType=x-scheme-handler/discord-367827983903490050;x-scheme-handler/osu;
```

### Workaround for OBS Audio Capture

OBS Studio on Linux has trouble capturing low-latency audio. This can be worked around with a null sink and a loopback which will capture audio fine:
```bash
pactl load-module module-null-sink sink_name="audiocap" sink_properties=device.description="audiocap"
pactl load-module module-loopback latency_msec=1 sink="audiocap"
```
Then, open `pavucontrol`, go into the Recording tab, set display to "All Streams" and set the loopback source to "Monitor of \<playback device>". You will need to execute these commands and do that step everytime you restart your session/PulseAudio.

Finally, in OBS set the Audio Capture device to "audiocap". OBS should capture playback audio fine now.

### Tablet Configuration, Troubleshooting & More

That's it for me! I leave the rest to [Franc[e]sco's Ultimate guide](https://osu.ppy.sh/community/forums/topics/367783&start=0). His post on the osu! forums covers osu! installation, but also has many tips such as how to fix Japanese fonts and tablet configuration.  
I also covered configuration of the XP-PEN Star G640 in a [previous article](/XPPenLinux)!

## Conclusion

To conclude, the lowest latency you can get comes down to what your hardware is capable of. The higher your CPU frequency is, the faster it can process sound, the lower latency gets. Your audio interface and its drivers also indeed play a big role.

Also, I'm interested in your results. Feel free to measure your latency on both Windows and Linux the way I did and share the results in the comment!

I'm very saddened how janky applying this patch is right now on distros other than Arch. I will look into building an osu! Snap/AppImage for every Linux distributions in my spare time!

I hope you found this post interesting, despite being very lengthy. Feel free to hop on [my Discord](https://discord.gg/ThePooN) in the fresh #osu-linux channel if you need some help, or in the comments below! Feedback is as always well appreciated.