# Opinionated Arch Install Helper

- [Why](#why)
- [What](#what)
- [Thought Process](#thought-process)
- [Choices](#choices)
- [Install](#install)
  - [Bootstrapping](#bootstrapping)
  - [Finalization](#finalization)
  - [Configuration](#configuration)
  - [Manual Steps](#manual-steps)
  - [TODO](#todo)
- [References](#references)

## Why

NOTE: The [official guide](https://wiki.archlinux.org/index.php/installation_guide)
is authoritative. If something here doesn't work, fall back to that. Feel free
to open a PR if something can be better. Also check [References](references).
**Read the scripts before running!**

After installing Arch a handful of times I decided to automate more of
the process. Aside from having complete control, the great thing about Arch
is the wiki. It's a wonderful resource, but can be overwhelming. I end up following
2^32 links and have 100 tabs open just to get through a basic install
(sincere thanks to all doc writers, this is a personal problem vs any fault
of yours).

## What

This project attempts to condense the Arch install guide and several pages
it links to so my typical install is easier to follow top down.
This could become more full featured and support additional
options (PRs welcome), but as a starting point I just semi-automated my
typical install. Depending on your needs, this might save you time
as well or serve as a starting point for automating your own preferences.

## Thought Process

My goal was consistency (without needing to remember what utilities to
track down on each install) and a "minimal but not too minimal" install.
I also wanted to be able to run the same install process on an old
Thinkpad or a modern desktop and get reasonably adjusted defaults,
whether Intel- or AMD-based.

I wasn't aiming for "the most minimal possible", since the ultimate
goal was consistent usability. Some choices could undoubtedly be replaced
with slimmer alternatives or eliminated entirely. For example, the base install
 includes things like an image viewer and PDF reader. I found
having these available helpful in normal life. To minimize impact,
I chose the leanest alternatives I could find with reasonable functionality
([sxiv](https://github.com/muennich/sxiv) and
[zathura](https://pwmt.org/projects/zathura) in these specific cases).
You might have different chocies, and these can be swapped out easily.

For larger things I only wanted some places (desktop vs laptop), I split
out separate manifests so they can be selectively included. Again you 
are free to have different choices, and even I find myself moving more
of "base" into "bloat" over time. Examples in this category are
[LibreOffice](https://www.libreoffice.org), [gimp](https://www.gimp.org),
and even [picom](https://wiki.archlinux.org/index.php/Picom). While I
typically pull these in, they are often hundreds of megabytes with
a plethora of dependencies so I wanted it to be easy to exclude them.

## Choices

For any who care, these are some choices I've made and reasoning behind them.
These are strong opinions weakly held, so I reserve the right to change my mind.

Wired is nice and fast, but modern wifi is fast enough for me and useful
while roaming. I've tried to ensure wifi works on first boot. To adhere to
a minimalist approach, I went with [iwd](https://wiki.archlinux.org/index.php/Iwd)
(vs networkmanager, wpa_supplicant and dhclient). I thought that would
be a hard choice, but it is well documented and capable. It integrates
nicely with
[systemd-networkd](https://wiki.archlinux.org/index.php/Systemd-networkd) and
[systemd-resolved](https://wiki.archlinux.org/index.php/Systemd-resolved).
This also meant dropping [nm-applet](https://aur.archlinux.org/packages/network-manager-applet-git)
which served me well for years but is a bit bloated. `iwctl` can
be used to manage wireless connections. I'm working on a dmenu-based
network picker for more WM-integrated UX, without the need to
run a system tray or pull in another dependency (dmenu is also used as
a run menu and general prompter).

Speaking of [systemd](https://wiki.archlinux.org/index.php/Systemd), I
was never a fan. I didn't object because it wasn't a solid init system,
or because SysV init was good enough (I grew up on SysV init, but everything
needs to evolve over time). My objection was softer, in that systemd's
"everything and the kitchen sink" approach clearly violates
[the UNIX philosophy](https://en.wikipedia.org/wiki/Unix_philosophy).

For those who argue "you aren't forced to use anything" -- true, but
if you want to avoid systemd entirely it limits your distro choices
(I wanted [Arch](https://archlinux.org) vs
[Artix](https://artixlinux.org) or [Void](https://voidlinux.org)),
requires extra work to disable, and includes bloat you don't need
if swapping components. As a result, since Arch uses systemd, I decided
to go all in and drink the Koolaid. This includes using
[systemd-boot](https://wiki.archlinux.org/index.php/Systemd-boot)
rather than GRUB, systemd-resolved vs resolveconf, and similar choices.
Each of the alternatives are valid, but require pulling in additional
dependencies and effectively working around vs with Arch's choices as a distro.
In the end this path was smooth thanks to Arch's documentation, but
you may need to adjust based on your requirements.

In terms of "desktop environments", whatever that means, I've again tried to
eliminate bloat. To me (we all have opinions), these "environments" represent
a systemd-class problem (lots of choices made for me which become irritating after
prolonged use but are so entangled it's difficult to use only the parts you need).
This repo enables a slim window manager ([bspwm](https://github.com/baskerville/bspwm))
vs full blown desktop environment. Lightweight tools including
[sxhkd](https://github.com/baskerville/sxhkd) and [polybar](https://github.com/polybar/polybar)
are pulled in to round out the experience. Ultimately, this implies
a preference for a tiling workflow with VIM-like navitation. If you follow
this end to end, `Super-F1` provides a list of default keybindings.

While no organization is perfect, philosophically I align more with Firefox
than other popular browsers. I've tried a handful of minimal browsers
including [Midori](https://astian.org/en/midori-browser),
[qutebrowser](https://www.qutebrowser.org),
and [surf](https://surf.suckless.org)
but been unimpressed with performance and features. If you haven't,
give these a spin and decide for yourself.

I've come to the conclusion the modern web is bloat, and requires a bloated
browser to be of much use. While Firefox fulfills that need for me, my
day job also requires specific plugins which only work with Chromium based
browsers. You certainly don't want to run Chrome and provide deep
analytics to a profiteering giant, but there are alternatives including
[ungoogled-chromium](https://aur.archlinux.org/packages/ungoogled-chromium)
and [Brave](https://aur.archlinux.org/packages/brave). I initially balked at Brave
due to monetization aspects, but you can disable everything unsavory
(for now) and the way it natively supports HTTPS redirects and ad blocking
eliminate the need for some plugins.

In the end I decided to
[benchmark a handful of popular browsers](#tbd)
on my hardware and let the numbers decide. I encourage you to do
the same. My number-driven choices are embedded in default configuration
when using this install process, but can be easily swapped out.
Regardless of your choice, change your default search to something like
[Startpage](https://startpage.com),
[Qwant](https://qwant.com), or
[Searx])(https://searx.me).

Over the years I noticed myself using more and more Electron apps. Spotify,
Slack, VS Code (yes I went there)... I didn't think much about it,
mostly accepting the bloat. As part of this project I decided to rethink
my choices. While not perfect, and accompanied by a learning curve, I've
been moving to CLI-based replacements. This includes
[spotify-tui](https://github.com/Rigellute/spotify-tui),
[wee-slack](https://github.com/wee-slack/wee-slack), going back to VIM
(or [vscodium](https://aur.archlinux.org/packages/vscodium-bin) if you must)
and even swapping things like
[pavucontrol](https://www.archlinux.org/packages/extra/x86_64/pavucontrol)
for [pulsemixer])(https://github.com/GeorgeFilipkin/pulsemixer).
After initial adjustment you end up with the same or more functionality
that works equally well on hardware of any age.

## Bootstrapping

- [Download ISO](https://www.archlinux.org/download) and write to USB
  - `lsblk` to determine right `/dev/sdX`
  - `dd if=/path/to/file.iso of=/dev/sdX status=progress`
- Reboot from USB (enable in BIOS, ensure UEFI is on, secure boot is off)
- Configure wifi network
  - `rfkill unblock wifi` (not always needed)
  - `iwctl`
  - `device list`
  - `station wlan0 scan`
  - `station wlan0 get-networks`
  - `station wlan0 connect <ssid>`
  - config is saved in `/var/lib/iwd/<ssid>.psk` (we'll use it later)
  - `timedatectl set-ntp true`
- Verify boot mode
  - `ls /sys/firmware/efi/efivars`
  - Missing == legacy BIOS (stick to install guide vs these notes)
  - Present == UEFI (keep going)
- Partition disk
  - `fdisk -l` to see devices
  - `fdisk /dev/XXX` then erase all partitions and recreate
  - `g` (create new empty GPT partition table)
  - Create partitions
    - 512MB EFI boot partition (type 1, EFI System)
    - Use rest of disk for `/` (type 24, Linux x86-64 root)
  - Format
    - `mkfs.fat -F32 /dev/X` (EFI)
    - `mkfs.ext4 /dev/Y` (root)
    - Mount root on `/mnt`
    - `mkdir /mnt/boot` then mount EFI partition on `/mnt/boot`
- Install base packages
  - `pacstrap /mnt base base-devel linux linux-firmware vim man-db man-pages iwd efibootmgr`
- Maintain wifi config
  - `mkdir /mnt/var/lib/iwd`
  - `chmod 700 /mnt/var/lib/iwd`
  - `cp /var/lib/iwd/<ssid>.psk /mnt/var/lib/iwd`
- Fix fstab
  - `genfstab -U /mnt >> /mnt/etc/fstab`
- Bootstrap
  - `arch-chroot /mnt`
  - `curl -s https://raw.githubusercontent.com/deadlysyn/arch-helper/main/bootstrap | bash -s <manifests...>`
  - manifests are ABS package lists ([files with abs suffix](https://github.com/deadlysyn/arch-helper/tree/main/manifests))
- Reboot (make sure boot manager works)

## Finalization

Login as root, then configure a local user and get AUR packages installed.

- `curl -s https://raw.githubusercontent.com/deadlysyn/arch-helper/main/usersetup | bash -s <username>`
- Logout of root and back in as local user
- `mkdir ~/src;cd ~/src;git clone https://github.com/deadlysyn/arch-helper.git`
- `cd ~/src/dotfiles/scripts;./yaystrap <manifests...>`
  - manifests are AUR package lists ([files with aur suffix](https://github.com/deadlysyn/arch-helper/tree/main/manifests))

## Configuration

Now we've got a base install with some useful pacakges, working network, and
a local user. Time to tweak our environment and start X. The install script
will symlink configs for bspwm, sxhkd, polybar, etc. into home.

- While logged in as local user...
  - `git clone git@github.com:deadlysyn/dotfiles.git`
  - `cd dotfiles/linux`
  - `../install <packages...>`

## Manual Steps

These items are currently left as manual since they can vary from
machine to machine or user to user.

- [UFW](https://wiki.archlinux.org/index.php/Uncomplicated_Firewall) is enabled but nothing allowed. `ufw allow 22/tcp`, etc as needed.
- Enable sshd if you want remote SSH connectivity. Only allow key-based auth.
- Setup SSH config/keys (private data)
- Browser plugins (mostly handled by sync)
- Install locally built tools (I've had better luck with local builds for these)
  - [awscli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)
  - [cw](https://github.com/lucagrulla/cw)
- spotifyd/sptify-tui are installed but not configured or enabled. Edit `~/.config/spotifyd/spotifyd.conf` and `systemctl --user enable spotifyd`.

## TODO

- Consider condensing ABS entries using groups and meta packages
- Prune unnecessary xorg components
- Better automation of yay installs

## References

- https://wiki.archlinux.org/index.php/installation_guide
- https://wiki.archlinux.org/index.php/General_recommendations
- https://wiki.archlinux.org/index.php/Users_and_groups
- https://wiki.archlinux.org/index.php/Iwd
- https://wiki.archlinux.org/index.php/EFI_system_partition
- https://wiki.archlinux.org/index.php/Systemd-boot
- https://www.freedesktop.org/software/systemd/man/systemd-resolved.service.html
- https://wiki.archlinux.org/index.php/GnuPG#Key_servers
