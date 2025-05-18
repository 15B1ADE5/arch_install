# Secureboot with Systemd:

## 1. Install required pkgs:

```bash
sudo pacman -Sy efitools sbctl
```

## 1. Prepare pacman hooks for systemd-boot:
```bash
yay -S systemd-boot-pacman-hook
sudo nano /etc/pacman.d/hooks/80-secureboot.hook
```

`/etc/pacman.d/hooks/80-secureboot.hook`:
```
[Trigger]
Operation = Install
Operation = Upgrade
Type = Path
Target = usr/lib/systemd/boot/efi/systemd-boot*.efi

[Action]
Description = Signing systemd-boot EFI binary for Secure Boot
When = PostTransaction
Exec = /bin/sh -c 'while read -r i; do sbctl sign -s -o "$i"; done;'
Depends = sh
Depends = sbctl
NeedsTargets
```



## 1. Backup current keys:
Print current keys:
```bash
sbctl list-enrolled-keys
```

Backup:
```bash
cd <your-keys-backup-path>
for var in PK KEK db dbx ; do efi-readvar -v $var -o original_${var}.esl ; done
```

## 1. Put firmware in Setup Mode: delete or clear certificates.~


## 1. Use `sbctl`:
1. Check if Setup mode is enabled:
	```bash
	sbctl status

	Installed:	✓ sbctl is installed
	Owner GUID:	<GUID>
	Setup Mode:	✗ Enabled
	Secure Boot:	✗ Disabled
	Vendor Keys:	none
	```

1. Create/enroll keys:
	```bash
	sudo sbctl create-keys
	sudo sbctl enroll-keys -f -m   # Recommended
	sudo sbctl enroll-keys -f      # Only if system will work without MS keys

	# check
	sbctl status
	sbctl list-enrolled-keys
	```

1. Sign binaries:
	```bash
	sudo sbctl verify

	sudo sbctl sign -s /efi/EFI/BOOT/BOOTX64.EFI
	sudo sbctl sign -s /efi/EFI/systemd/systemd-bootx64.efi
	sudo sbctl sign -s /efi/EFI/Linux/arch-linux.efi
	sudo sbctl sign -s /efi/EFI/Linux/arch-linux-fallback.efi

	sudo sbctl verify
	```

1. (Optional) Check if linux kernel will be signed during update:

	Reinstall kernel:
	```bash
	sudo pacman -Sy linux
	```

	the output should contain:
	```
	...
	-> Running post hook: [sbctl]
	Signing /efi/EFI/Linux/arch-linux.efi
	✓ Signed /efi/EFI/Linux/arch-linux.efi
	==> Post processing done
	...

	...
	-> Running post hook: [sbctl]
	Signing /efi/EFI/Linux/arch-linux-fallback.efi
	✓ Signed /efi/EFI/Linux/arch-linux-fallback.efi
	==> Post processing done
	(4/5) Signing EFI binaries...
	Generating EFI bundles....
	File has already been signed /efi/EFI/Linux/arch-linux.efi
	File has already been signed /efi/EFI/systemd/systemd-bootx64.efi
	File has already been signed /efi/EFI/BOOT/BOOTX64.EFI
	File has already been signed /efi/EFI/Linux/arch-linux-fallback.efi
	...
	```


## 1. (Optional) Re-Enroll backup keys
If your machine had dbx database cleared during Secure boot setup, it's recommended to write them back:

1. Copy dbx keys to `sbctl` (just to keep with other keys):
	```bash
	# REMOVE: sudo pacman -Sy sbsigntools

	sudo mkdir -p /var/lib/sbctl/keys/custom/dbx
	sudo cp <your-keys-backup-path>/original_dbx.esl  /var/lib/sbctl/keys/custom/dbx/
	```

1. Check Setup Mode:
	```bash
	sbctl status
	```

1. If `Setup Mode: Enabled`:
	you can enroll `dbx` database without signing:
	```bash
	sudo efi-updatevar -a -e -f /var/lib/sbctl/keys/custom/dbx/original_dbx.esl -k /var/lib/sbctl/keys/KEK/KEK.key dbx
	```


1. TODO: Fix/verify: ~If `Setup Mode: Disabled` the `dbx` database needs to be signed with KEK first:~
	```bash
	sudo sign-efi-sig-list -g "$(< /var/lib/sbctl/GUID)" -k /var/lib/sbctl/keys/KEK/KEK.key -c /var/lib/sbctl/keys/KEK/KEK.pem dbx /var/lib/sbctl/keys/custom/dbx/original_dbx.esl /var/lib/sbctl/keys/custom/dbx/original_dbx.auth

	sudo efi-updatevar -a -f /var/lib/sbctl/keys/custom/dbx/original_dbx.auth dbx
	```
