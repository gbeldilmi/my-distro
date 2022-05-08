# Install Garuda

## Partitions

Partition | Mount point | Size | File system
--- | --- | --- | ---
/dev/sda1 | /boot/efi | 512Mo | FAT32
/dev/sda2 | / | Remaining space | BTRFS
/dev/sda3 | --- | RAM size | Linux swap
/dev/sdb1 (optional) | /home | The whole device | BTRFS

## Post-install

Update the system with `garuda-update`.

With the `Setup Assistant`, install :
- Printer, Scanner, and Samba Support
- additional Garuda wallpapers
- additional KDE components ana applications
- BlackArch repo + settings

## Install packages

Install all packages from the [package list](package-list.txt) with `pacman -S [packages]`.
