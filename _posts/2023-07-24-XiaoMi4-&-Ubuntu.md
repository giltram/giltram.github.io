这是捯饬小米4和Ubuntu的踩坑记录

/* */的内容都是我瞎说的



## 零、常用命令

### 0.信息记录

Ubuntu版本：16.04 TLS

处理器：ARMv7 Processor rev 1 (v7l)



### 1.关闭/启动GUI

```bash
sudo service lightdm stop
sudo service lightdm start
```







## +∞、报错和解决

### 0.无法ssh连接

应该是什么安全设置禁止使用密码登录，所以只能采用ssh秘钥进行无密码登录。

首先登录到服务器后，在命令行输入命令用来生成秘钥：

```bash
 ssh-keygen
```

其中第一步是确认保存秘钥的位置，一般使用默认的位置即：

```text
/home/timer/.ssh/id_rsa #此处timer为用户名
```

第二步是为秘钥设置一个密码：如果输入的话以为只即使被人有你的秘钥没有你的密码也是无法登录你的服务器的这样会比较保险但也比较繁琐，直接回车表示不设置密码； 第三步是确认密码，如果没有设置的话可以直接回车。后面的信息是给出秘钥保存的位置 和秘钥信息。 最终我们可以看到在 `/home`目录下的 `time`目录中生成了一个隐藏目录 `.ssh`。里面包含两个密钥文件，id_rsa 为私钥，id_rsa.pub 为公钥。

配置SSH，打开秘钥登录功能：

使用vi编辑 /etc/ssh/sshd_config 文件

```text
sudo vi /etc/ssh/sshd_config
```

然后按 `i`进入编辑模式，在空白位置输入：

```text
RSAAuthentication yes
PubkeyAuthentication yes
```

注意此处需要留意root 用户能否通过 SSH 登录， 如果需要进行如下设置：

```text
PermitRootLogin yes
```

此处便已经设置好了使用秘钥登录了，但是如果需要禁用密码登录可以进行如下设置：

```text
PasswordAuthentication no
```

这一步最好是在完成前面的全部设置，然后能够用秘钥登录的前提下设置，不然又不能用密码登录，秘钥又没法登录就尴尬了。 编辑完文本后按 `ESC`，`:wq` 保存文件并退出。

最后，输入如下指令重启 SSH 服务：

```text
service sshd restart
```

然后用ssh软件连接时，把私钥文件传上去即可。



### 1.

```bash
E: The repository 'https://mirrors.tuna.tsinghua.edu.cn/ubuntu focal Release' does not have a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
E: The repository 'https://mirrors.tuna.tsinghua.edu.cn/ubuntu focal-updates Release' does not have a Release file.
......
```

换成阿里云的源即可（https://developer.aliyun.com/mirror/ubuntu）

### 2.

```bash
Reading state information... Done
You might want to run 'apt-get -f install' to correct these.
The following packages have unmet dependencies:
 bleachbit : Depends: gir1.2-gtk-3.0 but it is not installable
             Depends: gir1.2-notify-0.7 but it is not installable
E: Unmet dependencies. Try using -f.
```

运行

```
apt -f -y install
```

### 3.

```bash
E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?
```

首先查看是否有apt-get这个程序在运行`ps aux|grep apt-get`

如果发现存在这样的程序在运行那么就kill掉，没有就继续往下执行

直接删除锁文件：

```bash
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/lib/dpkg/lock
```

### 4. no enough free space

```bash
You don‘t have enough free space in /var/cache/apt/archives
```

网上简单粗暴的解决办法通常是这个：

```bash
sudo apt autoclean
sudo apt autoclean
sudo apt autoremove
```

但是不管用啊。由于本身硬盘空间只有16G，所以不管用，唯一的办法就是给`var/cache/apt/archives`创建一个软链接。

先用`df -h`查看存储使用情况：

```bash
root@ubuntu-phablet:/dev# df -h
Filesystem                      Size  Used Avail Use% Mounted on
......
/dev/mmcblk0p25                  13G  2.2G   11G  18% /userdata
/dev/loop0                      2.0G  1.8G   36M  99% /
/dev/loop1                      168M  164M  3.2M  99% /android/system
......
```

可以看到/目录下只有36M的空间，但是在/userdata下还有11G/* 真够多的。。。。*/，于是我们就可以在/userdata目录下创建软链接。命令如下:

```bash
sudo mkdir -p "/userdata/partial"
sudo rm -rf /var/cache/apt/archives
sudo ln -s "/userdata" /var/cache/apt/archives
```

然后再执行sudo apt-get upgrade就不会报错了。

/* 这个煞笔错误困扰了我两个下午才解决*/

### 5. Read-only file system

```bash
dpkg: error: unable to access dpkg status area: Read-only file system
```

/* 这个错误当时没有找到解决方案，直接导致用小米4做服务器的计划失败*/




2026-01-10 增：

现在来看，我当时的根目录挂载在/dev/loop0上，这意味着这个Ubuntu是跑在一个镜像文件（类似虚拟磁盘？）里，而不是直接安装在物理分区上，而这个镜像文件的大小在刷机的时候就已经固定为2GB，这对一个系统来说太小了，当系统在尝试写入数据却发现空间耗尽时，为了保护文件系统不被损坏，内核会自动将该分区降级为read only模式。

一旦镜像文件满了，物理分区/userdata剩下再多空间也没用。如果想要解决这个问题，可能需要重新分区。但是当时折腾的那个手机已经找不到了，无从验证🤣