# Arch Linux Installation

## Download the Boot
I basically just followed the [installation guide](https://wiki.archlinux.org/title/installation_guide) on ArchWiki.

First, download the .iso from [ArchWiki](https://archlinux.org/download/) (ideally verify the signature) and flash it onto a USB using a tool like [rufus](https://rufus.ie/en/) or [etcher](https://etcher.balena.io/). Insert installation device into target machine and boot from USB. Will likely need to select it from the boot menu (usually accessed via `Esc`, `F2`, or `F12` key during boot up). From there follow the prompts until you are greated with a shell login.

Verify keyboard layout, language, internet connection, and time as per the [installation guide](https://wiki.archlinux.org/title/installation_guide).

## Disk Partitions
Check existing disk partitions and mount locations via the `lsblk` command.

Partition the disk using `fdisk`. Example partition reference (for UEFI with GPT):
```
Device     Size  Type
/dev/sda1  1G    EFI System
/dev/sda2  4G    Linux swap
/dev/sda3  -
```

1. `d` as many times as needed to delete all partitions
2. `n` to add first partition (start: _default_, size: `+1G`) & `t` to cahnge type to "EFI System" (partition: `1`, type: `1`)
3. `n` to add second partition (start: _default_, size: `+4G`) & `t` to change type to "swap" (partition: `2`, type: `swap`)
4. `n` to add third partition (start: _default_, size: _default_) & verify type is "Linux filesystem"
5. `v` to verify the partition table is valid
6. `w` to actually write the table to disk and exit

Format the disk partitions using `mkfs`. Following with the same example partition:

1. `mkfs.fat -F 32 /dev/sda1` to format the EFI partition as a FAT32 filesystem
2. `mkswap /dev/sda2` to format the Swap partition
3. `mkfs.ext4 /dev/sda3` to format the Linux filesystem as a EXT4 filesystem

Mount the various partitions and setup swap memory:

1. `mount /dev/sda3 /mnt` to mount the root filesystem
2. `mount --mkdir /dev/sda1 /mnt/boot` to mount the EFI system partition
3. `swapon /dev/sda2` to enable the swap volume

## Linux Setup

1. `pacstrap -K /mnt base linux linux-firmware` to install the base package, linux kernel, and firmware for common hardware
2. `genfstab -U /mnt >> /mnt/etc/fstab` to generate an fstab file and verify it is correct
3. `arch-chroot /mnt` to change root into the new system
4. `pacman -S intel-ucode grub efibootmgr` to install packages for Intel firmware updates and GRUB bootloader (note: if installed along-side other OS, also install `os-prober` package)
5. `mkdir /boot/EFI` to make directory for UEFI bootloader
6. `grub-install --target=x86_64-efi --efi-directory=/boot/ --bootloader-id=grub_uefi --recheck` to install UEFI bootloader
7. `grub-mkconfig -o /boot/grub/grub.cfg` to generate GRUB boot configuration
8. Use `psswd` to set the root password
9. `pacman -S networkmanager` to install NetowrkManager for handling internet connection
10. `echo hostname > /etc/resolv.conf` to configure the hostname (replace _hostname_ with whatever name you want)

Exit the chroot environment with `exit` and unmount all partitions with `umount -R /mnt` and ensure no partitions are "busy". Restart the machine with `reboot`, remove installation medium, and login into the new system with root account.

## Users & Graphical Interface
Create a new underprivillaged user (ex. _kameron_) to protect root access, and give it a password
```shell
pacman -S sudo
useradd -m kameron
sudo adduser kameron sudo
passwd kameron
```

Optionally, also add the user to the _sudoer_ group so that they can access root privillages (important for the main 'adminstrative' account). First install _sudo_ with `pacman -S sudo` and edit the file `/etc/sudoers` by adding the following line in the _User privilege specification_ section (replace USER with user's name):
```
USER ALL=(ALL:ALL) ALL
```

Start and enable the NetworkManager:
```shell
systemctl start NetworkManager
systemctl enable NetworkManager
```

Download plasma desktop environment for KDE (or another GUI if you prefer) with the folling (all defaults):
```shell
pacman -S plasma plasma-workspace
```

For default KDE applications:
```shell
pacman -S kde-applications
```

Enable the simple desktop display manager (SDDM) and reboot:
```shell
systemctl enable ssdm
reboot now
```

## Applications

### Steam
Enable the _multilib_ repository by uncommenting the following lines in `/etc/pacman.conf`:
```conf
[multilib]
Include = /etc/pacman.d/mirrorlist
```

