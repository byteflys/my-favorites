##### Edit User Dirs

open dir file, and remove unused dirs

``````bash
sudo open ~/.config/user-dirs.dirs
``````

``````properties
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Downloads"
``````

##### Set Dirs Unmodifiable

dirs may be modified by system program, disable it

``````bash
sudo open /etc/xdg/user-dirs.conf
``````

``````properties
enabled=False
filename_encoding=UTF-8
``````

##### Reboot to Take Effect



