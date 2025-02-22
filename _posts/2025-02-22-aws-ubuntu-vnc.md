---
mathjax: true
title:  "aws ubuntu server安装vnc"
categories: 
  - linux
---
# ec2上
```
sudo su
apt install xfce4
apt install tightvncserver
shutdown -r now #(不确定需不需要重启)
重连后用ubuntu身份就行
vncserver :1 #(生成默认配置)
vncserver -kill :1
vim ~/.vnc/xstartup
```
写入下边的内容
```
#!/bin/sh

# Uncomment the following two lines for normal desktop:
unset SESSION_MANAGER
# exec /etc/X11/xinit/xinitrc
unset DBUS_SESSION_BUS_ADDRESS
autocutsel -fork
startxfce4 &

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
# x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
# x-window-manager &
```
再次启动 `vncserver :1`
# 在本地
```
apt install gvncviewer
ssh -L 5901:localhost:5901 远程ec2地址
在另一个cmd里：
gvncviewer :1
```
