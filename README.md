Linux tips
-------------------------

Contents
--------

- [ssh](#ssh)
    - [Generate ssh key](#generate-ssh-key)
    - [Set ssh alive](#set-ssh-alive)
    - [Set ssh alias](#set-ssh-alias)
    - [ssh without password](#ssh-without-password)
- [git](#git)
- [vim](#vim)
- [apt mirror](#apt-mirror)
- [terminal shortcut](#terminal-shortcut)
- [docker](#docker)
    - [Show docker images](#show-docker-images)
    - [Show all containers](#show-all-containers)
    - [Enter the container](#enter-the-container)

ssh
---

### Generate ssh key

    $ ssh-keygen -t rsa -C <email> -b 4096
    
### Set ssh alive

Add follow command to ~/.ssh/config

    Host *
    ServerAliveInterval 240
    
and

    $ chmod 600 ~/.ssh/config

### Set ssh alias

Add following commands to ~/.ssh/config

    # for proxy
    Host <alias proxy name>
        Hostname <proxy ip>
        User <uname>
        ForwardX11 yes
        ForwardX11Trusted yes
        
    # for server
    Host <alias server name>
        Hostname <server name>
        User <uname>
        ProxyCommand ssh <proxy name> -W %h:%p

### ssh without password

    $ scp ~/.ssh/id_rsa.pub <server_hostname>:~/.ssh/<new_name.pub>
    $ ssh <server_hostname>
    $ cat ~/.ssh/<new_name.pub> >> ~/.ssh/authorized_keys

git
---

    $ git config --global user.name "<name>"
    $ git config --global user.email <email>
    $ git config --global core.editor vim
    $ git config --global alias.ci commit
    $ git config --global alias.st status

vim
---

Add following commands to ~/.vimrc

    set tabstop=4
    set shiftwidth=4
    set nu
    set hlsearch
    "set expandtab

apt mirror
----------

Save the following commands as foo.sh

    #!/bin/bash

    # replace the mirror as nchc
    LINK=`cat /etc/apt/sources.list | grep main | awk '{ print $2 }' | cut -d'/' -f3 | sed -n '3P'`
    sudo sed -i 's/${LINK}/free.nchc.org.tw/g' /etc/apt/sources.list

and

    $ source foo.sh

terminal shortcut
-----------------

- ctrl + U: Clear **whole** command line.
- ctrl + W: Clear words to the **left** of the cursor
- ctrl + K: Clear words to the **right** of the cursor
- ctrl + L: Clear words **above** the cursor.

docker
------

### Show docker images

    $ docker images

### Show all containers

    $ docker ps -a

### Enter the container

    $ docker exec -e LANG=en_US.UTF-8 -e LC_ALL=en_US.UTF-8 --user <uname> -it <container id> bash

ncdu
----

### List recursive file sizes of files and directories in a directory

    $ ncdu
