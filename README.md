# GRUB Theme Ciel Mocha

GRUB2 theme for Linux. The background is the Ciel character inside a HUD
frame, using the Catppuccin Mocha palette and the BigBlueTerm437 Nerd Font.

<p align="center">
  <img src="assets/grub-ciel-vm-6rows-icons.png" alt="GRUB boot screen — 6 rows with OS icons" width="900">
</p>

## Structure

```
src/
├── theme.txt              # theme configuration
├── background.png         # 1920x1080, PNG24, non-interlaced
├── item_c.png             # unselected option bar
├── select_c.png           # selected option bar
├── logo_ciel.png          # 226x226 PNG32 (unused — the avatar is baked into the background)
├── bigblue24.pf2          # font for menu items
├── bigblue28.pf2          # font for the label
└── icons/                 # OS icons, 36x36 PNG32
```

## Install

Copy the theme and the fonts:

```bash
sudo mkdir -p /boot/grub/themes/TEN_THEME
sudo cp -r src/*.png src/*.pf2 src/theme.txt src/icons /boot/grub/themes/TEN_THEME/
sudo cp src/bigblue*.pf2 /boot/grub/fonts/
```

Edit `/etc/default/grub`:

```
GRUB_THEME="/boot/grub/themes/TEN_THEME/theme.txt"
GRUB_FONTS="bigblue28 bigblue24"
GRUB_GFXMODE=1920x1080x32,auto
```

Do not set `GRUB_TERMINAL_OUTPUT`, it disables the theme.

Update the configuration:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

A `Found theme` line means the theme was picked up.
