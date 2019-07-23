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
* Install [CopyQ](https://hluk.github.io/CopyQ/), and:
  * Release the global `Win+V` shortcut from Ubuntu's "Keyboard settings", and assign it as the global shortcut for "Show/hide main window" in CopyQ
  * Change its preferences to treat mouse select/paste the same as normal copy/paste
* After installing Docker with `sudo apt install docker.io`, and `sudo usermod -a -G docker $USER`, no amount of logouts/logins fixed permission errors, but a reboot did. :shrugmoji:
* The Escape/Caps Lock remap doesn't work in Visual Studio Code without [setting `"keyboard.dispatch": "keyCode"`](https://github.com/microsoft/vscode/issues/23991#issuecomment-292336504)
