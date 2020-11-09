# Opinionated Arch Install Helper

- [Why](#why)
- [What](#what)
- [Install](#install)
  - [Bootstrap](#bootstrap)
  - [Post Install](#post-install)
  - [Customize](#customize)
  - [Manual Steps](#manual-steps)
- [TODO](#todo)
- [References](#references)

## Why

NOTE: The [official guide](https://wiki.archlinux.org/index.php/installation_guide)
is authoritative. If something doesn't work, fall back to that. Feel free
to open a PR if something can be better. Also check [References](references).
**Read scripts before running!**

After installing Arch a handful of times I decided to automate more of
the process. Aside from having complete control, the great thing about Arch
is the wiki. It's a wonderful resource, but can be overwhelming. I end up following
2^32 links and have 100 tabs open just to get through a basic install
(sincere thanks to all doc writers, this is a personal problem vs any fault
of yours).

## What

This project attempts to condense the official guide so my typical install
is faster. This could become more full featured and support additional
options (PRs welcome), but as a starting point I semi-automated my
personal preferences. Depending on your needs, this might save you time
or serve as a starting point for your own automation.
See [CHOICES.md](https://github.com/deadlysyn/archer/blob/main/CHOICES.md)
for thought process and opinions.

## Install

This is still in infancy, with the need for a lot of refinement, but
in current form it greatly reduces total install time for me and results
in 548 (expect this to be further trimmed) packages consuming ~8.4GB on
disk using only 147MB of RAM on boot.

### Bootstrap

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
  - `curl -s https://raw.githubusercontent.com/deadlysyn/archer/main/bootstrap | bash -s <hostname> <manifests...>`
  - manifests are ABS package lists ([files with abs suffix](https://github.com/deadlysyn/archer/tree/main/manifests))
- Reboot (make sure boot manager works; if not reboot from USB and repair)

### Post Install

Login as root, then configure a local user and get AUR packages installed.

- `curl -s https://raw.githubusercontent.com/deadlysyn/archer/main/usersetup | bash -s <username>`
- Logout of root and back in as local user
- `mkdir ~/src;cd ~/src;git clone https://github.com/deadlysyn/archer.git`
- `cd ~/src/archer;./yaystrap <manifests...>`
  - manifests are AUR package lists ([files with aur suffix](https://github.com/deadlysyn/archer/tree/main/manifests))

### Customize

Now we've got a base install with some useful pacakges, working network, and
a local user. Time to tweak our environment and start X. The install script
will symlink configs for bspwm, sxhkd, polybar, etc. into home.

- While logged in as local user...
  - `git clone git@github.com:deadlysyn/dotfiles.git`
  - `cd dotfiles/linux`
  - `../install <packages...>`
- `startx` and add desired manual tuning

### Manual Steps

These items are currently left as manual since they can vary from
machine to machine or user to user.

- Pair bluetooth devices
  - `systemctl enable bluetooth`
  - `systemctl start bluetooth`
  - `bluetoothctl`
  - `power on`
  - `scan on`
  - `pair XX:YY:ZZ...`
- Run `lxappearance` to select theme, fonts, etc.
- Tweak font settings for machine
  - Link `/etc/fonts/conf.avail/xx-*.conf` into `~/.config/fontconfig/conf.d`
  - Varies by screen (e.g. RGB vs BGR)
- [UFW](https://wiki.archlinux.org/index.php/Uncomplicated_Firewall) is enabled but nothing allowed; open ports as needed.
  - `ufw allow 22/tcp` (ssh)
  - `ufw allow 24800/tcp` (synergy)
- Drop Synergy license in `~/.config/Synergy/license` if this is a server
- Enable sshd if you want remote SSH connectivity.
  - Default Arch sshd_config allows key auth and disables root password logins
  - `systemctl enable sshd`
  - `systemctl start sshd`
- Setup SSH config/keys (copy from another host or backup)
- Browser plugins (sign-in to sync, adjust settings as needed)
- If spotifyd/spotify-tui are installed...
  - Edit `~/.config/spotifyd/spotifyd.conf`
  - `systemctl --user enable spotifyd`
- Install locally built tools (I've had better luck with local builds for these)
  - [awscli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)
    - Setup `~/.aws/*`
    - `aws-vault add <profile>`
  - [cw](https://github.com/lucagrulla/cw)
  - [docker-credential-helper](https://github.com/docker/docker-credential-helpers)

## TODO

- Better organize manifests
- Fix and better integrate tmux config
- Better automation of yay installs
- Move all manual compilations to packages (create or fix packages where necessary)
- Pull in / tune laptop tools (power management, backlight oddities)

## References

- https://wiki.archlinux.org/index.php/installation_guide
- https://wiki.archlinux.org/index.php/General_recommendations
- https://wiki.archlinux.org/index.php/Users_and_groups
- https://wiki.archlinux.org/index.php/Iwd
- https://wiki.archlinux.org/index.php/EFI_system_partition
- https://wiki.archlinux.org/index.php/Systemd-boot
- https://www.freedesktop.org/software/systemd/man/systemd-resolved.service.html
- https://wiki.archlinux.org/index.php/GnuPG#Key_servers

