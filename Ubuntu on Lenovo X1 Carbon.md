# Ubuntu on Lenovo X1 Carbon

Writing down things of note after switching from a MacBook Pro (13" 2016) to Lenovo X1 Carbon (6th gen, Windows 10/Ubuntu dual boot).

* [Built-in and external displays can have different scaling under Wayland](https://askubuntu.com/a/1029559)
* If some apps don't work under Wayland yet (errors like `Gtk-WARNING **: 16:32:40.175: cannot open display: :0`), run [`xhost +SI:localuser:root`](https://askubuntu.com/a/981508)
