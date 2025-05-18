# LiveUSB preparation
```bash
gpg -v archlinux-*.iso.sig

dd bs=4M if=archlinux-*.iso of=/dev/sdX status=progress oflag=sync
```

# Boot LiveUSB

## After live-usb booted

### load locale:
```bash
loadkeys pl
```

### (Optional) set font:
```bash
setfont ter-120b
```

### Verify the boot mode
```bash
cat /sys/firmware/efi/fw_platform_size # should return 64
```

### Connect to Internet:

#### Connect to WiFi:
```bash
iwctl
```

```
device list
device wlan0 set-property Powered on

adapter list
adapter phy0 set-property Powered on

station wlan0 scan
station wlan0 get-networks

station wlan0 connect <SSID>
station wlan0 connect-hidden <SSID>
```


#### Chech connection
```bash
ip addr
ping archlinux.org
```

### Update the system clock
```bash
timedatectl list-timezones
timedatectl set-timezone Europe/Warsaw
timedatectl
```

## Prepare partitions
- [Systemd (recommended for Secure boot)](partition-systemd.md)
- [GRUB: Boot partition not encrypted](partition-grub-unencrypted.md)
- [GRUB: Boot partition encrypted](partition-grub-encrypted.md) - a little bit more secure but adds +20s to boot time because of the way GRUB works

## Install and configure Base System

### (Optional) Select the mirrors
Move the geographically closest mirrors to the top of the list:
```bash
nano /etc/pacman.d/mirrorlist
```

### Install essential packages:
```bash
pacstrap -K /mnt base base-devel linux linux-firmware dosfstools mtools git nano sudo usbutils networkmanager man-db man-pages python gcc openssh
```

### Generate an fstab file:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

### Chroot into the new system:
```bash
arch-chroot /mnt
```

### Set TimeZone:
```bash
# See available timezones:
ls /usr/share/zoneinfo/

# Set timezone:
ln -sf /usr/share/zoneinfo/Europe/Warsaw /etc/localtime
```

### Run hwclock to generate /etc/adjtime:
```bash
hwclock --systohc
```

### Set Localization:
```bash
# uncomment necessary (en_GB.UTF-8 en_US.UTF-8 pl_PL.UTF-8 ru_RU.UTF-8 uk_UA.UTF-8)
nano /etc/locale.gen

locale-gen

echo LANG=en_GB.UTF-8 > /etc/locale.conf
echo KEYMAP=pl > /etc/vconsole.conf
```

### Set hostname:
```bash
echo <yourhostname> > /etc/hostname
```

### Set the root password:
```bash
passwd
```

### Create a user:
```bash
useradd -m -G wheel <yourusername>
passwd <yourusername>

# Uncomment "%wheel ALL=(ALL) ALL" with:
EDITOR=nano visudo
```

### Configure `mkinitcpio` with modules needed to create the initramfs image:
```bash
# Add 'encrypt' to HOOKS before 'filesystems' and remove 'consolefont'
nano /etc/mkinitcpio.conf

# regenerate initramfs image
mkinitcpio -P
```

### Setup Bootloader:
- [Systemd-boot (recommended for Secure boot)](systemd-boot.md)
- [GRUB](grub.md)

### Enable required services
```bash
systemctl enable NetworkManager.service grub-btrfsd systemd-timesyncd.service
```

1. [Optional] Enable multilib
```bash
nano /etc/pacman.conf
```
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```



### Reboot
```bash
exit
umount -R /mnt
reboot
```


# After Boot
## Network Connection
Use NetworkManager to connect to WIFI:
```bash
nmcli device wifi list
nmcli device wifi connect SSID_or_BSSID --ask
```

### (TODO: Remove?) Check if NTP is running and clock synchronized:
```bash
timedatectl status
timedatectl timesync-status
```

Enable if not:
```bash
sudo timedatectl set-ntp true
```

# Further installation

- System
  - [YAY (AUR helper)](install/yay.md)
  - [Snapper (BTRFS timeline snapshots)](install/snapper.md)
  - Secure Boot:
    - [Secure Boot with systemd](install/secure-boot-systemd.md)
    - [Secure Boot with GRUB](install/secure-boot-systemd.md)
  - [Fonts](install/fonts.md)
- [Hardware (in progress)](install/hardware)
- [Desktop (in progress)](install/desktop)



# References
- [Efficient Encrypted UEFI-Booting Arch Installation](https://gist.github.com/HardenedArray/31915e3d73a4ae45adc0efa9ba458b07)
