# Boot

## After live-usb booted

### load locale:
```
loadkeys pl
```

### (Optional) set font:
```
setfont ter-120b
```

### Verify the boot mode
```
cat /sys/firmware/efi/fw_platform_size # should return 64
```

### Check connection to Internet (Ethernet):
```
ip addr
ping archlinux.org
```

### Update the system clock
```
timedatectl
```

## Prepare partitions
#### List available devices:
```
fdisk -l
```

### Partition disk:
#### Run `fdisk` to create Linux partitions:
(forked from https://gist.github.com/mjnaderi/28264ce68f87f52f2cabb823a503e673)
```
fdisk /dev/<your-disk>
```

If you have installed Windows 11 on that disk, you already have a GPT partition table and EFI partition here.  

Otherwise, if needed, create an empty GPT partition table using the `g` command (**WARNING:** This will erase the entire disk):
```
Command (m for help): g
Created a new GPT disklabel (GUID: ...).
```

#### Create partitions:
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

REMOVE?
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

1. Create the LUKS partition:
	```
	Command (m for help): n
	Partition number: <Enter>
	First sector: <Enter>
	Last sector, +/-sectors or +/-size{K,M,G,T,P}: <Enter>

	Command (m for help): t
	Partition number (1,2,3, default 3): <Enter>
	Partition type or alias (type L to list all): linux
	```

#### Print the partition table using the `p` command and check that everything is OK:
```
Command (m for help): p
Disk ...
```

#### Write changes to the disk using the `w` command.
(Make sure you know what you're doing before running this command).
```
Command (m for help): w
The partition table has been altered.
Calling ioctrl() to re-read partition table.
Syncing disks.
```

### Format partitions:
1.  Format the EFI and Boot Partitions:
	```
	mkfs.fat -F 32 /dev/<your-disk-efi>
	REMOVE? mkfs.ext4 /dev/<your-disk-boot>
	```

1.  Set up the encrypted partition.
	You can choose any other name instead of `root`:
	```
	cryptsetup --use-random luksFormat /dev/<your-disk-luks>
	cryptsetup luksOpen /dev/<your-disk-luks> root
	```

1.  Prepare encrypted volume:  
	#### Option A: Create BTRFS on top of LUKS (recommended):  
	1. Format partition:
		```
		mkfs.btrfs -L archlinux /dev/mapper/root
		```
	
	1. Create BTRFS subvolumes:
		```
		# Mount the root fs
		mount /dev/mapper/root /mnt

		btrfs subvolume create /mnt/@
		btrfs subvolume create /mnt/@home
		btrfs subvolume create /mnt/@snapshots

		# Unmount the root fs
		umount /mnt
		```

	1. Mount BTRFS subvolumes (for installation):  
		(Selected mount flags: `noatime,discard=async,compress=zstd:1,ssd,space_cache=v2`)
		```
		mount -o noatime,discard=async,compress=zstd:1,ssd,space_cache=v2,subvol=@ /dev/mapper/root /mnt

		mkdir -p /mnt/home /mnt/.snapshots /mnt/efi /mnt/boot/EFI

		mount -o noatime,discard=async,compress=zstd:1,ssd,space_cache=v2,subvol=@home /dev/mapper/root /mnt/home
		mount -o noatime,discard=async,compress=zstd:1,ssd,space_cache=v2,subvol=@snapshots /dev/mapper/root /mnt/.snapshots
		```

	1.  Mount EFI and BOOT filesystems:
		```
		REMOVE?
		mount /dev/<your-disk-boot> /mnt/boot
		mount /dev/<your-disk-efi> /mnt/boot/EFI
		```
## Install and configure Base System

### (Optional) Select the mirrors  
Move the geographically closest mirrors to the top of the list:
```
nano /etc/pacman.d/mirrorlist
```

### Install essential packages:
```
# pacstrap -K /mnt base base-devel linux linux-firmware grub grub-btrfs dosfstools os-prober mtools efibootmgr git nano sudo usbutils networkmanager man-db man-pages python gcc openssh
```

### Generate an fstab file
```
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab

```

### Chroot into the new system: 
```
arch-chroot /mnt
```

### Set TimeZone:
```
# See available timezones:
ls /usr/share/zoneinfo/

# Set timezone:
ln -sf /usr/share/zoneinfo/Europe/Warsaw /etc/localtime
```

### Run hwclock to generate /etc/adjtime:  
```
hwclock --systohc
```

### Set Localization:  
```
# uncomment necessary (en_GB.UTF-8 en_US.UTF-8 pl_PL.UTF-8 ru_RU.UTF-8 uk_UA.UTF-8)
nano /etc/locale.gen

locale-gen

echo LANG=en_GB.UTF-8 > /etc/locale.conf
echo KEYMAP=pl > /etc/vconsole.conf
```

### Set hostname:  
```
echo <yourhostname> > /etc/hostname
```

### Set the root password:  
```
passwd
```

### Create a user:  
```
useradd -m -G wheel <yourusername>
passwd <yourusername>

# Uncomment "%wheel ALL=(ALL) ALL" with:
EDITOR=nano visudo
```

### Configure `mkinitcpio` with modules needed to create the initramfs image:
```
# Add 'encrypt' to HOOKS before 'filesystems'
nano /etc/mkinitcpio.conf

# regenerate initramfs image
mkinitcpio -P
```

### Setup GRUB:  
(`grub`, `grub-btrfs`, `dosfstools`, `os-prober`, `mtools` and `efibootmgr` packages required)  

1. Install GRUB:  
	```
	# with CA Keys:

	# without Secure Boot
	grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRÓB --modules="tpm" --disable-shim-lock --recheck
	```

1. Edit `/etc/default/grub`:
	```
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
		```
		GRUB_CMDLINE_LINUX="cryptdevice=/dev/<your-disk-luks>:root"
		```

### 
```

```

### 
```

```

### 
```

```

### 
```

```

### 
```

```

### 
```

```

### 
```

```

### 
```

```