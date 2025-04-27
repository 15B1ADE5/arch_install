# Install

Install required pkgs:
```bash
sudo pacman -Sy hyprland uwsm greetd greetd-regreet pipewire wireplumber xdg-desktop-portal-hyprland brightnessctl hyprpaper hyprpicker hypridle hyprlock hyprcursor hyprpolkitagent hyprsunset mako hyprland-qt-support hyprutils hyprland-qtutils qt5-wayland qt6-wayland waybar wofi cliphist network-manager-applet udiskie kitty
```

# Setup

## WindowManager (Greetd)
create `/etc/greetd/hyprland.conf`:
```
exec-once = regreet; hyprctl dispatch exit
misc {
	disable_hyprland_logo = true
	disable_splash_rendering = true
	disable_hyprland_qtutils_check = true
}
```

and edit `/etc/greetd/config.toml`:
```
[terminal]
# The VT to run the greeter on. Can be "next", "current" or a number
# designating the VT.
vt = 1

# The default session, also known as the greeter.
[default_session]

# `agreety` is the bundled agetty/login-lookalike. You can replace `/bin/sh`
# with whatever you want started, such as `sway`.
# command = "agreety --cmd /bin/sh"
command = "Hyprland --config /etc/greetd/hyprland.conf"

# The user to run the command as. The privileges this user must have depends
# on the greeter. A graphical greeter may for example require the user to be
# in the `video` group.
user = "greeter"
```

enable Greetd:
```bash
systemctl enable greetd.service
```

restart system

During first login in Greetd, remember to select Hyprland with sw
