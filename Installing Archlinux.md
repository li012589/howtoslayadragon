### Installing Archlinux

#### 0.Setting up Wifi connection

first, find out what your interface is called, run

```bash
ip -a
```

There should be another interface beside `lo`, call it `<interface>`

to see all wifi access point around you, run

```bash
ip link set <interface> up
```

Then do the scan

```bash
iwlist interface scan | less
```

Then, bring the interface down, by

```bash
ip link set interface down
```

**The easy way**

use`wifi-menu`, and it's done

**The hard way(if you are linking to a hidden wifi)**

use `netctl`

move example config

```bash
cp /etc/netctl/examples/wireless-wpa /etc/netctl/<config name>
nano /etc/netctl/your_profile
```

edit it like this:

```bash
Description="Your Description"
Interface=wlp58s0
Connection=wireless
IP=dhcp
Priority=10
Security=wpa
ESSID="<SSID Name>"
Key="<PASSPHRASE>"
Hidden=yes #<-- key to link to a hidden wifi
```

Check your connection by `ping`

```bash
ping www.baidu.com
```

[1]. https://michaelheap.com/connecting-to-a-hidden-network-on-arch-linux/

[2]. http://www.linuxandubuntu.com/home/how-to-setup-a-wifi-in-arch-linux-using-terminal

[3]. https://wiki.archlinux.org/index.php/Wireless_network_configuration#Temporary_internet_access

#### 0.1 update system clock

```bash
timedatectl set-ntp true
timedatectl status
```

#### 1 Partition

to see your current partition

```
fdisk -l
```

**The easy way**

use `cfdisk`, you will get a graphic interface

**The hard way**

use `fdisk`

```bash
fdisk /dev/nvme0n1
```

use `n` to create a partition,  and follow the instructions to create a primary partition.

Format the partition

```bash
mkfs.ext4 /dev/nvme0n1pX
```

mount the partition

```bash
mount /dev/nvme0n1pX /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot # mount the fei partition
```

[1]. https://itsfoss.com/install-arch-linux/

[2]. https://www.ostechnix.com/install-arch-linux-latest-version/

#### 2 install the system

configure pacman source list

edit `/etc/pacman.d/mirrorlist` place mirror closest to you at the top position.

Then, run

```bash
pacstrap /mnt base base-devel
genfstab -U /mnt >> /mnt/etc/fstab
```

#### 3 configure locale

```bash
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```

edit `\etc\locale.gen` uncomment the locale configuration you want, then

```bash
locale-gen
```

```bash
nano /etc/locale.conf
```

enter this 

```bash
LANG=en_US.UTF-8
```

```bash
nano /etc/hostname
```

enter this file your host name

```
nano /etc/hosts
```

enter this 

```bash
127.0.0.1	localhost
::1		localhost
```

set password 

```bash
passwd
```

#### 4 install grub

```bash
pacman -S grub efibootmgr os-prober ntfs-3g
```

Run

```bash
fdisk -l
```

to find out where your efi partition is. Then mount it at `\efi`

```bash
#mkdir /efi
#mount /dev/nvme0n1p1 /efi
# TODO generate fstab according to this
```

Then install grub:

```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB /dev/nvme0n1
```

Then generate the main configuration file

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

[1]. https://wiki.archlinux.org/index.php/GRUB#Installation_2

[2]. https://wiki.archlinux.org/index.php/EFI_system_partition#Mount_the_partition



#### 5. install wifi

```bash
pacman -S wpa_supplicant netctl
```

Reboot into new system `C+d` then `umount -R /mnt` `reboot`

#### 6.dual boot

```bash
os-prober
grub-mkconfig -o /boot/grub/grub.cfg
```

[1]. https://superuser.com/questions/1280177/unknown-device-type-using-grub-probe

#### 7.add new user

```bash
useradd --create-home <name>
```

#### 8. install xfce and lightdm

```bash
pacman -S xfce4 xfce4-goodies lightdm lightdm-gtk-greeter
sudo systemctl enable lightdm.service
```

#### 9. install Chinese fonts

```bash
adobe-source-han-sans-cn-fonts

```

#### 10. create swap file

#### 11. map suspend and hibernation to hybrid-sleep

```bash
#/etc/systemd/sleep.conf
[Sleep]
# suspend=hybrid-sleep
SuspendMode=suspend
SuspendState=disk
# hibernate=hybrid-sleep
HibernateMode=suspend

```

