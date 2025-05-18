For network scanner, tested on Brother DCP-7065DN:
```bash
sudo pacman -Sy sane sane-airscan simple-scan
yay -S brscan4 brother-dcp7065dn

sudo brsaneconfig4 -a name=Brother-DCP-7065DN model=DCP-7065DN ip=<printer-IP>
```

Additional packages:
```
xsane-gimp # A GIMP plugin to scan from GIMP

```
