# VNC
 Virtual Network Computing (VNC) is a graphical desktop sharing system that uses the Remote Frame Buffer protocol (RFB) to remotely control another computer. It transmits the keyboard and mouse events from one computer to another, relaying the graphical screen updates back in the other direction, over a network.

## Server
### Ubuntu
###### Install Ubuntu Desktop VNC and other dependencies for Unity, etc

`sudo apt-get update && sudo apt-get upgrade`

`sudo apt-get install ubuntu-desktop gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal`

`sudo apt-get install vnc4server`

#### Setup VNC Configuration

Depending on what user runs the following commands this is the user that will be responsible for running the server.

###### Setup Password and Initialize Configuration
`vncserver :1`

This command will create the `$HOME/.vnc/xstartup` file which is a bash script. It is run whenever starting the VNC server (under this user).  
VNC servers always start on 5901 and increment. The `:1` specifies which display to run the VNC server on.

###### Rewrite Configuration to start Unity

`vim $HOME/.vns/xstartup`

Paste the following into the above file
```
#!/bin/sh
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
x-window-manager &

gnome-panel &
gnome-settings-daemon &
metacity &
nautilus &
```

###### Restart VNC

`vncserver :1` :tada:

Now is a good time to connect with your VNC client to test Ubuntu with Unity is running correctly.

###### Starting VNC Server on boot


## Client
