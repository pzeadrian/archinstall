Ok, let's go, first of all, install Arch...

# Arch installation

### WARNING

-   This guide is aimed to install arch linux but using btrfs filesystem, ensuring compatibility with [Timeshift](https://github.com/linuxmint/timeshift) tool.

First of all, assuming that you already created the Arch bootable USB, let's check the boot mode. So, run:

```sh
ls /sys/firmware/efi/efivars
```

If it returns values, the boot mode is UEFI, else, it's BIOS (UEFI is better and this guide uses UEFI, if you have BIOS, I recommend you to change to UEFI, just with a little search in uncle Google).

Now, connect to the internet, an easy way is to use iwctl:

```sh
iwctl
```

Then,

```sh
station wlan0 scan
station wlan0 get-networks
```

and finally,

```sh
station wlan0 connect "Wifi Name" # It will prompt to input the password automatically
```

## Disk Partitioning

For me, the ideal partition scheme just have boot(/boot), root(/), home(/home) and swap partitions. The space that you give to those partitions remains in yourself.
For partitioning your disk(s), I recommend you to use cfdisk, because, in my opinion, is the easiest way to do it from terminal. So, first run:

```sh
lsblk # For checking your disk(s) name
```

```sh
cfdisk /dev/"Your disk name" # If you plan to distribute your partitions across multiple disks, just run this command changing the disk name
```

## Formatting and mounting partitions

-   Currently, I use two types of filesystems, btrfs (for snapshots with Timeshift) on a machine and the old and trusty EXT4 in another one, but, with LVM2, here's how I partition my disk according to these filesystems.

<details>
<summary><b>1. ext4</b></summary>
<br/>
<h3>Format partitions</h3>

> In this case, you should do some research in order to learn to partition using LVM2

This is so simple, but effective:

> For boot:

```sh
mkfs.fat -F 32 -n boot /dev/"Boot Partition"
```

> For swap:

```sh
mkswap -L swap /dev/"lvm name"/"Swap Partition"
```

> For root:

```sh
mkfs.ext4 -L root /dev/"lvm name"/"Root Partition"
```

> For home:

```sh
mkfs.ext4 -L home /dev/"lvm name"/"Home Partition"
```

<h3>Mount partitions</h3>

```sh
mount /dev/"lvm name"/root /mnt
mkdir -p /mnt/home
mkdir -p /mny/boot
mount /dev/"lvm name"/home /mnt/home
mount /dev/disk/by-label/boot /mnt/boot
swapon /dev/"lvm name"/swap
```

</details>

<details>
<summary><b>2. btrfs (for snapshots with Timeshift)</b></summary>
<br />
<h3>Format and mount partitions</h3>

> For boot partition:

```sh
mkfs.fat -F 32 -n boot /dev/"Boot Partition"
```

> For root partition:

```sh
mkfs.btrfs -f -L arch /dev/"Root Partition"
```

> For home partition: --> Skip this step if you don't want a home dedicated partition, because in btrfs, you can always create a home subvolume in root partition <--

```sh
mkfs.btrfs -f -L home /dev/"Home Partition"
```

> For swap partition:

```sh
mkswap -L swap /dev/"Swap Partition"
```

<details>
<summary><b>You created home partition</b></summary>
<br />

> Create btrfs subvolumes
```sh
mount -t btrfs /dev/"Root partition" /mnt; cd /mnt 
btrfs subvolume create @
cd /
umount /mnt
mount -t btrfs /dev/"Home partition" /mnt; cd /mnt
btrfs subvolume create @home
cd /
umount /mnt 
```

> Mount partitions
```sh
mount -t btrfs -o subvol=@ /dev/"Root Partition" /mnt
mkdir -p /mnt/home
mount -t btrfs -o subvol=@home /dev/"Home Partition" /mnt/home
```

</details>

<details>
<summary><b>You didn't create home partition</b></summary>
<br />

> Create btrfs subvolumes
```sh
mount -t btrfs /dev/"Root partition" /mnt; cd /mnt
btrfs subvolume create @
btrfs subvolume create @home
cd /
umount /mnt
```

> Mount partitions
```sh
mount -t btrfs -o subvol=@ /dev/"Root Partition" /mnt
mkdir -p /mnt/home
mount -t btrfs -o subvol=@home /dev/"Root Partition" /mnt/home
```

</details>

> And for boot and swap
```sh
mkdir -p /mnt/boot/efi
mount /dev/"Boot Partition" /mnt/boot/efi
swapon /dev/"Swap Partition"
```

</details>

## Installing Linux kernel and basic packages
```sh
pacstrap -i /mnt base base-devel linux-zen linux-firmware # If you plan to use btrfs, then add btrfs-progs, or add lvm2 if you plan to use LVM2
```

## Configuring the system basics
### Fstab
```sh 
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot
```sh 
arch-chroot /mnt
```

### Timezone
```sh 
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
```sh
hwclock --systohc
```

### Localization
```sh
pacman -S neovim nano
```
Edit /etc/locale.gen and uncomment en_US.UTF-8 and other needed locales. Generate the locales by running:
```sh
locale-gen
```

Create the /etc/locale.conf file, and set the LANG variable accordingly:
```sh
LANG=en_US.UTF-8 # or the language that you want
```

If you set the console keyboard layout, make the changes persistent in /etc/vconsole.conf:
```sh
KEYMAP=us # or your keymap
```

### Network
Create the /etc/hostname file:
```sh
"myhostname"
```

Edit the /etc/hosts file by adding:
```sh
127.0.0.1 localhost
127.0.1.1 "myhostname".localdomain  "myhostname"
```

### Root password
```sh
passwd
```

And, that's it, you've installed Arch Linux, at least until what the official guide shows you... Now, you could do:
```sh
pacman -S networkmanager
systemctl enable NetworkManager
```

> In case you used LVM2, you have to add it to mkinitcpio.conf, HOOKS line, and run:
```sh
mkinitcpio -P
```

Now you can install a bootloader, I used to install GRUB, but now that all my machines are UEFI compatible, I prefer using systemd-boot, it seems faster for me.

<details>
<summary><b> Long life to GRUB </b></summary>
<br />

> It's ok, so, do
```sh
pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot
os-prober
grub-mkconfig -o /boot/grub/grub.cfg
```

</details>

<details>
<summary><b> I have UEFI, and I'm so cool </b></summary>
<br />

> Ok, just do
```sh
bootctl install
```

> In /boot/loader/loader.conf, add:
```sh
default  arch.conf
timeout  5
console-mode max
editor   no
```

> In /boot/loader/entries/ create arch.conf file and add:
```sh
## This is just an example config file.
## Please edit the paths and kernel parameters according to your system.

title   Arch Linux
linux   /vmlinuz-linux-zen
initrd  /initramfs-linux-zen.img
options root="LABEL=root" rw quiet splash loglevel=0
```

</details>

Now you can create your user:

```bash
useradd -m username
passwd username
usermod -aG wheel,video,audio,storage,network username
```

In order to have root privileges we need sudo:

```bash
pacman -S sudo
```

Edit **/etc/sudoers** with nano or vim by uncommenting this line:

```bash
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```

Now you can reboot:

```bash
# Exit out of ISO image, unmount it and remove it
exit
umount -R /mnt
reboot
```

After logging in, your internet should be working just fine, but that's only if
your computer is plugged in. If you're on a laptop with no Ethernet ports, you
might have used **[iwctl](https://wiki.archlinux.org/index.php/Iwd#iwctl)**
during installation, but that program is not available anymore unless you have
installed it explicitly. However, we've installed
**[NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager)**,
so no problem, this is how you connect to a wireless LAN with this software:

```bash
# List all available networks
nmcli device wifi list
# Connect to your network
nmcli device wifi connect YOUR_SSID password YOUR_PASSWORD
```

Check [this page](https://wiki.archlinux.org/index.php/NetworkManager#nmcli_examples)
for other options provided by *nmcli*.

```bash
nmcli device wifi connect *ssid* password *password*
```
