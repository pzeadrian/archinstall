Ok, let's go, first of all, install Arch...

# Arch installation
### WARNING
- This guide is aimed to install arch linux but using btrfs filesystem, ensuring compatibility with [Timeshift](https://github.com/linuxmint/timeshift) tool.

First of all, assuming that you already created the Arch bootable USB, let's check the boot mode. So, run:
```bash
ls /sys/firmware/efi/efivars
```
If it returns values, the boot mode is UEFI, else, it's BIOS (UEFI is better and this guide uses UEFI, if you have BIOS, I recommend you to change to UEFI, just with a little search in uncle Google).

Now, connect to the internet, an easy way is to use iwctl:
```bash
iwctl
```
Then, 
```bash
station wlan0 scan
station wlan0 get-networks
```
and finally,
```bash
station wlan0 connect "Wifi Name" # It will prompt to input the password automatically
```

## Disk Partitioning
For me, the ideal partition scheme just have boot(/boot), root(/), home(/home) and swap partitions. The space that you give to those partitions remains in yourself.
For partitioning your disk(s), I recommend you to use cfdisk, because, in my opinion, is the easiest way to do it from terminal. So, first run:
```bash
lsblk # For checking your disk(s) name
```

```bash
cfdisk /dev/"Your disk name" # If you plan to distribute your partitions across multiple disks, just run this command changing the disk name
```

## Format partitions
- Currently, I use two types of filesystems, btrfs (for snapshots with Timeshift) on a machine and the old and trusty EXT4 in another one, here's how I partition my disk according to these filesystems.

<details>
<summary><b>1. EXT4 </b></summary>
<br />

This is so simple, but effective...
For boot:
```bash
mkfs.fat -F 32 -n boot /dev/"Boot Partition"
```

For swap:
```bash
mkswap -L swap /dev/"Swap Partition"
```

For root:
```bash
mkfs.ext4 -L root /dev/"Root Partition"
```

For home:
```bash
mkfs.ext4 -L home /dev/"Home Partition"
```

## Mounting Partitions
```bash
mount /dev/disk/by-label/root /mnt
mkdir -p /mnt/home
mkdir -p /mny/boot
mount /dev/disk/by-label/home /mnt/home
mount /dev/disk/by-label/boot /mnt/boot
swapon /dev/disk/by-label/swap
```

</details>

<details>
    <summary><b>2. BTRFS </b></summary>
    <br />
    Before formatting, run another "lsblk" for being sure all is OK.
    <br />
    For boot partition:
    <br />
    ```bash
        mkfs.fat -F 32 -n boot /dev/"Boot Partition"
    ```
    <br />
    For root partition:
    <br />
    ```bash
        mkfs.btrfs -f -L arch /dev/"Root Partition"
    ```
    <br />
    For home partition: --> Skip this step if you don't want a home dedicated partition, because in btrfs, you can always create a home subvolume in root partition <--
    <br />
    ```bash
        mkfs.btrfs -f -L home /dev/"Home Partition"
    ```
    <br />
    For swap partition:
    <br />
    ```bash
        mkswap -L swap /dev/"Swap Partition"
    ```

## Creating btrfs subvolumes
    <details>
        <summary><b> You created Home partition></b></summary>
        <br/>
         ```bash
            mount -t btrfs /dev/"Root partition" /mnt; cd /mnt 
            btrfs subvolume create @
            cd /
            umount /mnt
            mount -t btrfs /dev/"Home partition" /mnt; cd /mnt
            btrfs subvolume create @home
            cd /
            umount /mnt 
        ```
    </details>
    <details>
        <summary><b>You didn't create Home partition</b></summary>
        <br />
        ```bash
            mount -t btrfs /dev/"Root partition" /mnt; cd /mnt
            btrfs subvolume create @
            btrfs subvolume create @home
            cd /
            umount /mnt
        ```
    </details>

## Mounting Partitions
    <details>
        <summary><b> You created Home partition></b></summary>
        <br />
        ```bash
            mount -t btrfs -o subvol=@ /dev/"Root Partition" /mnt
            mkdir -p /mnt/home
            mount -t btrfs -o subvol=@home /dev/"Home Partition" /mnt/home
        ```
    </details>
    <details>
        <summary><b>You didn't create Home partition</b></summary>
        <br />
        ```bash
            mount -t btrfs -o subvol=@ /dev/"Root Partition" /mnt
            mkdir -p /mnt/home
            mount -t btrfs -o subvol=@home /dev/"Root Partition" /mnt/home
        ```
    </details>
    Then,
    ```bash
    mkdir -p /mnt/boot/efi
    mount /dev/"Boot Partition" /mnt/boot/efi
    ```
    ```bash 
    swapon /dev/"Swap Partition"
    ```
</details>

## Installing Linux kernel and basic packages
```bash
pacstrap -i /mnt base base-devel linux-zen linux-firmware btrfs-progs
```

## Configuring the system basics
### Fstab
```bash 
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot
```bash 
arch-chroot /mnt
```

### Timezone
```bash 
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
```bash
hwclock --systohc
```

### Localization
```bash
pacman -S neovim nano
```
Edit /etc/locale.gen and uncomment en_US.UTF-8 and other needed locales. Generate the locales by running:
```bash
locale-gen
```

Create the /etc/locale.conf file, and set the LANG variable accordingly:
```bash
LANG=en_US.UTF-8 # or the language that you want
```

If you set the console keyboard layout, make the changes persistent in /etc/vconsole.conf:
```bash
KEYMAP=us # or your keymap
```

### Network
Create the /etc/hostname file:
```bash
"myhostname"
```

Edit the /etc/hosts file by adding:
```bash
127.0.0.1 localhost
127.0.1.1 "myhostname".localdomain  "myhostname"
```

### Root password
```bash
passwd
```
And, that's it, you've installed Arch Linux, at least until what the official guide shows you... Now, you could do:

```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

Now you can install a bootloader and test it "safely", this is how to do it on
modern hardware,
[assuming you've mounted the efi partition on /boot](https://wiki.archlinux.org/index.php/Installation_guide#Example_layouts):

<br />
For bootloader, I used to install GRUB, but now that all my machines are UEFI compatible, I prefer using systemd-boot, it seems faster for me.

<details>
<summary><b> Long life to GRUB </b></summary>
<br />

```bash
pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot
os-prober
grub-mkconfig -o /boot/grub/grub.cfg
```

</details>

<details>
<summary><b> I have UEFI, and I'm so cool </b></summary>
```bash
bootctl install
```

In /boot/loader/loader.conf, add:
```bash
default  arch.conf
timeout  5
console-mode max
editor   no
```

In /boot/loader/entries/ create arch.conf file and add:
```bash
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
