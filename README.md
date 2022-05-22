Linux tips
==========

Contents
--------
- [bash array](#bash-array)
- [sed](#sed)
- [unzip](#unzip)
- [grep](#grep)
- [find](#find)
- [batch shell script](#bash-shell-script)
- [parallel](#parallel)
- [remove efi partition](#remove-efi-partition)
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

bash array
----------
Append element to array  
Elements in double quote is treat as an element in for loop  
<a href="https://linuxhint.com/bash_append_array/">link0</a> /
<a href="https://www.cyberciti.biz/faq/bash-for-loop-array/">link1</a> /
<a href="https://stackoverflow.com/questions/18383291/loop-over-set-of-strings-containing-spaces">link2</a>
```
x="a b"
y="c d"
arr=()
arr+=("$x" "$y")
for i in "${arr[@]}"
do
  echo "$i"
done

# a b
# c d
```

Show length of array
```
echo "${#arr[@]}"
```

iterate array
```
for i in "${arr[@]}"
do
  echo "$i"
done
```

iterate array by index
```
for (( idx=0; idx<"${#arr[@]}"; idx++ ))
do
  echo "$idx, ${arr[$idx]}"
  or
  printf "%d, %s\n" "$idx" "${arr[$idx]}"
done
```

sed
---
& is the placeholder for the match  
<a href="https://askubuntu.com/questions/76808/how-do-i-use-variables-in-a-sed-command">link</a>
```
sed 's/pat/\"&\"/' [file]
# pat -> "pat"
```

unzip
-----
Decompressing multiple files at once  
<a href="https://askubuntu.com/questions/431478/decompressing-multiple-files-at-once">link</a>
```
# search *.zip at current directory, unzip the .zip only 10 parallel processes at a time
find . -maxdepth 1 -name '*.zip' -print0 | xargs -0 -I {} -P 10 unzip {}
```

grep
----
<a href="https://www.thegeekstuff.com/2011/10/grep-or-and-not-operators/">link0</a> /
<a href="https://www.thegeekstuff.com/2011/01/advanced-regular-expressions-in-grep-command-with-10-examples-%E2%80%93-part-ii/">link1</a> /
<a href="https://www.thegeekstuff.com/2009/03/15-practical-unix-grep-command-examples/">link2</a>

Grep OR / Grep AND / Grep NOT
```
- Grep OR
grep "pat0\|pat1" [file]
or
grep -e "pat0" -e "pat1" [file]

- Grep AND
grep "pat0.*pat1\|pat1.*pat0" [file]

- Grep NOT
grep -v "pat" [file]
```

Character class expression
```
[[:digit:]] -> [0-9]
[[:alnum:]] -> [0-9A-Za-z]
[[:alpha:]] -> [A-Za-z]
[[:blank:]] -> [space or tab]

grep "[[:digit:]]\+" [file]
```

Match occurences
```
- M to N occurences ({m,n})
grep "pat\{m,n\}" [file]

- Exact M occurence ({m})
grep "pat\{m\}" [file]

- M or more occurences ({m,})
grep "pat\{m,\}" [file]
```
Word boundary (\b)
```
grep "\bpat\b" [file]
```

Displaying lines before/after/around the match using grep -A, -B and -C
```
grep -A m "pat" [file]
# show m lines after match

grep -B m "pat" [file]
# show m lines before match

grep -C m "pat" [file]
# show m lines before & after match
```

Counting the number of matches using grep -c
```
grep -c "pat" [file]
```

Show line number while displaying the output using grep -n
```
grep -n "pat" [file]
```

Display only the file names which matches the given pattern using grep -l
```
grep -l "pat" [file]
```

find
----
Find exec multiple commands syntaxes  
<a href="https://www.howtouselinux.com/post/linux-find-exec-examples-advanced-part">link</a>

```
- find [path] -type f -exec [cmd] {} \;
# cmd result0; cmd result1 ; ...

- find [path] -type f -exec [cmd] {} \+
# cmd result0 result1 ... ;
```

bash shell script
-----------------

1. $()

    () means the start and end of the sub-shell  
    $ means the variable  
    $() means run the command and return the result  

    $ uname -r  
    5.4.0-72-generic

    $ version=$(uname -r)  
    $ echo $version  
    5.4.0-72-generic

    $ version=\`uname -r\`  
    $ echo $version  
    5.4.0-72-generic

2. ${} could remove or substitute the string in the variable

    $ echo ${version}  
    5.4.0-72-generic

3. $(()) could calculate integer operation

    $ echo $((15*3))  
    45

    $ echo {15.5*3} | bc  
    46.5

4. ${} for string substitute
    
    $ version=\`uname -r\`  
    5.4.0-72-generic
    
    get length of string  
    $ echo ${#version}  
    16
    
    slicing, first number is start index, second number is amount to slice.  
    $ echo ${version: 0:3}  
    5.4

    If second number is negative, the second number is the end index.  
    $ echo ${version: 3:-3}  
    .0-72-gene

parallel
--------

Running 4 cmds with args simultaneously

    parallel -j 4 [cmd] [args] < data.txt

For example

    parallel -j 4 youtube-dl {} < url.txt  
    
    or  
    
    cat url.txt | parallel -j 4 youtube-dl {}
    
Convert audio to mp3

    find . -name '*.m4a' | parallel '[ ! -f {.}.mp3 ] && ffmpeg -i {} -acodec libmp3lame -aq 2 {.}.mp3'

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

    git config --global user.name [name]
    git config --global user.email [email]
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

    $ sudo groupadd docker
    $ sudo usermod -aG docker $USER
    $ newgrp docker

### Create container

    $ docker run -it [image name] [shell]
    $ docker run -it ubuntu bash

### Enter the container

    $ docker exec -it <container id> bash
    $ docker exec -it c357 bash

### Remove "none" docker image

    $ docker rmi -f $(docker images --filter "dangling=true" -q)
    
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

    wsl --shutdown
    notepad "$env:USERPROFILE/.wslconfig"

```
[wsl2]
memory=4GB
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
