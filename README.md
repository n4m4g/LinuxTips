ssh, git and vim configuration
-------------------------

Contents
--------
- [Configuration](#configuration)
    - [ssh](#ssh)
    - [Git](#git)
    - [Vim](#vim)

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

### Git

    $ git config --global user.name "<name>"
    $ git config --global user.email <email>
    $ git config --global core.editor vim
    $ git config --global alias.ci commit
    $ git config --global alias.st status

### Vim

Add following commands to ~/.vimrc

    set tabstop=4
    set shiftwidth=4
    set nu
    set hlsearch
    "set expandtab
