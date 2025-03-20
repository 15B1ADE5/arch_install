# LiveUSB preparation
```bash
gpg -v archlinux-*.iso.sig

dd bs=4M if=archlinux-*.iso of=/dev/sdX status=progress oflag=sync
```

# Boot

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
#### List available devices:
```bash
fdisk -l
```

### Partition disk:
1.  #### Run `fdisk` to create Linux partitions:
	(forked from https://gist.github.com/mjnaderi/28264ce68f87f52f2cabb823a503e673)
	```bash
	fdisk /dev/<your-disk>
	```

	If you have installed Windows 11 on that disk, you already have a GPT partition table and EFI partition here.  

	Otherwise, if needed, create an empty GPT partition table using the `g` command (**WARNING:** This will erase the entire disk):
	```
	Command (m for help): g
	Created a new GPT disklabel (GUID: ...).
	```

1. #### Create partitions:
	1. Create the EFI partition:
		```
		Command (m for help): n
		Partition number (1-128, default 1): <Enter>
		First sector (2048-X, default 2048): <Enter>
		Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-Y, default Y): +200M

		Command (m for help): t
		Selected partition 1
		Partition type or alias (type L to list all): uefi
		```

	1. Create the Boot partition:
		```
		Command (m for help): n
		Partition number: <Enter>
		First sector: <Enter>
		Last sector, +/-sectors or +/-size{K,M,G,T,P}: +512M

		Command (m for help): t
		Partition number (1,2, default 2): <Enter>
		Partition type or alias (type L to list all): linux
		```

	1. Create the LUKS Root partition:
		```
		Command (m for help): n
		Partition number: <Enter>
		First sector: <Enter>
		Last sector, +/-sectors or +/-size{K,M,G,T,P}: <Enter>

		Command (m for help): t
		Partition number (1,2,3, default 3): <Enter>
		Partition type or alias (type L to list all): linux
		```

1. #### Print the partition table using the `p` command and check that everything is OK:
	```
	Command (m for help): p
	Disk ...
	```

	##### Info:  
	- According to numbers lately following naming used:  
		- partition 1: `/dev/<your-efi-partition>` - EFI
		- partition 2: `/dev/<your-boot-partition>` - boot
		- partition 3: `/dev/<your-root-luks-partition>` - root

		Example:
		```
		/dev/nvme0n1p1 -> /dev/<your-efi-partition>
		/dev/nvme0n1p1 -> /dev/<your-boot-partition>
		/dev/nvme0n1p1 -> /dev/<your-root-luks-partition>
		```

1. #### Write changes to the disk using the `w` command.
	(Make sure you know what you're doing before running this command).
	```
	Command (m for help): w
	The partition table has been altered.
	Calling ioctrl() to re-read partition table.
	Syncing disks.
	```

### Format partitions:
1.  Format the EFI partition:
	```bash
	mkfs.fat -F 32 /dev/<your-efi-partition>
	```

1.  Format the Boot partition:
	```bash
	mkfs.ext4 /dev/<your-boot-partition>
	```

1.  Set up root partition:
	1. Set up encrypted partition:
		```bash
		cryptsetup -v --use-random luksFormat /dev/<your-root-luks-partition>
		cryptsetup luksOpen /dev/<your-root-luks-partition> root
		```
		Note: You can choose any other name instead of `root`.

	1. Format the encrypted partition:
		```bash
		mkfs.btrfs -L archlinux /dev/mapper/root
		```
	
	1. Create BTRFS subvolumes:
		```bash
		# Mount the root fs
		mount /dev/mapper/root /mnt

		btrfs subvolume create /mnt/@
		btrfs subvolume create /mnt/@home
		btrfs subvolume create /mnt/@snapshots

		# Unmount the root fs
		umount /mnt
		```

### Mount partitions:
1. Mount BTRFS subvolumes (for installation):  
	(Selected mount flags: `noatime,discard=async,compress=zstd:1,ssd,space_cache=v2`)
	```bash
	mount -o noatime,discard=async,compress=zstd:1,ssd,space_cache=v2,subvol=@ /dev/mapper/root /mnt

	mkdir -p /mnt/home /mnt/.snapshots

	mount -o noatime,discard=async,compress=zstd:1,ssd,space_cache=v2,subvol=@home /dev/mapper/root /mnt/home
	mount -o noatime,discard=async,compress=zstd:1,ssd,space_cache=v2,subvol=@snapshots /dev/mapper/root /mnt/.snapshots
	```

1.  Mount EFI and BOOT filesystems:
	```bash
	mkdir -p /mnt/boot
	mount /dev/<your-boot-partition> /mnt/boot
	
	mkdir -p /mnt/boot/EFI
	mount /dev/<your-efi-partition> /mnt/boot/EFI
	```

1.  Check mounts (verify last entries):
	```bash
	mount
	```
## Install and configure Base System

### (Optional) Select the mirrors  
Move the geographically closest mirrors to the top of the list:
```bash
nano /etc/pacman.d/mirrorlist
```

### Install essential packages:
```bash
pacstrap -K /mnt base base-devel linux linux-firmware grub grub-btrfs dosfstools os-prober mtools efibootmgr git nano sudo usbutils networkmanager man-db man-pages python gcc openssh dhcpcd
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
# Add 'encrypt' to HOOKS before 'filesystems'
nano /etc/mkinitcpio.conf

# regenerate initramfs image
mkinitcpio -P
```

### Setup GRUB:  
(`grub`, `grub-btrfs`, `dosfstools`, `os-prober`, `mtools` and `efibootmgr` packages required)  

1. Install GRUB:  
	```bash
	grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB --recheck

1. Edit `/etc/default/grub`:
	```
	# Optionally: append root-disk-luks-uuid to use later:
	blkid -o value /dev/<your-root-luks-partition> | head -n 1 >> /etc/default/grub

	nano /etc/default/grub
	```

	1. (Optional) remove `quiet` from `GRUB_CMDLINE_LINUX_DEFAULT` to display boot logs

	1. (Optional) set `loglevel=` to 6-7 in `GRUB_CMDLINE_LINUX_DEFAULT` to display boot logs

		Other possible levels:
		```
		0 (KERN_EMERG)          system is unusable
		1 (KERN_ALERT)          action must be taken immediately
		2 (KERN_CRIT)           critical conditions
		3 (KERN_ERR)            error conditions
		4 (KERN_WARNING)        warning conditions
		5 (KERN_NOTICE)         normal but significant condition
		6 (KERN_INFO)           informational
		7 (KERN_DEBUG)          debug-level messages
		```

	1. Comment out `GRUB_DISABLE_RECOVERY=true` to enable generation of recovery mode menu entries.

	1. Uncomment `GRUB_DISABLE_OS_PROBER=false` to enable os-prober.
	
	1. Set/add or uncomment `GRUB_ENABLE_CRYPTODISK`:
		```
		GRUB_ENABLE_CRYPTODISK=y
		```

	1. set correct `cryptdevice` in `GRUB_CMDLINE_LINUX`:
		(use `blkid -o value /dev/<your-root-luks-partition> | head -n 1` )
		```
		GRUB_CMDLINE_LINUX="cryptdevice=UUID=<your-root-luks-partition-UUID>:root root=/dev/mapper/root"
		# or without UUID
		GRUB_CMDLINE_LINUX="cryptdevice=/dev/<your-root-luks-partition>:root root=/dev/mapper/root"
		```

1. Generate grub.cfg:
	```bash
	grub-mkconfig -o /boot/grub/grub.cfg
	```
### Enable required services
```bash
systemctl enable NetworkManager.service
```


### Reboot
```bash
exit
umount -R /mnt
reboot
```


# References
- [Efficient Encrypted UEFI-Booting Arch Installation](https://gist.github.com/HardenedArray/31915e3d73a4ae45adc0efa9ba458b07)
