# List available devices:
```bash
fdisk -l
```

# Partition disk:
1.  ## Run `fdisk` to create Linux partitions:
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

1. ## Create partitions:
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

1. ## Print the partition table using the `p` command and check that everything is OK:
	```
	Command (m for help): p
	Disk ...
	```

	### Info:
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

1. ## Write changes to the disk using the `w` command.
	(Make sure you know what you're doing before running this command).
	```
	Command (m for help): w
	The partition table has been altered.
	Calling ioctrl() to re-read partition table.
	Syncing disks.
	```

# Format partitions:
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

# Mount partitions:
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
