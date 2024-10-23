##### Enter Files

select `Home` tab

##### Create File Hidding Config

right-click and select `Open In Terminal` menu

input command below

``````bash
sudo touch .hidden
sudo open .hidden
``````

##### Edit File Hidding Rule

something like this

``````properties
snap
Library
Desktop
Public
Downloads
Pictures
Music
Videos
Documents
Templates
``````

##### Refresh UI

press `Ctrl+H` to `hide/show` hidden files and refresh rules

##### Attention

`.hidden` file only apply to current directory, not for sub directories

