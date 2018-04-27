---
layout: post
title: Getting the XP-PEN Star G640 to work on Linux
---

Few weeks ago, I noticed I was getting "input jumps" on the tablet I was using, the GAOMON S56K.
I've had been using this tablet since November~December 2016, when GAOMON contacted me to send me a free sample in exchange for a review (no matter the opinion I give, of course!). It was so great that I sticked with it, until issues arose.

## What issues?

For the context, I noticed them during a regular tournament weekend. I was very unhappy about my performance - I was shitmissing everywhere. On every map, on random notes, I would misaim. I ended up of course, ragequitting. :D  

<div class="embed">
	<blockquote class="twitter-tweet" data-lang="fr"><p lang="en" dir="ltr">That was too much.<br>I was at a peak, I&#39;m not anymore and on top of that I randomly shitmiss, just like in OWC again. Lost the two most important matches of this weekend.<br>Looks like next weekend is gonna be a disaster. I&#39;m going downhill.<a href="https://t.co/NCG4bNzCiN">https://t.co/NCG4bNzCiN</a></p>&mdash; ThePooN (@hugodenizart1) <a href="https://twitter.com/hugodenizart1/status/985556715226779649?ref_src=twsrc%5Etfw">15 avril 2018</a></blockquote>
	<blockquote class="twitter-tweet" data-conversation="none" data-lang="fr"><p lang="en" dir="ltr">I&#39;m so done and terribly disgusted by myself today...<br>Sorry to all my teammates first for not being able to show up for entire matches due to unfortunate schedules, second for my performance for which it wasn&#39;t even worth to even show up.</p>&mdash; ThePooN (@hugodenizart1) <a href="https://twitter.com/hugodenizart1/status/985557448189792256?ref_src=twsrc%5Etfw">15 avril 2018</a></blockquote>
	<blockquote class="twitter-tweet" data-conversation="none" data-lang="fr"><p lang="en" dir="ltr">I don&#39;t understand what&#39;s wrong. I read and play everything, the same way as before. It just doesn&#39;t work. Randomly missing notes everywhere because of my aim which became inaccurate as hell. I&#39;m so confused.</p>&mdash; ThePooN (@hugodenizart1) <a href="https://twitter.com/hugodenizart1/status/985558042283511808?ref_src=twsrc%5Etfw">15 avril 2018</a></blockquote>
	<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

About an hour after stopping both the game and the stream, I was investigating why I would shitmiss. I wasn't feeling bad, I was healthy, and actually motivated to play and win. My consistency usually drops when I'm unmotivated, but that wasn't the case here. That's what led me to think I wasn't being the issue.

So I started going deeper. I analyzed clips again, and I was indeed misaiming, but it seemed just like how anyone would misaim, nothing involving an issue with the tablet stood out at the time.  
I couldn't find any issue anywhere, I was getting crazy. And as I was getting crazy, I ended up using a ruler to "draw" straight lines on my tablet. Cursor was going straight... but if you would go slower, you would indeed notice a blatant skip. At the same point on the tablet, while drawing the line, the cursor would stop moving and start moving again after what seems to be a "dead" portion of the tablet.

At this point, I had found the potential reason of all my shitmisses. I was so relived.  
<div class="embed">
	<blockquote class="twitter-tweet" data-cards="hidden" data-lang="fr"><p lang="en" dir="ltr">You know what, I figured it out. It&#39;s the tablet. Come on stream, I&#39;ll explain everything at 19:30 (in 12mn from this tweet).<br>After that I&#39;ll unbox the Huion H430P I&#39;ve had lying for a while and start writing my review. <a href="https://t.co/BrLbkr1G9B">https://t.co/BrLbkr1G9B</a> <a href="https://t.co/kTUJuiAnVq">https://t.co/kTUJuiAnVq</a></p>&mdash; ThePooN (@hugodenizart1) <a href="https://twitter.com/hugodenizart1/status/985567770770333696?ref_src=twsrc%5Etfw">15 avril 2018</a></blockquote>  
	<i>before you ask, I'll get to that HUION tablet in another article ;)</i>
</div>

I wanted to show off the issue on stream, it sadly didn't go according to plan and I could only reproduce it once, and it wasn't even that blatant. So here I'll plug in a clip of KairoKepp, who is having the exact same issue - at a much more insane amplitude though, ahah!  
<div class="embed">
	<iframe src="https://clips.twitch.tv/embed?clip=DullAverageKathyRedCoat&autoplay=false&tt_medium=clips_embed" width="640" height="360" frameborder="0" scrolling="no" allowfullscreen="true"></iframe>
</div>

That obviously doesn't tell if that was the actual reason of my shitmisses of course - but at least I know I shouldn't trust that tablet anymore. That's progress.

So I ended up picking the XP-PEN Star G640.

## Why this tablet?

I may write a review about this tablet in the future, so I'll keep it short. Pros:
- Perfect size
- Battery-less, very light pen
- No hardware smoothing

Sounds good, right?  
Con:
- Doesn't work out-of-the-box on Linux

AAAAARRRRRRRRRRRGH. That's a hard one. That one hits hard.  
The only two tablets that are deemed good for osu! and work on Linux that I knew were the HUION 420 (aka. osu!tablet, which I used until I got the GAOMON) and the GAOMON S56K.  
I didn't trust GAOMON anymore, and I couldn't go back to the osu!tablet: it became too small. :(
There's also Wacom, but their Windows drivers are bloated, have software smoothing and some of their tablets even have hardware smoothing. There are workarounds, but sorry, having to use workarounds on officially supported platforms sound absurd to me. Most of their recommended models for osu! also are discontinued and expensive on the secondary market. So I don't wanna go with Wacom neither.

Eventually, I took the decision to go with the XP-Pen Star G640. Hawku, developer of [custom tablet drivers for osu! for Windows](https://github.com/hawku/TabletDriver) could confirm at the time that he could get the tablet to work on Ubuntu out-of-the-box, with some issues though. Sadly, I also encountered them on my way. But until I fixed it, playing on Windows with Hawku's drivers was still an amazing experience! Thanks to him for making the life of many tablet players simpler.

## The issue, out-of-the-box

This is gonna be tough to explain. A video hardly illustrates this, and so does text.  
A graphic tablet is an *absolute pointing device*. That's because no matter where your system cursor currently is, when you point your pen somewhere on the tablet, your cursor will always move to the exact same area. The tablet tells the system "pen is at coordinates (150,300)" for example.  
On the opposite, we have computer mouses. They are *relative pointing device*. When you move your mouse, it won't tell the system the coordinates, it will just tell by how much it moved.

Here, it seems the Linux kernel identifies my tablet as... both. At least that's how it behaves, I guess?  
For your information, here's the output of `dmesg` (kernel messages) after plugging in the tablet:
```
[63259.004025] usb 1-3: new full-speed USB device number 7 using xhci_hcd
[63259.241053] usb 1-3: New USB device found, idVendor=28bd, idProduct=0094
[63259.241055] usb 1-3: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[63259.241056] usb 1-3: Product: STAR G640
[63259.241057] usb 1-3: Manufacturer: XP-PEN
[63259.251307] input: XP-PEN STAR G640 as /devices/pci0000:00/0000:00:01.1/0000:01:00.0/usb1/1-3/1-3:1.0/0003:28BD:0094.0010/input/input32
[63259.251391] hid-generic 0003:28BD:0094.0010: input,hidraw2: USB HID v1.00 Mouse [XP-PEN STAR G640] on usb-0000:01:00.0-3/input0
[63259.257987] input: XP-PEN STAR G640 as /devices/pci0000:00/0000:00:01.1/0000:01:00.0/usb1/1-3/1-3:1.1/0003:28BD:0094.0011/input/input33
[63259.314161] hid-generic 0003:28BD:0094.0011: input,hiddev1,hidraw3: USB HID v1.00 Keyboard [XP-PEN STAR G640] on usb-0000:01:00.0-3/input1
[63259.317890] hid-generic 0003:28BD:0094.0012: hiddev2,hidraw4: USB HID v1.00 Device [XP-PEN STAR G640] on usb-0000:01:00.0-3/input2
```

It says `Mouse`. I can't recall what displayed for the GAOMON S56K and HUION 420, but I don't think it said mouse. For some reason I can't even get them to work anymore, it is probably because of all the configuration edits I've done in order to try to get the XP-PEN to work.

On the desktop, the area is calibrated wrong, and the positioning is relative. When I click however, the cursor teleports and that seems like absolute positioning.  
I would say it's a bug in the HID implementation in the tablet itself, but I might be wrong. There's likely a small possible patch that could be applied on the Linux kernel to fix that, as otherwise it works just fine with evdev and libinput.

## Now, how did I fix it?

Shit, that was a long intro. This is my first blog article, so I don't really know the amount of details you guys expect. What I know for sure though, is that I repeated this story too many times on stream, and I can now link you all to this article instead!!! How sweet. ;)

Enough talk, let's get into how to get this tablet properly working on Linux now.

When a tablet doesn't have support in the kernel, the project you would look for nowadays is [DIGImend](https://digimend.github.io/). It sadly doesn't support XP-PEN tablets.  
Despite that, their site still has an article on setting up tablets with the **Wizardpen** driver. It has built-in support for some tablets the DIGImend driver doesn't support, and it happens to have some luck with other tablets as well. Sounds good to try, and so I did.

I started following [the Wizardpen guide on DIGImend's website](https://digimend.github.io/support/howto/drivers/wizardpen/#custom-tablet-conf) to get it running, and it was relatively easy : all I did was installing the [xf86-input-wizardpen package from AUR](https://aur.archlinux.org/packages/xf86-input-wizardpen/) and binding my input device (the tablet) to the driver. That was just a matter of adding this config file in `/etc/X11/xorg.conf.d/52-tablet.conf`:
```
Section "InputClass"
	Identifier "Tablet on WizardPen"
	#MatchIsTablet "on"
	MatchProduct "XP-PEN STAR G640"
	MatchDevicePath "/dev/input/event*"
	Driver "wizardpen"
	# Apply custom Options below.
	Option "TopX" "0"
	Option "TopY" "0"
	Option "BottomX" "21845"
	Option "BottomY" "18255"
EndSection
```
The 4 options at the bottom correspond to my area. Full area on this is 32768 * 32768 units, so I just reduced those numbers proportionally to the tablet active area to get my own area on the top-left.  
Here's how I obtained those values (quick maths™):
```
Active area I want (Huion 420): 101.6*56.6mm
Full area (XP-PEN Star G640): 152.4*101.6mm

Width = 101.6/152.4 = 2/3 * 32768 = ~21845 units
Height = 56.6/101.6 = 0,557086614173 * 32768 = ~18255 units
```
Then, I had to restart my session to apply the changes.  
When my KDE session closed and my screen was supposed to go back to the login screen, I was greeted by this instead:

<div class="embed">
	<video src="/assets/articles/2018-04-27-XPPenLinux/Xorg-bootloop.mp4" controls preload="metadata"></video>
</div>

This is Xorg bootlooping. When I unplug the tablet, everything works back again as usual. When I replug it, Xorg instantly crashes.  
After disabling my login manager by SSH (I hadn't figured out unplugging it was enough yet...), I tried starting X with startx, and was greeted with this error:
```
/usr/lib/xorg-server/Xorg: symbol lookup error: /usr/lib/xorg/modules/input/wizardpen_drv.so: undefined symbol: RemoveEnabledDevice
xinit: giving up
xinit: unable to connect to X server: Connection refused
xinit: server error
Couldn't get a file descriptor referring to the console
```
The code was outdated. I have near to no experience with C, I have zero experience with Xorg code and drivers. All I was left with was Google and my brain.  
Typing the error on Google, partially or entirely, returned no relevant results - when it found one. That means... everyone gave up on Wizardpen?  
I had lost hope in getting this driver to work at this moment, but since I couldn't find anything else to help me, I eventually continued and tried to fix it.

The wizardpen developers wouldn't have let that error in when they released the last update, so it must have broken with an update, likely from Xorg.  
So, I had to fix it myself. I found repositories of some other drivers, and noticed they were using `xf86RemoveEnabledDevice(local)` instead of `RemoveEnabledDevice(local->fd)`. So I did patch the drivers and replaced all occurences of `RemoveEnabledDevice`, and it worked. Xorg stopped bootlooping, I could log in, my tablet was working. I was relieved!

Now, to be clear, I am still not a Xorg's internals specialist. Maybe this isn't the right, or the best way to fix it, but it works. I still consider this a workaround until it works out-of-the-box, anyway. Also do keep in mind that a Xorg driver doesn't enable it to work under Wayland, so I'm stuck with Xorg now (which I don't mind at all personally).

To conclude, I updated the [xf86-input-wizardpen AUR package](https://aur.archlinux.org/packages/xf86-input-wizardpen/) with [my patch](https://aur.archlinux.org/cgit/aur.git/tree/xf86RemoveEnabledDevice.patch?h=xf86-input-wizardpen). For the people not using Arch Linux, compiling and installing Wizardpen should be as simple, as executing this in a terminal:
```
wget https://launchpad.net/wizardpen/trunk/0.8/+download/xorg-input-wizardpen-0.8.1.tar.bz2
tar xf xorg-input-wizardpen-0.8.1.tar.bz2
cd xorg-input-wizardpen-0.8.1
curl https://aur.archlinux.org/cgit/aur.git/plain/xf86RemoveEnabledDevice.patch?h=xf86-input-wizardpen | patch -Np0
./autogen.sh --prefix=/usr --with-xorg-conf-dir=/etc/X11/xorg.conf.d
make
sudo make install
```

If your tablet isn't officially supported by the Wizardpen drivers, don't forget to add the Xorg config file and edit it accordingly (the name refers to the one listed in the output of `xinput --list`).

## Conclusion

This was a long article, perhaps too long, compared to the relatively simple solution in the end. Hope you enjoyed it anyway, please leave me your feedback in the comments below! As this is my first blog post, it would be very appreciated :D  
If you're having issues getting your tablet to work with Wizardpen, it being the XP-PEN Star G640 or not, feel free to leave a comment as well, so I can help.