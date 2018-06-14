---
layout: post
title: Installing an AUR Package on Arch Linux
---

Arch Linux is a very rich distribution. There are plenty of packages in the official repositories that will suit most needs, however sometimes it may lack of something. But most of the time, there's a solution in the AUR!

## What is AUR?

The [Arch Linux wiki](https://wiki.archlinux.org/index.php/Arch_User_Repository) explains it better than I am able to:
<blockquote>
The Arch User Repository (AUR) is a community-driven repository for Arch users. It contains package descriptions (PKGBUILDs) that allow you to compile a package from source with makepkg and then install it via pacman. The AUR was created to organize and share new packages from the community [...].
</blockquote>

Basically, every package on Arch Linux is packaged using `PKGBUILD` files, on AUR or not. It is a file that enables `makepkg` to build a package. It contains instructions and scripts to achieve this, including everything from download to install. The official repositories contain the most popular packages while the AUR is more "niche". But that's not the only difference.

## Differences between Official Repositories and the AUR

In the official repositories, every package is pre-built. A server or someone has done that for you before. When you install a package from the repos, it doesn't contain source files. If you execute `pacman -Syu`, your system will upgrade these packages.

In AUR however, the only stored files are the `PKGBUILD` files (aside a few others that the `PKGBUILD` file may need such as patches). That means when you want to install a package from AUR, you'll have to build it.  
Sometimes, it will just package a pre-built binary provided by the program author (such as `visual-studio-code-bin`). This kind of AUR package is usually suffixed by `-bin`.  
But most of the time, it will download the source code itself and build it.  
Also, some AUR packages may provide a development branch of the program you're looking for. They usually are suffixed by `-git`, such as `obs-studio-git`. These packages may be useful if you need a bleeding-edge feature, or a too fresh bugfix that didn't get into a regular release yet.

## How to Install AUR Packages?

First, I recommend you to run this to ensure you have all the requirements to build packages:
```
sudo pacman -S base-devel wget git
```

There are 2 methods, the manual one and the more fancy one, using a CLI tool: `yaourt`. But to get to install `yaourt`, you'll have to use the manual one first. So let's dive into that :)

### Manual

I recommend doing this in a temporary directory, such as `/tmp`, as once the package is built you won't need the source files anymore. You could go further and create a subdirectory to make sure there's no conflict:
```bash
mkdir -p /tmp/aur-build # Creates the directory if it doesn't exist
cd /tmp/aur-build
```

First, we need to decide on the package we want to install. `sublime-text-dev` (providing Sublime Text 3) is one of the most popular AUR packages, so let's build it. As a first step, we will make our package name a variable to make the install process easier:
```bash
export pkg="sublime-text-dev"
```

Now, let's download the tarball and uncompress it:
```bash
wget "https://aur.archlinux.org/cgit/aur.git/snapshot/$pkg.tar.gz" # Download a snapshot of the package source files
tar xvf "$pkg.tar.gz" # Decompress it
```

Finally, get into the directory and build the package:
```bash
cd "$pkg"
makepkg -si # -s stands for build, -i stands for install
```

If all goes well, it should prompt you for your root password at the end to install the package. That one should have been quick to build, because all it did is repackaging a pre-built binary (as Sublime Text isn't open-source).

You could sum all these steps with this simple script, that you could place at `/usr/local/bin/aurinstall` for example:
```bash
#!/bin/sh
mkdir -p /tmp/aur-build # Creates the directory if it doesn't exist
cd /tmp/aur-build

pkg="$@"

wget "https://aur.archlinux.org/cgit/aur.git/snapshot/$pkg.tar.gz" # Download a snapshot of the package source files
tar xvf "$pkg.tar.gz" # Decompress it

cd "$pkg"
makepkg -si # -s stands for build, -i stands for install
```

Don't forget to make it executable with `chmod +x /usr/local/bin/aurinstall`. After that, you can just type `aurinstall sublime-text-dev` to install Sublime Text, or any other package from AUR, given you know its name.

### Yaourt

Yaourt is a CLI tool that enables you to search for packages from both official repos and AUR and simplify every step. But I figured a small video will give you a better idea than a wall of text:

<div class="embed">
	<video src="/assets/articles/2018-06-14-InstallingAURPackage/yaourt.mp4" controls preload="metadata"></video>
</div>

But before using yaourt comes installing yaourt. Luckily, it's an AUR package, so we should be able to install it with our previous script, right?

```
==> Making package: yaourt 1.9-1 (Thu Jun 14 11:43:30 2018)
==> Checking runtime dependencies...
==> Installing missing dependencies...
error: target not found: package-query>=1.8
==> ERROR: 'pacman' failed to install missing dependencies.
```

That is underwhelming. `yaourt` needs a dependency that is not installed, and not on the official repositories neither, consequently the install fails. Luckily, it's an AUR package and this one doesn't depend on any other AUR package. So we can just execute `aurinstall package-query`, and finally `aurinstall yaourt`.  

Turns out that if you use `yaourt` from now on, you won't ever face that issue again, as yaourt supports lookup from the AUR.

`yaourt` can actually fully replace `pacman`: it supports all of its commands in their original syntax, and it can even do more, such as `yaourt --aur -Syu`: that will update the remote database, and propose you to upgrade both official packages and AUR packages, something `pacman` doesn't handle. I encourage you to keep your AUR packages up-to-date with this command. AUR packages aren't officially supported and have more chance of breaking something on your system, so paying more attention to them is essential.

In addition to "replacing `pacman`", `yaourt` enables you to look up for a package, as I did in the first step of my video. It is as simple as `yaourt <keywords>`.

I encourage you to read the `yaourt` manual to learn more: `man yaourt`

## Conclusion

AUR is one of Arch Linux's strengths: it makes virtually nearly all software available on Arch, for you to easily install with easy-to-use tools.  
You can learn more about AUR on [its wiki page](https://wiki.archlinux.org/index.php/Arch_User_Repository) and even submit your own package whenever you can't find one that suits your needs! Also, feel free to take a look at [my own packages](https://aur.archlinux.org/packages/?SeB=m&K=ThePooN) ;-)