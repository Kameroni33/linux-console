# Arch Linux Installation

I basically just followed the [installation guide](https://wiki.archlinux.org/title/installation_guide) on ArchWiki.

First, download the .iso from [ArchWiki](https://archlinux.org/download/) (ideally verify the signature) and flash it onto a USB using a tool like [rufus](https://rufus.ie/en/) or [etcher](https://etcher.balena.io/). Insert installation device into target machine and boot from USB. Will likely need to select it from the boot menu (usually accessed via `Esc`, `F2`, or `F12` key during boot up). From there follow the prompts until you are greated with a shell login.

Verify keyboard layout, language, internet connection, and time as per the installation guide.

Partition the disk. Example partition reference (for UEFI with GPT):
```
Device     Size  Type
/dev/sda1  1G    EFI System
/dev/sda2  4G    Linux swap
/dev/sda3  -
```

Use `fdisk` to edit partition table:

1. `d` as many times as needed to delete all partitions
2. `n` to add first partition (start: _default_, size: `+1G`) & `t` to cahnge type to "EFI System" (partition: `1`, type: `1`)
3. `n` to add second partition (start: _default_, size: `+4G`) & `t` to change type to "swap" (partition: `2`, type: `swap`)
4. `n` to add third partition (start: _default_, size: _default_) & verify type is "Linux filesystem"
5. `v` to verify the partition table is valid
6. `w` to actually write the table to disk and exit

Format the disk partitions using `mkfs`. Following with the same example partition:

1. `mkfs.fat -F 32 /dev/sda1` to format the EFI partition as a FAT32 filesystem
2. 
