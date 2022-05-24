# Install Archlinux

## Set the keyboard layout

```sh
loadkeys fr
```

## Connect to the internet

Connect to a wifi network if necessary with `iwctl`.

```sh
timedatectl set-ntp true
```

## Partitions

### BIOS

Partition | Mount point | Size
--- | --- | ---
/dev/sda1 (bootable) | /boot | 512Mo
/dev/sda2 | --- | RAM size
/dev/sda3 | / | Remaining space
/dev/sdb1 (optional) | /home | The whole device

```sh
fdisk -l # Results ending in rom, loop or airoot may be ignored

cfdisk /dev/sda
# cfdisk /dev/sdb

mkfs.ext4 /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
# mkfs.ext4 /dev/sdb1

mount /dev/sda3 /mnt
mkdir /mnt/{boot,home}
mount /dev/sda1 /mnt/boot
# mount /dev/sdb1 /mnt/home
```

### UEFI

Code | Partition | Mount point | Size | Description
--- | --- | --- | --- | ---
ef00 | /dev/sda1 | /boot/efi | 512Mo | EFI System
8200 | /dev/sda2 | --- | RAM size | Linux swap
8300 | /dev/sda3 | / | Remaining space | Linux filesystem
8300 | /dev/sdb1 (optional) | /home | The whole device | Linux filesystem

```sh
fdisk -l # Results ending in rom, loop or airoot may be ignored

cgdisk /dev/sda
# cgdisk /dev/sdb

mkfs.fat -F32 /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
# mkfs.ext4 /dev/sdb1

mount /dev/sda3 /mnt
mkdir /mnt/{boot{,/efi},home}
mount /dev/sda1 /mnt/boot/efi
# mount /dev/sdb1 /mnt/home
```

## Install essential packages

```sh
pacstrap /mnt base base-devel linux linux-firmware intel-ucode grub zip unzip p7zip vim mtools dosfstools lsb-release ntfs-3g exfat-utils bash-completion man-db man-pages texinfo # efibootmgr (for UEFI)
```

## Configure the system

```sh
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt

ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc --utc

vim /etc/vconsole.conf # KEYMAP=fr-latin9

vim /etc/hostname # hostname
```

## Install GRUB

### BIOS

```sh
grub-install --no-floppy --recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

### UEFI

```sh
mount | grep efivars &> /dev/null || mount -t efivarfs efivarfs /sys/firmware/efi/efivars
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --recheck
mkdir /boot/efi/EFI/boot
cp /boot/efi/EFI/grub/grubx64.efi /boot/efi/EFI/boot/bootx64.efi
grub-mkconfig -o /boot/grub/grub.cfg
```

## Install packages and enable services

```sh
pacman -Syy networkmanager syslog-ng cronie ntp openssh

vim /etc/systemd/journald.conf # #ForwardToSyslog=no -> ForwardToSyslog=yes
vim /etc/ssh/sshd_config # AllowUsers user

systemctl enable NetworkManager
systemctl enable syslog-ng@default
systemctl enable cronie
systemctl enable ntpd
systemctl enable sshd
```

Install all packages from the [package list](package-list.txt) with `pacman -S [packages]`.

## Create user's profile

```sh
passwd root

visudo # #Uncomment to allow members of group wheel to execute any command
useradd -m -g wheel -c 'user' -s /bin/bash user
passwd user
```

## Update initramfs and finish installation

```sh
mkinitcpio -P

exit
umount -R /mnt
shutdown now
```
