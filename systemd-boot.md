## 1. Install:

```bash
bootctl install
```

## 1. Configure for EFI stub (recommended):
1. Setup `loader.conf`:
	```bash
	nano /efi/loader/loader.conf
	```

	1. Auto entries:
		put the following to `/efi/loader/loader.conf`:
		```
		default arch-linux.efi
		timeout 2
		console-mode keep
		auto-entries false
		auto-firmware true
		editor false
		secure-boot-enroll off
		```

1. Edit mkinitcpio config:
	```bash
	nano /etc/mkinitcpio.d/linux.preset
	```

	Comment `default_image=` and `fallback_image=`.

	Uncomment `default_uki=`, `default_options=` and `fallback_uki=`.

	`/etc/mkinitcpio.d/linux.preset`:
	```bash
	# mkinitcpio preset file for the 'linux' package

	#ALL_config="/etc/mkinitcpio.conf"
	ALL_kver="/boot/vmlinuz-linux"

	PRESETS=('default' 'fallback')

	#default_config="/etc/mkinitcpio.conf"
	#default_image="/boot/initramfs-linux.img"
	default_uki="/efi/EFI/Linux/arch-linux.efi"
	default_options="--splash /usr/share/systemd/bootctl/splash-arch.bmp"

	#fallback_config="/etc/mkinitcpio.conf"
	#fallback_image="/boot/initramfs-linux-fallback.img"
	fallback_uki="/efi/EFI/Linux/arch-linux-fallback.efi"
	fallback_options="-S autodetect"
	```

1. Add `/etc/kernel/cmdline`:
	```bash
	echo root=UUID=$(blkid -o value /dev/mapper/root | sed -n '2 p') >> /etc/kernel/cmdline
	echo cryptdevice=UUID=$(blkid -o value /dev/<your-root-luks-partition>:root | sed -n '1 p') >> /etc/kernel/cmdline

	nano /etc/kernel/cmdline
	```

	`/etc/kernel/cmdline`:
	```
	root=UUID=<your-decrypted-root-partition-UUID> rw rootflags=subvol=@ cryptdevice=UUID=<your-encrypted-root-luks-partition-UUID>:root root=/dev/mapper/root loglevel=6
	```

## 1. Configure for kernel and initramfs separated:
1. Setup `loader.conf`
	```bash
	nano /boot/loader/loader.conf
	```

	put the following:
	```
	default arch.conf
	timeout 2
	console-mode keep
	auto-entries true
	auto-firmware true
	editor false
	secure-boot-enroll off
	```

1. Setup `arch.conf`
	```bash
	echo root=UUID=$(blkid -o value /dev/mapper/root | sed -n '2 p') >> /boot/loader/entries/arch.conf
	echo cryptdevice=UUID=$(blkid -o value /dev/<your-root-luks-partition>:root | sed -n '1 p') >> /boot/loader/entries/arch.conf

	nano /boot/loader/entries/arch.conf
	```

	change according to:
	```
	title   Arch Linux
	linux   /vmlinuz-linux
	initrd  /initramfs-linux.img
	options root=UUID=<your-decrypted-root-partition-UUID> rw rootflags=subvol=@ cryptdevice=UUID=<your-encrypted-root-luks-partition-UUID>:root root=/dev/mapper/root loglevel=6
	```

## 1. Re-generate images:
```bash
mkinitcpio -P
```


## 1. Secure boot:
After system booted successfully, you may want to proceed to signing images and enabling Secure boot.
Check the manual: [Secure boot for systemd-boot](secure-boot-systemd.md)
