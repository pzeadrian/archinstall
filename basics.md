## Themes

### GTK 4
As libadwaita brokes all the themes, I use the default one, but with dark mode activated by default in all non fltapak apps.
GTK-4.0 Config (For flatpak use flatseal)

### GTK 2/3
[Colloid-GTK-Theme](https://github.com/vinceliuice/Colloid-gtk-theme)

### Qt
Use Kvantum with [Colloid-KDE](https://github.com/vinceliuice/Colloid-kde)

### Icons
[Colloid-Icon-Theme](https://github.com/vinceliuice/Colloid-icon-theme)

## Timeshift
The idea of use btrfs is to approach the snapshots, so you can use Timeshift...
```bash
yay -S timeshift # or use the aur manager that you've installed
```
Don't forget to check if cronie is enabled. Otherwise, automatic backups won't work.
```bash
sudo systemctl enable cronie
sudo systemctl start cronie
```

## Removable Media
For get the computer recognizing USB, Android phones and even NTFS disks, I recommend:
```bash
sudo pacman -S udiskie mtpfs gvfs-mtp gvfs-gphoto2
```
Configure udiskie systemd service:
- Create udiskie.service file in /etc/systemd/user and copy to it:
    ```
    [Unit]
    Description=udiskie daemon

    [Service]
    Type=simple
    ExecStart=/usr/bin/udiskie
    Restart=always

    [Install]
    WantedBy=default.target
    ```
- Run:
    ```
    systemctl enable --user udiskie
    ```

## Bluetooth
Three steps to configure bluetooth
- Step 1: Install utilities
- Step 2: Enable Bluetooth service
- Step 3: Pair with Arch Linux

Now, let's check if all is OK.
```bash
lsmod | grep btusb
sudo nano /etc/bluetooth/main.conf
```

The following command will tell you if the adapter is connected or is blocked.
```bash
sudo rfkill list
```

In the case that your adapter is blocking the connectivity, input the following command to unblock the connection.
```bash
sudo rfkill unblock bluetooth
```

And finally, enable bluetooth.
```bash
sudo systemctl start bluetooth.service
sudo systemctl enable bluetooth.service
```

## Network
For managing networks, I use [Network Manager](https://wiki.archlinux.org/title/NetworkManager) with the terminal command nmcli or nm-applet
### Trick - Slow WiFi
```bash
sudo nano /etc/modprobe.d/iwlwifi.conf
```
- Now copy that inside the file:
```bash
options iwlwifi 11n_disable=8
```
Then,
```bash
sudo nano /etc/sysctl.d/40-ipv6.conf
```
And paste that,
```bash
# Disable IPv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.nic0.disable_ipv6 = 1
net.ipv6.conf.nicN.disable_ipv6 = 1
```

## Clipboarding
### Wayland
```bash
sudo pacman -S wl-clipboard
```

### X.org
```bash
sudo pacman -S xclip
```

## Emoji Support
```bash
sudo pacman -S noto-fonts-emoji
```
Additionally, if you're using a window manager with waybar or polybar, you may need a Sans-serif font, I use:
```bash
sudo pacman -S ttf-opensans
```
