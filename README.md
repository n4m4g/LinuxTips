Linux configuration
-------------------------

Contents
--------
- [Configuration](#configuration)
    - [ssh](#ssh)
    - [git](#git)
    - [vim](#vim)
    - [apt mirror](#apt-mirror)

Configuration
-------------

### ssh

**Generate ssh key**

    $ ssh-keygen -t rsa -C <email> -b 4096
    
**Set ssh alive**

Add follow command to ~/.ssh/config

    Host *
    ServerAliveInterval 240
    
and

    $ chmod 600 ~/.ssh/config

**Set ssh alias**

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

### git

    $ git config --global user.name "<name>"
    $ git config --global user.email <email>
    $ git config --global core.editor vim
    $ git config --global alias.ci commit
    $ git config --global alias.st status

### vim

Add following commands to ~/.vimrc

    set tabstop=4
    set shiftwidth=4
    set nu
    set hlsearch
    "set expandtab

### apt mirror

    #!/bin/bash

    # replace the mirror as nchc
    LINK=`cat /etc/apt/sources.list | grep main | awk '{ print $2 }' | cut -d'/' -f3 | sed -n '3P'`
    sudo sed -i 's/${LINK}/free.nchc.org.tw/g' /etc/apt/sources.list
