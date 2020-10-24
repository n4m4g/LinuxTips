Linux tips
==========

Contents
--------
- [nvidia](#nvidia)
- [ssh](#ssh)
- [git](#git)
- [vim](#vim)
- [terminal shortcut](#terminal-shortcut)
- [docker](#docker)
- [virtualenv](#virtualenv)
- [apt tools](#apt-tools)
- [youtube-dl](#youtube-dl)
- [awk](#awk)
- [ps](#ps)
- [screen](#screen)
- [sed](#sed)
- [history](#history)

nvidia
------
    
### Remove nvidia driver

    $ sudo apt purge nvidia*
    $ sudo apt purge libnvidia*
    $ dpkg -l | grep -i nvidia | awk '{print $2}' | xargs sudo dpkg -r
    $ sudo reboot

### cuda

if you are going to install using the deb installer (which is what was installing 387 driver etc.) then you could just do:

```
sudo apt-get install cuda-toolkit-X-Y
```

instead of

```
sudo apt-get install cuda
```

and that should skip the driver install.


### cudnn

    $ tar -xzvf cudnn-*.tgz
    $ sudo cp cuda/include/cudnn.h /usr/local/cuda/include
    $ sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
    $ sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
    
### ldconfig

    $ sudo ldconfig 
    /sbin/ldconfig.real: /usr/local/cuda/lib64/libcudnn.so.x is not a symbolic link

    $ cd /usr/local/cuda/lib64
    $ ls -lha libcudnn*

    $ sudo rm libcudnn.so
    $ sudo rm libcudnn.so.x
    $ sudo ln libcudnn.so.x.y.z libcudnn.so.x
    $ sudo ln libcudnn.so.x libcudnn.so
    
    $ sudo ldconfig

### ldconfig using python3 script

```
#!/usr/bin/python3
import os
from glob import glob
from subprocess import Popen, PIPE

def main():
    VERBOSE = True
    path = '/usr/local/cuda/lib64/libcudnn.so*'
    cudnn_files = sorted(glob(path))
    assert len(cudnn_files) == 3

    # c0: libcudnn.so
    # c1: libcudnn.so.x
    # c2: libcudnn.so.x.y.z
    c0, c1, c2 = cudnn_files
    
    cmd1 = f"sudo rm {c0}"
    cmd2 = f"sudo rm {c1}"
    cmd3 = f"sudo ln {c2} {c1}"
    cmd4 = f"sudo ln {c1} {c0}"
    cmd5 = f"sudo ldconfig"

    print("Start...")
    for cmd in [cmd1, cmd2, cmd3, cmd4]:
        p = Popen(cmd.split(" "), stdout=PIPE)
        out = p.communicate()
        if out[1] != None:
            print(out[1])
            assert False
        else:
            if VERBOSE:
                print(f"\t{cmd}")

    # check no warning from ldconfig
    p = Popen(cmd5.split(" "), stdout=PIPE)
    out = p.communicate()
    assert out[0] == b''

    print("Done...")

if __name__ == "__main__":
    main()
```

Example output
```
Start...                                                                                                  
        sudo rm /usr/local/cuda/lib64/libcudnn.so                                                         
        sudo rm /usr/local/cuda/lib64/libcudnn.so.7                                                       
        sudo ln /usr/local/cuda/lib64/libcudnn.so.7.6.5 /usr/local/cuda/lib64/libcudnn.so.7               
        sudo ln /usr/local/cuda/lib64/libcudnn.so.7 /usr/local/cuda/lib64/libcudnn.so                     
Done...
```

### GPU fan speed control

    $ sudo nvidia-xconfig
    $ sudo nvidia-xconfig --cool-bits=28
    
    Restart the computer and search for NVIDIA X Server Settings in the Dash. 
    There should be an option to change fan speed under Thermal Settings.

ssh
---

### Generate ssh key

    $ ssh-keygen -t rsa -C <email> -b 4096
    
### Set ssh alive

Add follow command to ~/.ssh/config

    Host *
    ServerAliveInterval 300
    
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
        ProxyJump <proxy name>

### ssh without password

    $ ssh-copy-id <host name>

git
---

### command alias

    git config --global user.name "<name>"
    git config --global user.email <email>
    git config --global core.editor vim
    git config --global alias.ci commit
    git config --global alias.st status
    git config --global alias.co checkout
    git config --global alias.br branch
    git config --list # show git config
 
### show tracked files

    git ls-tree -r master --name-only

vim
---

### Install latest vim

    sudo apt purge vim -y && sudo apt autoremove -y
    sudo add-apt-repository ppa:jonathonf/vim
    sudo apt update
    sudo apt install vim

### Setting

Add follow command to ~/.vimrc

    set nu
    set hlsearch
    set smartindent
    set splitright

    set encoding=utf-8

    " replace tab with space
    set expandtab

    " modify display width of tab
    set tabstop=4

    " modify display width of indent of tab
    set shiftwidth=4

    " the number of context lines 
    " you would like to see above and below the cursor
    set scrolloff=5

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
    
### Add user to docker group

    $ sudo usermod -aG docker $USER

### Create container

    $ docker run -it [image name] [shell]
    $ docker run -it ubuntu bash

### Enter the container

    $ docker exec -it <container id> bash
    $ docker exec -it c357 bash

virtualenv
----------

### Install virtualenv

    $ pip3 install --user virtualenv
    
### Create a virtualenv

    $ python3 -m virtualenv -p python3 [name]

apt tools
---------

    $ sudo apt install ncdu # NCurses Disk Usage
    $ sudo apt install pragha # support m4a format
    $ sudo apt install blueman # gui bluetooth manager
    $ sudo apt install pavucontrol # audio fixing

youtube-dl
----------

    $ sudo apt install aria2 -y
    $ sudo snap install youtube-dl
    $ snap run youtube-dl --no-playlist -F [url]
    # increase download speed with arguments below
    # --external-downloader aria2c --external-downloader-args "-x 8 -s 8 -k 1M" 

awk
---

overall memory usage  

    $ free -h | awk '/^Mem:/ {print $3 "/" $2}'
    2.6G/15G

ps
--

cpu usage of processes

    $ ps axch -o cmd:15,%cpu --sort=-%cpu | head
    chrome          32.4
    chrome          17.6
    chrome          11.7
    gnome-shell      7.6
    chrome           7.2
    chrome           2.4
    chrome           1.6
    Xorg             1.5
    dbus-daemon      0.9
    irq/31-nvidia    0.8
    
mem usage of processes

    $ ps axch -o cmd:15,%mem --sort=-%mem | head
    chrome           3.9
    chrome           2.1
    chrome           2.0
    gnome-shell      1.9
    chrome           1.2
    gnome-shell      1.2
    mysqld           1.0
    chrome           1.0
    gnome-software   1.0
    chrome           0.9

screen
------

    $ screen
    Cannot make directory '/var/run/screen': Permission denied  
  
The directory /var/run/screen/ is the socket directory for screen.  
Fortunately, screen reads a environment variable SCREENDIR to get an alternative socket directory.  

    $ mkdir ~/.screen && chmod 700 ~/.screen

Put this line into ~/.bashrc  

    export SCREENDIR=$HOME/.screen

sed
---

substitude

    $ sed 's/http/HTTP/g' [file]
    # replace http with HTTP in each line and printout
    # without g mean replace once per line
    # command above will NOT motify the file
    
    $ sed -i 's/http/HTTP/g' [file]
    # -i: inplace
    # command above will motify the file

delete and print

    $ sed '/http/ d' [file]
    # delete the line has http and printout
    
    $ sed '/http/ p' [file]
    # find the line has http and printout
    
    # sed '/^$/ d' [file]
    # delete blank line
    
multiple commands

    $ sed 's/http/HTTP/g;/^$/ d' [file]
    # use ';' to concatenate commands

history
-------

history command with timestamp

    $ export HISTTIMEFORMAT='%F %T '
