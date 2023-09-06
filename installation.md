Ok, let's go, first of all, install Arch...
# Arch installation
### WARNING
- This guide is aimed to install arch linux but using btrfs filesystem, ensuring compatibility with [Timeshift](https://github.com/linuxmint/timeshift) tool.

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

## Format partitions
- Currently, I use two types of filesystems, btrfs (for snapshots with Timeshift) on a machine and the old and trusty EXT4 in another one, here's how I partition my disk according to these filesystems.

<details>
<summary><b>1. ext4</b></summary>
<br/>

This is so simple, but effective:
> For boot:
```sh
mkfs.fat -F 32 -n boot /dev/"Boot Partition"
```

> For swap:
```sh
mkswap -L swap /dev/"Swap Partition"
```

> For root:
```sh
mkfs.ext4 -L root /dev/"Root Partition"
```

> For home:
```sh
mkfs.ext4 -L home /dev/"Home Partition"
```

</details>
