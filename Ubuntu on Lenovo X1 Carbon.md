# Ubuntu on Lenovo X1 Carbon

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
