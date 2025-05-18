# Grub set-up

1. Install required packages:
	```bash
	sudo pacman -Sy grub grub-btrfs dosfstools os-prober mtools efibootmgr
	```


1. Install GRUB:
	```bash
	grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB --recheck
	```


1. Edit `/etc/default/grub`:
	```bash
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
		```bash
		GRUB_CMDLINE_LINUX="cryptdevice=UUID=<your-root-luks-partition-UUID>:root root=/dev/mapper/root"
		# or without UUID
		GRUB_CMDLINE_LINUX="cryptdevice=/dev/<your-root-luks-partition>:root root=/dev/mapper/root"
		```

1. Generate grub.cfg:
	```bash
	grub-mkconfig -o /boot/grub/grub.cfg
	```
