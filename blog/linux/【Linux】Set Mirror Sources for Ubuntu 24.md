##### edit config file

``````bash
sudo nano /etc/apt/sources.list.d/ubuntu.sources
``````

##### high-speed mirror

``````
Types: deb
URIs: https://mirrors.huaweicloud.com/repository/ubuntu/
Suites: noble noble-security noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
``````

##### official source

``````
Types: deb
URIs: http://cn.archive.ubuntu.com/ubuntu/
Suites: noble noble-security noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
``````

##### compat older version librarys

``````
Types: deb
URIs: http://cn.archive.ubuntu.com/ubuntu/
Suites: jammy jammy-security jammy-updates jammy-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
``````

##### update

``````
sudo apt update
sudo apt upgrade
``````



###### 
