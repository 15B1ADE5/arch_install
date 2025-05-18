## 1. Install

```bash
sudo pacman -Sy bluez bluez-utils  bluetui blueman

systemctl enable bluetooth.service
systemctl start bluetooth.service
```

Additional reboot may be requred.

## 2. Connect devices

```bash
rfkill # check if bluetooth is SOFT/HARD unblocked

bluetoothctl
```

`[bluetoothctl]`:
```
# power on
# scan on
# trust <MAC>
# pair <MAC>
# connect <MAC>
```

additionall unblock may be requred:
```
[<DEVICE>] # unblock
```

Sometimes the above must be repeat with firstly:
```
# power on
```
