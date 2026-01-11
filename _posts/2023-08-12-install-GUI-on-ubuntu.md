Ubuntu Server 默认是没有 GUI 的，只有命令行。

要有 GUI，至少要装一个桌面环境。常见的有GNOME、XFCE、KDE等


这里选择XFCE：
```bash
sudo apt update
sudo apt install xubuntu-desktop
```

到这里为止，GUI 已经安装完成了。


## 远程连接

#### 方式1：xrdp
```
sudo apt install xrdp
sudo systemctl enable xrdp
sudo systemctl restart xrdp
```

然后就可以用win自带的远程桌面连接了。

#### 方式2：vnc

```bash
sudo apt-get install vnc4server tightvncserver

#向xsession中写入xfce4-session 
echo “xfce4-session” >~/.xsession
```