**(in progress)**

## 1. Install

```bash
sudo pacman -Sy fprint fwupd
yay -S framework-system-git
```

For Battery/Fan control:
```bash
yay -S fw-ectool-git
```

## 1. Check HW firmware updates:
see https://knowledgebase.frame.work/en_us/updating-fingerprint-reader-firmware-on-linux-for-13th-gen-and-amd-ryzen-7040-series-laptops-HJrvxv_za


## Configuration:
1. Limit battery usage:
   ```bash
   sudo ectool chargecontrol normal 65 65
   ```
