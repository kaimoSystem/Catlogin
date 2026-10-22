Catlogin

A clean, Catppuccin-inspired SDDM login theme built with QML for Plasma 6 / Qt6.

![Catlogin preview](preview.png)

## Requirements
- SDDM with Qt6 support (QtVersion=6)
- Qt6 SVG plugin (only if using SVG icons — qt6-svg / libQt6Svg6 / qt6-qtsvg depending on distro)

## Credit

The weekday + date display design is **heavily inspired by zayronoxi's "Urban Date" 0.6 KDE Plasma widget**.
No code was copy-pasted from that widget — this is an original QML implementation built from scratch for SDDM — but visually the 
layout is very close to theirs, minus the proprietary font it originally shipped with (swapped here for a freely distributable alternative).
Full credit to zayronoxi for the original design concept.

## How to Install

### Login screen

```bash
sudo cp -r Catlogin /usr/share/sddm/themes/catlogin
```

Set it as the active theme in `/etc/sddm.conf.d/theme.conf`:

```ini
[Theme]
Current=catlogin
```

Then restart SDDM:

```bash
sudo systemctl restart sddm
```

### Lock screen (Super / Win + L)

Locking a running Plasma session is not handled by SDDM but by Plasma's own
lock screen (kscreenlocker), so by default you get the Breeze lock screen there.
The `lockscreen/` folder makes kscreenlocker show Catlogin too. It is only used by
KWin (which starts the lock screen), so your panels and desktop stay untouched.

The setting goes into KWin's systemd service, and its name depends on your session:
`plasma-kwin_wayland.service` on Wayland, `plasma-kwin_x11.service` on X11.
`$XDG_SESSION_TYPE` contains exactly that part (`wayland` or `x11`), so this works for both:

```bash
service=plasma-kwin_$XDG_SESSION_TYPE.service
mkdir -p ~/.config/systemd/user/$service.d
printf '[Service]\nEnvironment=PLASMA_DEFAULT_SHELL=/usr/share/sddm/themes/catlogin/lockscreen\n' \
    > ~/.config/systemd/user/$service.d/catlogin-lockscreen.conf
```

If you copied the theme somewhere other than `/usr/share/sddm/themes/catlogin`,
change the path after `PLASMA_DEFAULT_SHELL=` (it must point to the `lockscreen` folder).

Log out and back in once so KWin picks up the setting.

## Uninstall

### Login screen

SDDM uses whatever theme `Current=` names, so removing that line is enough.
If `/etc/sddm.conf.d/theme.conf` only contains the lines from above, just delete it:

```bash
sudo rm /etc/sddm.conf.d/theme.conf
```

SDDM falls back to its default theme the next time the login screen shows up.
Afterwards you can also delete the theme folder:

```bash
sudo rm -r /usr/share/sddm/themes/catlogin
```

If you set up the lock screen too, remove it first (see below), because it uses files from that folder.

### Lock screen

Delete the `.conf` file you created and log out and back in. Plasma's Breeze lock screen is back then:

```bash
rm ~/.config/systemd/user/plasma-kwin_$XDG_SESSION_TYPE.service.d/catlogin-lockscreen.conf
```

## Testing

### Login screen

```bash
sddm-greeter-qt6 --test-mode --theme /usr/share/sddm/themes/catlogin
```

Note: `--test-mode` disables real login/suspend/reboot/shutdown actions — it's for visual testing only.

### Lock screen

You can open the lock screen in a normal window without locking your session.
The program behind it, `kscreenlocker_greet`, lives in a different place on every distro
(e.g. `/usr/lib/x86_64-linux-gnu/libexec/` on Debian/Ubuntu, `/usr/lib/` on Arch,
`/usr/libexec/` on Fedora/openSUSE), so let `find` look it up:

```bash
greeter=$(find /usr/lib /usr/libexec -name kscreenlocker_greet 2>/dev/null | head -n 1)
"$greeter" --testing --shell /usr/share/sddm/themes/catlogin/lockscreen
```

Your real password closes the window, `Ctrl + C` in the terminal works too.
Any QML errors are printed right there in the terminal, so this is the place to look first
when something doesn't show up or looks wrong.

### Debugging the real lock screen

Check that KWin actually got the setting. This should print the `PLASMA_DEFAULT_SHELL` path:

```bash
systemctl --user show plasma-kwin_$XDG_SESSION_TYPE.service -p Environment
```

If it doesn't, the `.conf` file is in the wrong folder or you haven't logged out and back in yet.

The real lock screen is started by KWin, so its errors end up in KWin's log:

```bash
journalctl --user -b -u plasma-kwin_$XDG_SESSION_TYPE.service | grep -iE -A3 'kscreenlocker|lockscreen'
```

If the lock screen ever breaks and you can't unlock anymore, switch to a text console with
`Ctrl + Alt + F3`, log in there and run:

```bash
loginctl unlock-sessions
```

Then go back with `Ctrl + Alt + F1` (or `F2`, depending on your distro) and remove the
`.conf` file as described under [Uninstall](#uninstall).

## Configuration

All colors, icons, fonts, and wave-animation parameters live in `theme.conf`. To customize without losing your changes on theme updates, 
create a `theme.conf.user` file in the same directory with just the values you want to override.

## License

Feel free to fork, modify, and re-theme. If you build something cool with it, a shoutout is appreciated but not required.
