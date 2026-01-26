*此文档主要内容写于2022下半年，部分命令已经过时

部署环境：Ubuntu20.04；x86_64

# 部署Django

## 零、常用命令

#### 1.查看网站访问日志

```bash
tail -f /xxx/xxx/xxx.log
```

#### 2.在models.py中 创建/修改 表后同步到数据库

```bash
python3 manage.py makemigrations
python3 manage.py migrate
```





## 一、安装宝塔和升级

2025-12-25增：关于宝塔的安装应该已经过时了，现在宝塔已经更新到11了

#### 1. 下载宝塔面板

这个是老命令，我用这个安装的时候一直报下面第一个错。

```
wget -O install.sh http://download.bt.cn/install/install-ubuntu.sh && sudo bash install.sh
```

建议用新命令安装：

```
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh ed8484bec
```

安装完成之后，账户和密码在宝塔安装成功的时候会显示出来，记得记下来



##### 1）报错

```
E: Package 'python-pip' has no installation candidate
        Note, selecting 'python-dev-is-python2' instead of 'python-dev'
Package python-pip is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source
However the following packages replace it:
  python3-pip

E: Package 'python-pip' has no installation candidate
```

换成如下命令安装

```
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh ed8484bec
```







##### 2）报错

```
Traceback (most recent call last):
File "setup.py",line 19,in <module>
from setuptools import Extension,setup,find_packages
ImportError:No module named setuptools
======================
Pillow installation failed.
```

执行

```
wget --no-check-certificate https://www.moerats.com/usr/down/setuptools-0.6c11.tar.gz
tar -xvf setuptools-0.6c11.tar.gz
cd setuptools-0.6c11
python setup.py build
python setup.py install
```

然后再执行一遍安装命令。



##### 3）报错

```
ERROR: Command errored out with exit status 1: python setup.py egg_info Check the logs for full command output.
```

可以先不搭理它，笔者还没有遇到需要解决这个报错的情况。



#### 2. 第一个命令默认安装的是宝塔5，需要升级宝塔6，第二个命令直接整成宝塔7了。。。。。

```
curl -sSO http://download.bt.cn/install/update_to_6.sh && bash update_to_6.sh
```



#### 3. 宝塔用户名和密码

**记得先去服务器提供商那里开放端口！！！**

这个在宝塔安装成功的时候会显示出来，记得记下来

```
外网面板地址: xxx
username: xxx
password: xxx
```

如果忘记用户名和密码，可以执行这个查看

```
http://服务器ip:8888/1ogin
```





## 二、安装环境

#### 1. 安装软件

进入宝塔面板，把弹出来的关掉，去软件商店安装Python项目管理器、NGINX、mysql，注意版本。



#### 2. 去Python项目管理器里面创建项目

看看有没有可选的Python版本，如果没有就安装3.9。安装完再回来创建项目，启动方式选择uwsgi，架构选Django，设置一个端口，注意不要和其他的冲突，下面两个都勾选上。

项目路径（manage.py所在的目录）

用pip3 freeze > requirements.txt在项目路径生成requirements.txt

运行文件（wsgi.py文件的路径）

创建完之后会自动启动，看一下日志有没有报错，有的话就根据报错解决一下就好了。最后去模块里面下载Django和mysql。



到这里如果一切顺利，Django就能运行了。但是想啥呢，怎么会这么容易解决[Doge]





## 三、记录一下我踩过的坑



#### 1.同步静态文件的时候报错：

```
django.core.exceptions.ImproperlyConfigured: You're using the staticfiles app without having set the STATIC_ROOT setting to a filesystem path.
```

在settings.py中添加

```
import os
STATIC_ROOT = os.path.join(BASE_DIR,"static")
```

再执行载入的命令就行了



#### 2.迁移数据库的时候报错

```
django.db.utils.OperationalError: (1050, "Table 'app01_log' already exists")
```

执行manage.py makemigrations 未提示错误信息，但manage.py migrate时进行同步数据库时出现问题

进入 apps 下 migrations 目录， 删除除了 init.py之外的文件，再重新执行那两个命令就好了







## 四、部署完成后的注意事项

### 1. 如果项目中使用了 Django 的邮件功能

把settings.py发邮件的SMTP端口改成587，因不管是常用的25还是通常备用的465，全特么被煞笔阿里云封了。。。

血和泪的教训。。。哈哈哈。。。。

简单总结一下思路：  

首先一定要确保在本地调试时邮件功能是正常的，代码本身没问题、能成功发送。只要本地没问题，那么部署到服务器后出错，基本就可以确定是**配置层面的问题**。

如果部署后发现邮件发送失败，排查下来是卡在 `send_mail()` 这一块，那就围绕邮件配置去查。很多云服务器对外发端口都有严格限制，只能去翻文档、查资料，然后把 25 / 465 / 587 这些端口一个个试。



## 五、把本地代码上传上去之后

settings.py：

debug设置为False

改mysql账号密码，数据库名称（注意大小写）

改static路径

允许访问的主机'*'





## 六、部署完成后，下一次部署的步骤

1. 关闭Python任务，NGINX等

2. 在本地进入 apps 下 migrations 目录， 删除除了 init.py之外的文件，确保执行 python manage.py makemigrations 或者 migrate 不报错

3. 上传代码

4. 对项目内容进行修改，在settings.py中更改数据库名称、数据库用户名和密码、allow_host设置为"*"，在static配置路径的后面加上一条：

   ```python
   STATIC_ROOT = os.path.join(BASE_DIR, "static")
   ```
   
5. 进入wwwroot目录执行

   ```bash
   python3 manage.py makemigrations
   python3 manage.py migrate
   ```

6. 在manage.py同级目录下，进入虚拟环境

   ```bash
   source /xxxx/xxxx/bin/activate
   ```

   执行下面的命令是为了让NGINX能找到静态文件

   ```bash
   python3 manage.py collectstatic
   ```

   然后输入yes就行了。

7. 启动刚刚关闭的那些服务




