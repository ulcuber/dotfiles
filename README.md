Below described installation process for a new system. Instructions given suppose no `systemd`, focus on `OpenRC` and rely on `chezmoi`. Targeting `Artix` or secondly `Gentoo`

First install the system:

# Artix

Follow <https://wiki.artixlinux.org/Main/Installation>

## Wi-Fi

Note that artix-live will have `NetworkManager` running. You can see in `rc-status`. So you might want to use on Wi-Fi:

```bash
nmcli dev wifi list
nmcli dev wifi connect "name" password "pass"
# or better
nmcli dev wifi connect "name" --ask
```

So no `/etc/wpa_supplicant/` required like you could use with `netifrc`. And you don't need to install `dhcpcd` or `wpa_supplicant` with `NetworkManager` too

See connections: `/etc/NetworkManager/system-connections/`

Applicable for Gentoo `net-misc/networkmanager`

## Ntpd

`chrony` must be already running

## Hostname

Please set reasonable `/etc/hostname` or you will need later:

```bash
rm -rf ~/.config/chromium/Singleton*
```

It will help you to use `Deskflow`

## NetworkManager

So if you chose it then inside the chroot install it:

```bash
pacman -S networkmanager-openrc
rc-update add NetworkManager default
```

Then out of the chroot you can reuse connection:

```bash
cp /etc/NetworkManager/system-connections/* /mnt/etc/NetworkManager/system-connections/
```

Note that Gentoo can have different `interface-name` when Artix -> Gentoo chroot installing

## SSH

You can install `sshd` to continue from another machine after reboot:

```bash
pacman -S openssh-openrc
vim /etc/ssh/sshd_config
rc-update add sshd default
rc-service sshd start
```

From second machine: `ssh-copy-id hostname`

## Sudoers

You might want to run `visudo`

Add groups if needed: `usermod -aG wheel <user>`

# Gentoo

Follow the handbook: <https://wiki.gentoo.org/wiki/Handbook:AMD64>

You can chroot from Artix. Refer to Artix `NetworkManager` section

# If you corrupted UEFI

## Acer

- Turn off your laptop
- Hold down the **Fn** key and the **Left Alt** key simultaneously
- While keeping them held down, press the **Power button** once
- The moment the Acer logo flashes, start smashing the **F2** key (or **F12**).

You might want **Main/F12 Boot Menu/[Enabled]**

## Or

Flip the laptop and hold paperclip in a pinhole with a battery icon for 15-30 seconds, or more than 30 seconds

# Install some packages

Add needed groups:

```bash
usermod -aG video $USER
usermod -aG audio $USER
```

## Artix

### Intel

Check docs: <https://wiki.gentoo.org/wiki/Intel>

```bash
sudo pacman -S intel-media-driver vulkan-intel
# old hardware
sudo pacman -S libva-intel-driver
sudo pacman -S xf86-video-intel
sudo pacman -S xf86-video-fbdev
sudo pacman -S intel-compute-runtime
```

---

```bash
# fonts. You may want for PS1 return icons. And the last is needed for polybar
sudo pacman -S noto-fonts-emoji noto-fonts-cjk otf-comicshanns-nerd
sudo pacman -S git figlet fortune-mod bash-completion jq yazi ncdu ouch w3m imagemagick rsync fzf
# xorg. Includes i3 keybinds. acpilight -> xbacklight but not xrandr
sudo pacman -S xorg-server xorg-xinit i3-wm polybar rofi picom feh scrot xdotool kitty xclip gvim acpilight playerctl xsettingsd xorg-xev
# web browser
sudo pacman -S ungoogled-chromium
# or
sudo pacman -S chromium
# other utils
sudo pacman -S graphviz tmux perl-image-exiftool
# video player
vlc vlc-plugin-x264 vlc-plugin-ffmpeg
# video recording
sudo pacman -S obs-studio
# video editing
sudo pacman -S kdenlive
# might be too old. Replace
sudo pacman -S net-tools
# localization
sudo pacman -S hunspell-en_us hunspell-ru

# java runtime
sudo pacman -S jre-openjdk
# vim COC bash
sudo pacman -S shellcheck-bin
```

### Power profiles

```bash
sudo pacman -S python-gobject
sudo pacman -S power-profiles-daemon-openrc
sudo rc-update add power-profiles-daemon default
sudo rc-service power-profiles-daemon start
powerprofilesctl list
powerprofilesctl set power-saver
```

Might be good for AMD to use `amd-pstate-ut`

### Sound

```bash
sudo pacman -S pipewire-openrc pipewire-pulse-openrc wireplumber-openrc bluez-openrc bluez-utils sof-firmware pavucontrol
sudo rc-update add bluetoothd default
sudo rc-service bluetoothd start

# user services. No sudo
rc-update add -U pipewire default
rc-update add -U pipewire-pulse default
rc-update add -U wireplumber default
rc-service --user wireplumber start
# rc-service --user pipewire start
rc-service --user pipewire-pulse start
rc-status -U
pactl list sinks
```

# Rust

```bash
sudo pacman -S rustup
rustup default stable
```

## Cargo

```bash
cargo install gitui --locked
```

# Go

```bash
sudo pacman -S go
go install github.com/charmbracelet/glow/v2@latest
# extra/shfmt. For vim ALE bash
go install mvdan.cc/sh/v3/cmd/shfmt@latest
```

# C/C++

## Artix

```bash
sudo pacman -S make cmake cppcheck astyle
```

## Gentoo

```bash
sudo emerge --ask --quiet --verbose dev-util/cppcheck dev-util/astyle
```

# Python

## Artix

```bash
# openblas essential for all system
sudo pacman -S blas-openblas python-numpy
# good to have global
sudo pacman -S python-ruff ruff
```

## Gentoo

```bash
sudo mkdir -p /etc/portage/env
sudo vim /etc/portage/env/o3
```

```conf
COMMON_FLAGS="-O3 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
```

```bash
sudo mkdir -p /etc/portage/package.env
sudo vim /etc/portage/package.env/o3
```

```conf
sci-libs/openblas o3
dev-python/numpy o3
sci-libs/lapack o3
```

```bash
sudo emerge --ask --quiet --verbose sci-libs/openblas dev-python/numpy
```

For other Python packages check repository Makefiles. Most of them provide `make pacman`

# GRUB

For distinctive multiple kernel from other distros edit `/etc/grub.d/30_os-prober` around `menuentry`:

```bash
title="${LLABEL} $LKERNEL $onstr"
```

Edit `/etc/default/grub`

```bash
# remember last choice
GRUB_DEFAULT="saved"
# infinite timeout
GRUB_TIMEOUT=-1

# AMD optional `amd_pstate_ut=active`
# for lowmem only, if swap: `zswap.enabled=1 zswap.max_pool_percent=50`. Not recommended
# not working touchscreen: `usbcore.quirks=2386:3125:bk`. See https://gist.github.com/ulcuber/1a1da7886b8501ee724220750069a3ed
GRUB_CMDLINE_LINUX_DEFAULT=""

# usually it comes with theme
GRUB_THEME="/boot/grub/themes/starfield/theme.txt"
# to see all boot entries without nested menus
GRUB_DISABLE_SUBMENU=y
# detect other OSes
GRUB_DISABLE_OS_PROBER=false
```

Then make new config:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

# SSH

```bash
ssh-keygen
cat ~/.ssh/id_ed25519.pub
```

Add machine public key to <https://github.com/settings/ssh/new>

# Fstab

```bash
lsblk -f
sudo vim /etc/fstab
```

# Sync configs

Download `chezmoi`

```bash
export GITHUB_USERNAME=
sh -c "$(curl -fsLS https://get.chezmoi.io)" -- -b $HOME/.local/bin init git@github.com:$GITHUB_USERNAME/dotfiles.git
```

Fill `~/.config/chezmoi/chezmoi.yaml` data. You might need:

```bash
ls /sys/class/power_supply
ls /sys/class/backlight
sensors
```

Then `~/.local/bin/chezmoi apply`

## Theme

To run `i3` init theme files

```bash
~/.config/xsettingsd/set-theme <name>
~/.config/xsettingsd/set-color-scheme light
```

Note that these are the main scripts that trigger other `theme` and `color-scheme`

For example, `rofi`, `xrdb`, `kitty` have the same scripts that manage their own symlinks

Xsettingsd composes all these scripts

# .bashrc

Add to `~/.bashrc`:

```bash
if [ -f ~/.bashrc_common ]; then
	. ~/.bashrc_common
fi
```

The local .bashrc will be private for system specific vars like:

```bash
export BROWSER='/usr/bin/chromium'

. /usr/share/nvm/init-nvm.sh

export BUN_INSTALL="$HOME/.bun"
export PATH="$BUN_INSTALL/bin:$PATH"
```

# /etc

```bash
# from chezmoi
cd ~/.config/etc-backup/
# needs packages to install their configs
# Repeat for new packages
# Does not override existing configs
make install
```

## Kmonad

```bash
sudo mkdir /var/log/$USER
sudo chown $USER:$USER /var/log/$USER
sudo usermod -aG input $USER
sudo groupadd uinput
sudo usermod -aG uinput $USER
sudo rc-update add kmonad default
sudo udevadm control --reload-rules
sudo udevadm trigger
sudo modprobe uinput

# install kmonad itself too
sudo rc-service kmonad start
sudo rc-service kmonad status
```

# PHP

```bash
sudo pacman -S php-fpm php-fpm-openrc php-pgsql php-apcu php-imagick php-redis php-sqlite

cd /usr/bin
sudo ln -s php php8.5
```

## Composer

Check latest instruction <https://getcomposer.org/download/>

```bash
composer global require laravel/installer
```

## PIE

<https://github.com/php/pie>

```bash
curl -fL --output /tmp/pie.phar https://github.com/php/pie/releases/latest/download/pie.phar
# get hash: https://github.com/php/pie/releases
# no `sha256:` prefix
PIE_HASH="e8d27c100e85720f374a55f79d63fa8d686dedabc9ccbc6567085e0baf646d55"
echo "$PIE_HASH  /tmp/pie.phar" | sha256sum --check
sudo mv /tmp/pie.phar /usr/local/bin/pie
sudo chmod u+x /usr/local/bin/pie

pie install swoole/swoole
```

```bash
# uncomment extensions: bcmath curl iconv intl mysqli pdo_mysql pdo_pgsql pdo_sqlite pgsql sockets sodium sqlite3 zip
sudo vim /etc/php/php.ini

sudo vim /etc/php/php-fpm.d/www.conf
sudo rc-service php-fpm start
```

# MySQL

```bash
sudo pacman -S mariadb mariadb-openrc
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
sudo vim ~/.my.cnf
sudo rc-service mysql start
```

# Server: nginx

```bash
sudo pacman -S nginx nginx-openrc
sudo vim /etc/nginx/nginx.conf
sudo nginx -t
sudo vim /etc/hosts
```

# Redis

```bash
sudo pacman -S valkey redis-openrc
```

`sudo vim /etc/valkey/valkey.conf`:

```conf
unixsocket /run/valkey/valkey.sock
unixsocketperm 755
```

```
sudo mkdir /run/valkey
sudo chown valkey:wheel /run/valkey
```

`sudo vim /etc/init.d/redis`: `redis` to `valkey`

`sudo vim /etc/php/conf.d/redis.ini`

```bash
sudo rc-service redis start
```

# Xhprof

See: <https://gist.github.com/ulcuber/ea20e10fa961721afb94e7552ec9992b>

# Runlevels

```bash
sudo rc-update -s add default mysql
sudo rc-update add php-fpm mysql
sudo rc-update add mysql mysql
sudo rc-update add redis mysql
sudo rc-update add nginx mysql
```

# Node

## Artix

```bash
sudo pacman -S nvm unzip
```

## Generic/Gentoo

See: <https://github.com/nvm-sh/nvm>

---

```bash
curl -fsSL https://bun.sh/install | bash
```

You may want a workaround for vim ALE:

```bash
ln -sf ~/.nvm/versions/node/v24.16.0/bin/node ~/.local/bin/node
```

### Bun globals

```bash
# for vim COC bash
bun add -g bash-language-server
# for vim ALE markdown
bun add -g textlint textlint-rule-write-good @textlint-rule/textlint-rule-preset-google textlint-rule-terminology textlint-rule-spellcheck-tech-word
```

## Vim

With `bun` and `node` you can run `:PlugInstall!`

# Git repos

## `xkb-switch`

```bash
mkdir -p ~/git-repos/cpp
git clone git@github.com:sergei-mironov/xkb-switch.git ~/git-repos/cpp/xkb-switch
cd ~/git-repos/cpp/xkb-switch
mkdir build
cd build
cmake ..
make
sudo make install
sudo ldconfig
```

## `trans`

```bash
mkdir -p ~/git-repos/awk
git clone git@github.com:soimort/translate-shell.git ~/git-repos/awk/translate-shell
cd ~/git-repos/awk/translate-shell
make
sudo make install
```

## Snote

```bash
mkdir -p ~/git-repos/c
git clone git@github.com:ulcuber/snote.git ~/git-repos/c/snote
sudo pacman -S libx11 libxext libxft fontconfig freetype2
cd ~/git-repos/c/snote
make
make install
```

## Fbgrab

Portage upstream for `media-gfx/fbgrab`

```bash
mkdir -p ~/git-repos/c
git clone git@github.com:GunnarMonell/fbgrab.git ~/git-repos/c/fbgrab
cd ~/git-repos/c/fbgrab
make
sudo make install
```

## Fbterm

- <https://packages.gentoo.org/packages/app-i18n/fbterm>
- <https://aur.archlinux.org/packages/fbterm> has patches to <https://salsa.debian.org/debian/fbterm>

Portage upstream for `media-gfx/fbgrab` is too old for Artix. So using its fork

```bash
mkdir -p ~/git-repos/forks
git clone git@github.com:ulcuber/fbterm.git ~/git-repos/forks/fbterm
cd ~/git-repos/forks/fbterm
autoreconf -vfi
./configure --prefix=/usr
make
sudo make install
sudo setcap cap_sys_tty_config+ep /usr/bin/fbterm
```

## Fbida (fbi + ida)

```bash
sudo pacman -S ninja meson poppler-glib

mkdir -p ~/git-repos/c
git clone git@gitlab.com:kraxel/fbida.git ~/git-repos/c/fbida
cd ~/git-repos/c/fbida
make
cp build-meson-aspire/fbi ~/.local/bin/
```

- Neofetch

# OpenRC Console Font

## Install fonts

### Pacman

```bash
sudo pacman -S terminus-font
```

### Emerge

```bash
sudo emerge --ask media-fonts/terminus-font
```

See: <https://wiki.gentoo.org/wiki/Fonts/Console>

## Select font

Check available fonts in:

- Gentoo: `/usr/share/consolefonts`
- Artix: `/usr/share/kbd/consolefonts/ter-*`

Try in TTY: `setfont ter-c32n`

If font fits then apply it globally in `/etc/conf.d/consolefont`:

```bash
consolefont="ter-c32n"
```

If everything is OK add to runlevel:

```bash
sudo rc-update add consolefont boot
```

# Extra

- Check custom `/usr/local/bin`
- Copy your `~/.local/share/fonts`
- `battery` and `power_now` utils
- `bash-scripts`
