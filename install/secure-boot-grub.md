# Secureboot with GRUB:

## 1. Install required pkgs:

```bash
sudo pacman -Sy efitools sbctl
```

## 1. Backup current keys:
Print current keys:
```bash
sbctl list-enrolled-keys
```

Backup:
```bash
cd <your-keys-backup-path>
for var in PK KEK db dbx ; do efi-readvar -v $var -o old_${var}.esl ; done
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

1. Reinstall GRUB:
```bash
sudo grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB --modules="tpm" --disable-shim-lock --recheck
```

1. (To check) Possibly EFI partition is required to be additionally mounted under `/efi` or `/boot`

1. Sign binaries:
```bash

sudo sbctl verify    # May return 'failed to find EFI system partition'
sudo sbctl sign -s /boot/EFI/EFI/GRUB/grubx64.efi
sudo sbctl sign -s /boot/vmlinuz-linux
```
