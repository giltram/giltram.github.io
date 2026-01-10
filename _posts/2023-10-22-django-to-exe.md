## Django打包exe

### 1.  环境

win10

Python3.8.2

Django3.2.9

PyInstaller6.1.0

### 2. 制作项目的.spec文件

　　进入到项目的所在的路径中，执行如下命令生成 .spec文件,文件位于当前路径下

```
pyi-makespec -D manage.py
```

然后运行如下命令，生成打包的exe文件

```
pyinstaller manage.spec
```



### 3. 运行程序

进入`/dist/manage/`，输入

```
manage.exe runserver
```




### 4. 报错和解决

##### 1. 启动时报错 `RuntimeError: Script runserver does not exist`

这个问题一般是 Django 在打包后使用自动重载机制导致的。解决方法很简单，启动时加上 `--noreload` 参数即可，例如：

```bash
manage.exe runserver --noreload
```

---

##### 2. 提示 `No module named XXX`

这种报错通常是因为 Django 的某些模块不会被 PyInstaller 自动识别并收集，需要手动指定。

做法是在 `manage.spec` 文件中，把原本的

```python
hiddenimports=[]
```

改成类似下面这样：

```python
hiddenimports=['xxx', 'xxx', 'xxx']
```

缺哪个模块，就把对应的模块名加进去。比较讨厌的一点是，PyInstaller 一次只报一个缺失的模块，所以要反复报错 → 改 spec → 重新打包 → 再报错 → 再改。  
笔者这边前前后后打包了十几次才全部补齐，具体次数还是要看项目本身的依赖情况。

##### 3. 打开网页时报 `TemplateDoesNotExist`

这个错误通常是模板路径不对导致的。直接根据报错信息中提示的查找路径，把项目里的 `templates` 目录拷贝到对应位置即可。拷贝完成后刷新页面，一般就能正常显示了。

##### 4. 页面丢失 CSS、JS 等静态资源

这个问题可以参考 PyInstaller 的一个相关 issue：  
[https://github.com/pyinstaller/pyinstaller/issues/2368](https://github.com/pyinstaller/pyinstaller/issues/2368)

处理方式如下：

首先在项目的 `settings.py` 中添加静态文件配置。这里的 `static` 是项目原本存放静态文件的目录，`static_root` 是 `static` 目录下新建的一个空文件夹，用来集中存放收集后的静态资源：

```python
STATIC_ROOT = os.path.join(BASE_DIR, 'static', 'static_root')
```

然后执行一次：

```bash
python manage.py collectstatic
```

把所有静态文件收集到 `static_root` 中。

接着在 `urls.py` 里添加下面的代码，让 Django 能在运行时正确访问这些静态资源：

```python
from django.conf.urls import static
from project_1 import settings

urlpatterns += static.static(
    settings.STATIC_URL,
    document_root=settings.STATIC_ROOT
)
```

配置完成后，CSS 和 JS 丢失的问题一般就可以解决了。