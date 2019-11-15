# Ubuntu on Lenovo X1 Carbon

## 2nd iteration

* Start from a "Minimal installation" instead
  * Encrypt whole disk (obviously)
  * Choose to log in automatically (because of the above)
* Install some software: `sudo apt update && sudo apt install -y gnome-tweak-tool gpaste docker.io konsole fdupes vim gimp direnv tig curl adb ffmpeg exiftool graphicsmagick gnome-sushi python3-pip awscli kazam vlc virtualbox aha graphviz tree gthumb ffmpegthumbnailer`
* Swap Esc & Caps Lock: [`dconf write /org/gnome/desktop/input-sources/xkb-options "['caps:swapescape']"`](https://askubuntu.com/questions/363346/how-to-permanently-switch-caps-lock-and-esc)
* Gnome Tweaks:
  * Appearance: Switch to `Yaru-dark` theme
  * Extensions: Hide built-in desktop icons
  * Keyboard & Mouse: Switch touchpad right-click emulation to two-finger-click
  * Startup Applications: Firefox, Konsole
  * Top Bar: Show clock seconds; show week numbers; show battery percentage
  * Windows: Disable Attach Modal Dialogs
* Settings:
  * Mouse & Touchpad:
    * Mouse: Enable Natural Scrolling
    * Touchpad: Disable Tap to Click
  * Keyboard:
    * Remove `Super+V` keybinding (will be used for GPaste, to match Windows)
    * "Copy a screenshot of an area to clipboard": `Print`
    * "Show the notification list": `Super+N`
    * "Home folder": `Super+E`
  * Dock:
    * Auto-hide the Dock
  * Notifications:
    * Disable lock-screen notifications
* GPaste:
  * Use GNOME Tweaks to add GPaste to startup apps, otherwise it won't be recording after a reboot
  * Add custom keyboard shortcut (from Ubuntu's "Keyboard Settings") for `Win+V` to run command `/usr/lib/x86_64-linux-gnu/gpaste/gpaste-ui`(found this by looking at its `/usr/share/applications/org.gnome.GPaste.Ui.desktop` file)
  * If the daemon isn't running (e.g. it hangs on startup, see https://github.com/Keruspe/GPaste/issues/156), `rm -rf .local/share/gpaste` clears history file
* Docker:
  * `sudo usermod -a -G docker $USER`
  * No amount of logouts/logins fixed permission errors, but a reboot did. `¯\_(ツ)_/¯`
* [Change the emoji-key to `Ctrl + Alt + E`](https://askubuntu.com/a/1159087) via `ibus-setup`, so it doesn't clash with default VSC keybindings 
* [Disable avahi-daemon](https://askubuntu.com/a/339709), so it doesn't spam "Network service discovery disabled. Your current network has a .local domain, which is not recommended and incompatible with the Avahi network service discovery. The service has been disabled." notifications. That is, set `AVAHI_DAEMON_DETECT_LOCAL=0` in `/etc/default/avahi-daemon`.
* Visual Studio Code:
  * https://code.visualstudio.com/download
* Dropbox:
  * https://www.dropbox.com/install-linux
  * Link dotfiles from `~/Dropbox/.dotfiles`
* Node:
  * https://github.com/nvm-sh/nvm#install--update-script
* Set screen timeout to 30 min: `gsettings set org.gnome.desktop.session idle-delay $((30*60))`
* Google Chrome:
  * `wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb`
  * `sudo dpkg -i google-chrome-stable_current_amd64.deb`
* [Fix screen tearing](https://news.ycombinator.com/item?id=21331723):
  * `echo -e 'Section "Device"\n  Identifier "Intel Graphics"\n  Driver "intel"\n  Option "TearFree" "true"\nEndSection' > /etc/X11/xorg.conf.d/20-intel.conf`
  * ...though honestly I'm not yet 100% sure this had any effect

## 1st iteration

Writing down things of note after switching from a MacBook Pro (13" 2016) to Lenovo X1 Carbon (6th gen, Windows 10/Ubuntu dual boot).

* [Built-in and external displays can have different scaling under Wayland](https://askubuntu.com/a/1029559), so make the switch
* If some apps don't work under Wayland yet (errors like `Gtk-WARNING **: 16:32:40.175: cannot open display: :0`), run [`xhost +SI:localuser:root`](https://askubuntu.com/a/981508)
* To swap Esc & Caps Lock, run [`dconf write /org/gnome/desktop/input-sources/xkb-options "['caps:swapescape']"`](https://askubuntu.com/questions/363346/how-to-permanently-switch-caps-lock-and-esc)
* With `sudo apt install -y gnome-tweak-tool`:
  * Switch to `Yaru-dark` theme
  * Hide built-in desktop icons
  * Switch touchpad right-click emulation to two-finger-click
  * Set Mouse acceleration profile to "Adaptive"
* ~Install [CopyQ](https://hluk.github.io/CopyQ/), and:~
  * ~Release the global `Win+V` shortcut from Ubuntu's "Keyboard settings", and assign it as the global shortcut for "Show/hide main window" in CopyQ~
  * ~Change its preferences to treat mouse select/paste the same as normal copy/paste~
  * For whatever reason, it seemed impossible to get `Win+V` working in the built-in Terminal, even if it worked in all other apps `¯\_(ツ)_/¯`
* Install `gpaste`
  * ~~Set it to synchronize mouse-selects with clipboard~~ (never mind; this makes working in an IDE pretty unbearable)
  * Add custom keyboard shortcut (from Ubuntu's "Keyboard Settings") for `Win+V` to run command `/usr/lib/x86_64-linux-gnu/gpaste/gpaste-ui`(found this by looking at its `/usr/share/applications/org.gnome.GPaste.Ui.desktop` file)
  * Use GNOME Tweaks to add GPaste to startup apps, otherwise it won't be recording after a reboot
* After installing Docker with `sudo apt install docker.io`, and `sudo usermod -a -G docker $USER`, no amount of logouts/logins fixed permission errors, but a reboot did. `¯\_(ツ)_/¯`
* The Escape/Caps Lock remap doesn't work in Visual Studio Code without [setting `"keyboard.dispatch": "keyCode"`](https://github.com/microsoft/vscode/issues/23991#issuecomment-292336504)
* ~~Install [AppImageLauncher](https://github.com/TheAssassin/AppImageLauncher/releases/tag/v1.3.1)~~
  * DO NOT install it after all. In addition to not managing to create working `.desktop` files for any AppImages I tried, its launcher app seems to break e.g. the [Etcher](https://www.balena.io/etcher/) image so that it gives random errors when trying to flash something.
  * So just use `chmod` and the AppImage file from bash, like a proper Linux user. `¯\_(ツ)_/¯`
* Install `konsole`
  * Config files are e.g. `.config/konsolerc` & `.local/share/konsole/DarkPastels.colorscheme`
  * [Hide menu bar by default](https://unix.stackexchange.com/a/336100)
  * [Tab icons can be changed](https://laanwj.github.io/2011/4/7/changing-tab-icons-in-konsole)
  * [Try to make links behave sensibly](https://www.reddit.com/r/kde/comments/3qfa1k/is_there_any_way_to_make_kdeopen_not_reasolve/)... [unsuccessfully](https://unix.stackexchange.com/questions/525031/how-to-set-the-default-browser-in-kde)
* Install `fdupes` for finding duplicate files (e.g. photos)
* [Change the emoji-key to `Ctrl + Alt + E`](https://askubuntu.com/a/1159087) via `ibus-setup`, so it doesn't clash with default VSC keybindings 
* [Disable avahi-daemon](https://askubuntu.com/a/339709), so it doesn't spam "Network service discovery disabled. Your current network has a .local domain, which is not recommended and incompatible with the Avahi network service discovery. The service has been disabled." notifications
* ~Switch to KDE~
  * `sudo apt install tasksel && sudo tasksel install kubuntu-desktop`
  * Select `sddm` when prompted
  * Use `sudo dpkg-reconfigure gdm3` to switch back
