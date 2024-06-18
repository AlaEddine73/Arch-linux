# Arch Linux Installation Guide

#1 to use one of the largest fonts suitable : setfont ter-132b

#2 The connection may be verified with ping : ping archlinux.org
#note: you can write "ip address show" also to checking connection if your ip don't show up:
#List IP addresses: ip address show.
#Add an IP address to an interface: ip address add address/prefix_len broadcast + dev interface.
#Delete an IP address from an interface: ip address del address/prefix_len dev interface.
#Delete all addresses matching a criteria, e.g. of a specific interface: ip address flush dev interface.
#Routing table.
#List IPv4 routes: ip route show.
#List IPv6 routes: ip -6 route show.
#Add a route: ip route add PREFIX via address dev interface.
#Delete a route: ip route del PREFIX via address dev interface.

#3 Partitioning Our Disk:
#.1 The lsblk command, short for "list block devices" : lsblk.
#.2 The fdisk linux command to disk management usage guide: fdisk "/dev/partition":
  #n = for create new one.
  #add number the partition.
  #add storge do you like +1G.
  #p for show the partition.
  #t for type the partition like t enter number of partition and 44 is lvm type.
  #w for write and exit.
#.3 The cfdisk is a linux partition editor it's similar to fdisk with user interface: cfdisk "/dev/partition".
#note: you need to create three partition 1G for boot, 1G for efi, and what you want for lvm .

#4 Formatting Our Partitions:
#.1 mkfs.fat -F32 /dev/partition1 (1G). #boot partition .
#.2 mkfs.ext4 /dev/partition2 (1G). #efi partition.

#5 Encrypted Partition.
#.1 cryptsetup luksFormat /dev/partition. #lvm partition.
#.2 write capital YES.
#.3 enter passphrase for partition.
#.4 verify passphrase.

#6 Configuring LVM:
#.1 cryptsetup open --type luks /dev/partition3 lvm.
#.2 enter passphrase.
#.3 pvcreate /dev/mapper/lvm
#.4 vgcreate volgroup0 /dev/mapper/lvm
#.5 lvcreate -L 30GB volgroup0 -n lv_root
#.6 lvcreate -L what you want volgroup0 -n lv_hoom
#.7 vgdisplay
#.8 modprobe dm_mod
#.9 vgscan
#.10 vgchange -ay
#.11 mkfs.ext4 /dev/volgroup0/lv_root
#.12 mkfs.ext4 /dev/volgroup0/lv_home
#.13 mount /dev/volgroup0/lv_root /mnt
#.14 mkdir /mnt/boot
#.15 mount /dev/partition2 /mnt/boot
#.16 mkdir /mnt/home
#.17 mount /dev/volgroub0/lv_home /mnt/home

#7 install base:
#.1 pacstrap -i /mnt base

#8 Generating The Fstab file:
#.1 genfstab -U -p /mnt >> /mnt/etc/fstab

#9 Using arch-chroot To Finish Our Installation:
#.1 arch-chroot /mnt

#10 Setting Up Users:
#.1 passwd
#.2 useradd -m -g usesrs -G wheel the_name
#.3 passwd the_name

#11 Installing Additional Packages
#.1 pacman -S base-devel dosfstools grub efibootmgr gnome gnome-tweaks lvm2 mtools nano 
#networkmanager openssh os-prober sudo ......

#12 Enabling ssh:
#.1 systemctl enable sshd

#13 Installing A Linux Kernel:
#.1 pacman -S linux linux-headers linux-lts linux-lts-headers
#.2 pacman -S linux-firmware

#14 Installing A Gpu Driver:
#.1 lspci
#.2 pacman -S mesa
#.3 pacman -S nvidia nvidia-utils nvidia-lts

#15 Generating Ram Disk(s) For Our Kernel(s):
#.1 nano /etc/mkinitcpio.conf
#note: go to hooks line and go between block and filesystems and write "encrypt lvm_number"
#.2 mkinitcpio -p linux 
#.3 mkinitcpio -p linux-lts

#16 Configuring The Locale, GRUB, etc:
#.1 nano /etc/locale.gen
#note: delete #.
#.2 locale-gen.
#.3 nano /etc/default/grub
#note: go to Grub_cmdline_linux_default= "loglevel=3 "cryptdevice=/dev/partition3:volgroup0" quiet"
#.4 mkdir /boot/EFI
#.5 mount /dev/partition1 /boot/EFI
#.6 grub-install  --target=86_64-efi --bootloader-id=grub_uefi --recheck
#.7 cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
#.8 grub-mkconfig -o /boot/grub/grub.cfg
#.9 systemctl enable gdm
#.10 systemclt enable NetworkManager
#.11 exit
#.12 umount -a
#.13 reboot
