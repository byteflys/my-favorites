##### Solutions

type sudo and password repeatly is a terrible thing

especially you are handling lots of command-line work

there are several ways to skip typing sudo and password

choose your own way after considering security issue cautionsly

##### Skip Password for Sudo

open sudo config file

``````bash
sudo visudo
``````

edit sudo script

``````bash
root    ALL=(ALL:ALL) NOPASSWD:ALL
%admin  ALL=(ALL) NOPASSWD:ALL
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
``````

then you don't need to type password after use sudo command

##### Directly Delete Password

``````bash
sudo passwd -d easing
``````

then you don't need to type password any more, as there is totally no password

even if outside command line, you won't type it anywhere

##### Use Sudo for All Commands

``````bash
sudo -s
``````

this command means using super user doing all things below

then you don't need to type `sudo` after this, but just for current terminal window

##### Grant Permissions to All User

``````bash
sudo chmod -R 777 /home/<username>
``````

this can help to decrease usage of sudo in a certain degree, when visit files under home dir

do not try to chmod for system dirs, such as `/` `/etc` `/root` `/usr` , which may cause unrecoverable damage to system

as linux will check permission for key files, if fails, all things will not work
