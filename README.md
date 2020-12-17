Linux tips
==========

Contents
--------
- [chromium](#chromium)
- [parallel](#parallel)
- [bashrc](#bashrc)
- [remove efi partition](#remove-efi-partition)
- [tensorflow](#tensorflow)
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
- [ffmpeg](#ffmpeg)
- [xauth](#xauth)
- [wsl2 mem](#wsl2-mem)
- [X11 forwarding on WSL2](#X11-forwarding-on-WSL2)


chromium
--------

Hardware acceleration

1. Follow the <a href="https://www.linuxuprising.com/2018/08/how-to-enable-hardware-accelerated.html?m=1&fbclid=IwAR0-2Nne7eDA4393Z5W4rz9mHOMp8DeZ8AMhQJEPBM1gBatMNLVe6FgeOaM">How To Enable Hardware Acceleration In Chromium On Ubuntu Or Linux Mint (VA-API Patched PPA Builds)</a>  

    The tutorial will guide you install VA-API Patched Chromium

2. chrome://flags  

3. Enable follow settings
    - Hardware-accelerated video decode
    - Override software rendering list
    - GPU rasterization
    - Out of process rasterization using DDLs
    - Zero-copy rasterizer
    
4. In terminal

    chromium --use-gl=desktop &

parallel
--------

Running 4 cmds with args simultaneously

    parallel -j 4 [cmd] [args] < data.txt

For example

    parallel -j 4 youtube-dl {} < url.txt  
    
    or  
    
    cat url.txt | parallel -j 4 youtube-dl {}
    
bashrc
------

```
# alias
PLOTS=`[[ -d ./runs ]] && ls -tr ./runs | awk 'END { print }'`
alias tb="tensorboard --logdir=runs/$PLOTS --port "
alias cp="rsync -aP --info=progress2 "
# export
export PATH='/usr/local/cuda/bin':$PATH
export LD_LIBRARY_PATH='/usr/local/cuda/lib64':$LD_LIBRARY_PATH
```

remove efi partition
--------------------

1. run diskpart as administrator  
    
    diskpart
    
2. show drives

    list disk
    
3. select disk that hosts windows OS

    sel disk #
    
4. list volumes

    list vol
    
5. select volumne with filesystem FAT32

    sel vol #
    
6. assign a letter to selected volume

    assign letter=Z:

7. exit diskpart

    exit
    
8. cd select volume

    Z:

9. move to EFI directory

    cd EFI
    
10. show entries and remove ubuntu directory

    dir  
    rmdir /S ubuntu

11. run diskpart as administrator  
    
    diskpart
    
12. show drives

    list disk
    
13. select disk that hosts windows OS

    sel disk #
    
14. list volumes

    list vol
    
15. select volumne with filesystem FAT32

    sel vol #
    
16. remove letter

    remove letter=Z:

tensorflow
----------

<a href="https://www.tensorflow.org/install/source#tested_build_configurations">Official compatible table</a>  

tensorflow-gpu==2.3.1  
cuda==10.1  
cudnn==7.6.5  

### Install tensorflow-gpu

    $ pip install tensorflow-gpu
 
### Check require cuda version

```
>>> import tensorflow as tf
2020-11-05 10:42:50.498229: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'libcudart.so.10.1'; dlerror: libcudart.so.10.1: cannot open shared object file: No such file or directory
2020-11-05 10:42:50.498262: I tensorflow/stream_executor/cuda/cudart_stub.cc:29] Ignore above cudart dlerror if you do not have a GPU set up on your machine.
>>>
```
Warnings that "Could not load dynamic library 'libcudart.so.10.1'" means require cuda version 10.1

nvidia
------
    
### Remove nvidia driver

    $ sudo apt purge nvidia*
    $ sudo apt purge libnvidia*
    $ dpkg -l | grep -i nvidia | awk '{print $2}' | xargs sudo dpkg -r
    $ sudo reboot

### cuda

if you are going to install using the deb installer then you could just do:

```
sudo apt-get install cuda-toolkit-X-Y
```

instead of

```
sudo apt-get install cuda
```

and that should skip the driver install.

### cudnn

<a href="https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html#install-linux">Installation Guide</a>

    $ tar -xzvf cudnn-*.tgz
    $ sudo cp cuda/include/cudnn.h /usr/local/cuda/include
    $ sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
    $ sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

### environment variable setting

Add follow lines to ~/.bashrc

    export  PATH='/usr/local/cuda/bin':$PATH
    export  LD_LIBRARY_PATH='/usr/local/cuda/lib64':$LD_LIBRARY_PATH

### ldconfig

    $ sudo ldconfig 
    /sbin/ldconfig.real: /usr/local/cuda/lib64/libcudnn.so.x is not a symbolic link
    $ sudo ln -sf /usr/local/cuda/lib64/libcudnn.so.x.y.z /usr/local/cuda/lib64/libcudnn.so.x

### ldconfig using python3 script

```
#!/usr/bin/python3
import re
import os
from glob import glob
from subprocess import Popen, PIPE

def main():

    # get ldconfig warning
    cmd = "sudo ldconfig"
    p = Popen(cmd.split(" "), stdout=PIPE, stderr=PIPE)
    out, err = p.communicate()
    err = err.decode('utf-8')

    # /sbin/ldconfig.real: /usr/local/cuda-10.1/targets/x86_64-linux/lib/libcudnn_ops_train.so.8 is not a symbolic link
    files = re.findall(r'/sbin/ldconfig.real: (.*) is not a symbolic link', err)

    if files == []:
        print("ldconfig is clean")
    else:
        print(f"Found {files}, bulid soft link for each file")
        for f in files:
            if not os.path.exists(f):
                print(f'Failed parsing: {f}')
                assert False
            tmp_f = glob(f+".*.*")[0]
            if not os.path.exists(tmp_f):
                assert False
            cmd = f"sudo ln -sf {tmp_f} {f}"
            print(cmd)
            p = Popen(cmd.split(" "), stdout=PIPE, stderr=PIPE)

        print('Done')

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

    set hlsearch
    set smartindent
    set splitright
    set encoding=utf-8

    " show hybrid line number
    set number relativenumber
    
    " replace tab with space
    set expandtab

    " modify display width of tab
    set tabstop=4

    " modify display width of indent of tab
    set shiftwidth=4

    " the number of context lines 
    " you would like to see above and below the cursor
    set scrolloff=5
    
    nmap<C-j> <S-_>i#<Space><Esc>
    nmap<C-k> <S-_>xx<Space><Esc>
    vmap<C-j> <S-i>#<Space><Esc>

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

### Download

    pip install youtube-dl
    # increase download speed with arguments below
    # --external-downloader aria2c --external-downloader-args "-x 8 -s 8 -k 1M" 
    
### Config

    echo "-f 'bestvideo[height<=1440]+bestaudio[ext=m4a]'" >> ~/.config/youtube-dl/config

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
    
ffmpeg
------

convert m4a to mp3  
${foo%.m4a} means remove most right .m4a found

    $ for foo in *.m4a; do ffmpeg -i "$foo" -acodec libmp3lame -aq 2 "${foo%.m4a}.mp3"; done

xauth
-----

/usr/bin/xauth:  file /home/$USER/.Xauthority does not exist

    $ chown $USER:$USER -R /home/$USER/

wsl2 mem
--------

Limit memory assigned to WSL2 VM

```
[wsl2]
memory=8GB
swap=0
localhostForwarding=true
```
  

Open linux home directory in windows explorer <a href="https://docs.microsoft.com/zh-tw/windows/wsl/compare-versions">[link]</a>  

```
# In wsl2
cd
powershell.exe /c start .
```

X11 forwarding on WSL2
----------------------

<a href="https://stackoverflow.com/questions/61110603/how-to-set-up-working-x11-forwarding-on-wsl2">ref1</a>
<a href="https://github.com/cascadium/wsl-windows-toolbar-launcher#firewall-rules">ref2</a>

1. Install <a href="https://sourceforge.net/projects/vcxsrv/">VcXsrv Windows X Server</a>  

    - At Extra settings, enable public access for your X server by clicking "Disabling Access Control"  

    - When "Windows Security Alert" pop up, only select "Private networks, ..."  

2. Setup Windows Defender Firewall rules

    - Search "Windows Defender Firewall"
    
    - Advanced settings
    
    - Inbound Rules and disabling block rule for TCP on the Public Network.
    
    - New Rule..., select TCP port 6000 and select defaults
    
    - Refresh and scroll to the recently created name, right click and go to properties
    
    - Scope, go to Remote IP address, Select These IP addresses and add in 172.16.0.0/12

3. Add the following to your ~/.bashrc

```
export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):0
export LIBGL_ALWAYS_INDIRECT=1
```

4. Restart wsl2
