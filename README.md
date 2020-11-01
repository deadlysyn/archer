# Opinionated Arch Install Helper

NOTE: The [official guide](https://wiki.archlinux.org/index.php/installation_guide)
is authoritative. If something here doesn't work check that. Feel free to open a
PR if something can be better. Also check [References](references). **Read the
scripts before running!**

After installing Arch a handful of times around the house I decided to take
some time and automate more of the process. The great thing about Arch
aside from having complete control is the wiki. It's a wonderful resource,
but can be overwhelming. For someone a bit ADHD/Dyslexic like me I end up
following 2^32 links and have 100 tabs open just to get through a basic
install (sincere thanks to all the doc writers, this is a personal problem
not you).

This guide attempts to condense the install doc and several docs it links
to so my "typical case" is easier to follow "top down" with as much as
possible automated. This may turn into a more full featured script that
supports additional options (PRs welcome), but as a starting
point I just attempted to semi-automate my "typical" install.

Typical for me means getting wifi enabled, installing a similar set of base
packages, adding on some more heavyweight items as needed, and making minor
tweaks based on laptop/desktop/intel/amd. I assume GPT/UEFI
partitioning, and try to minimize bloat (e.g. systemd-boot since it's OOB
vs adding GRUB, iwd/iwctl vs networkmanager and wpa_supplicant, etc.).
You might have different choices, but can easily add those at the end.
Other things like locale or timezone preferences currently require editing
the scripts (again, please read them before running -- they are just shell
for a reason).

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
  - `../scripts/install <packages...>`

## Still Manual

- Enable sshd (might not want on all machines)
- Setup SSH config/keys (private data)
- Install locally built bins
  - cw
  - lf
  - ...

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
