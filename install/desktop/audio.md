[sof-firmware](https://archlinux.org/packages/?name=sof-firmware) - for Lenovo E14 Gen2 audio

```bash
sudo pacman -Sy pipewire lib32-pipewire pipewire-docs wireplumber qpwgraph pipewire-audio pipewire-alsa pipewire-pulse easyeffects
```

additional ALSA:
alsa-utils pavucontrol

# device specific (check if required):
alsa-firmware
sof-firmware


For Some Intel based (eg Thinkpad):
```bash
sudo pacman -Sy sof-firmware
```
