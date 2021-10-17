# arch-instructions

Arch
==================

Install option 1

`loadkeys uk`

Varify boot mode. If folder doesnt exist, boot mode is BIOS

`ls /sys/firmware/efi/efivars`

partition disks with fdisk

`fdisk /dev/sda`

n to create 3 partitions one for / and one for /home and /swap

p to list

w to commit

mkfs.ext4 for both / and /home

mkswap for /swap

mount root partition to /mnt

mount home partition to /mnt/home

enable swap: `swapon /dev/sda3`

install basic packages and other bits:
`pacstrap /mnt base linux linux-firmware man-db man-pages texinfo vim e2fsprogs lvm2 iputils iproute2 dhcpcd dhclient dnsmasq`

generate a new fstab:
`genfstab -U /mnt >> /mnt/etc/fstab`

chroot into new area: `arch-chroot /mnt`

set timezone:
`ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime`

generate /etc/adjtime: `hwclock --systohc`

uncomment `en_GB.UTF-8 UTF-8` in /etc/local

run `locale-gen`

create /etc/locale.conf:

`echo "LANG=en_GB.UTF-8" > /etc/locale.conf`

`echo "KEYMAP=uk" > /etc/vconsole.conf`

`echo "arch-host" > /etc/hostname`

add the following to /etc/hosts
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	arch-host
```

set the root passwd:

`passwd`

install and configure a bootloader:
```
pacman -S grub
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

* something here needs setting to show boot messages at startup. Otherwise you dont see much. I think its the `quite` option. In the grub prompt, select e, then delete the quiet parameter, then f10 to continue boot.

set up CPU microcode updates:
```
pacman -S intel-ucode
grub-mkconfig -o /boot/grub/grub.cfg
```

`ctrl+d`

`umount -R /mnt`

remove ISO from drive

`reboot`

=================================

post install 

Setup network

`pacman -S dhclient`

base network manager is systemd-networkd (`networkctl`)

```
systemctl enable systemd-networkd.service --now
systemctl enable systemd-resolved --now
```

create /etc/systemd/network/20-wired.network:

```
[Match]
Name=enp0s3

[Network]
DHCP=yes
```

`systemctl restart systemd-networkd.service`

=====DESKTOP=============

install the xrog group package
`pacman -S xorg`

Check GPU:
$ lspci -v | grep -A1 -e VGA -e 3D

install the driver. correct driver from https://archlinux.org/groups/x86_64/xorg-drivers/. or search :

`pacman -Ss xf86-video`

In my case vmware:

`pacman -S xf86-video-vmware`

Install a desktop environment, in my case KDE Plasma

pacman -S plasma 

to have a better time in kde install this stuff. kde-applications adds some bloat, but something in there is needed for plasma discover package manager to work properly (potentially `appstream` and `appstream-qt`):

`pacman -S konsole packagekit packagekit-qt5 kde-applications`

install a display manager. Im my case LightDM:

pacman -S lightdm

configure the greeter:

pacman -S xorg-server lightdm-gtk-greeter

systemctl enable lightdm

to test install `xorg-server-xephyr` then run `lightdm --test-mode --debug`

reboot

There is a KDE bug that will not let virtualbox resolution go higher than 800x600. Change the virtual box display adapter to VBoxSVGA instead of VMSVGA

