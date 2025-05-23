# BTRFS Snapshots (snapper)

### Install
```bash
sudo pacman -Sy snapper snap-pac
```
If grub installed:
```bash
sudo pacman -Sy grub-btrfs
yay -S snap-pac-grub
```

### Setup
```bash
sudo umount /.snapshots && sudo rm -fr /.snapshots
sudo snapper -c root create-config /
sudo btrfs subvolume delete /.snapshots
sudo mkdir /.snapshots && sudo mount -a
```

### Configure configs
```bash
sudo nano /etc/snapper/configs/root
```

### Enable snapshoting services
```bash
systemctl enable snapper-timeline.timer snapper-cleanup.timer
```
