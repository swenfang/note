---
tags:
	- python
categories: 发布包到pypi
title: python发布包到pypi
---
# python发布包到pypi

python更新太快了，甚至连这种发布上传机制都在不断的更新，这导致网上的一些关于python发布上传到pypi的教程都过时了，按着博文操作可能会失败。

<!-- more -->

## pypi相关概念介绍

#### 关于pypi本身

　　pypi是专门用于存放第三方python包的地方，你可以在这里找别人分享的模块，也可以自己分享模块给别人。可以通过easy_install或者pip进行安装。pypi针对分享提供了两个平台，一个是测试发布平台，一个是正式发布平台，我们正式发布前可以先用测试发布平台发布，看是否正确，然后再采用正式发布平台．

#### 关于python打包发布工具

　　python的打包安装工具也经历了很多次变化，由最早的distutils到setuptools到distribute又回到setuptools，后来还有disutils2以及distlib等，其中distutils是python标准库的一部分，它提出了采用setup.py机制安装和打包发布上传机制．setuptools(操作系统发布版本可能没有自带安装,需要自己额外安装)基于它扩展了很多功能，也是采用setup.py机制，针对安装额外提供了easy_install命令．distribute是setuptools的一个分之，后来又合并到setuptools了，所以姑且就把它看做是最新的setuptools吧！和我们打包最相关的貌似就是distutils或者setuptools，两者都可以用来打包发布并上传到pypi，后面介绍采用distutils，如果想更多的功能，比如想通过entry points扩展的一些功能，那么就要使用setuptools了．另外，还有一个工具可以用来发布到pypi，叫twine，需要额外安装．最后,需要确保自己的工具都是尽量新的,官方给出的版本参考:twine v1.8.0+ (recommended tool), setuptools 27+, or the distutils included with Python 3.4.6+,Python 3.5.3+, Python 3.6+, and 2.7.13+,升级的参考命令:

`sudo -H pip install -U pip setuptools twine`

## 写一个 setup.py

```python
from distutils.core import setup  # 从 Python 发布工具导入 setup 函数
setup(
  	  name='swfswf', # 要打包的模块名称
      version='1.0', 
      description='Python Distribution Utilities',
      author='shenwenfang',
      author_email='1978626782@qq.com',
      url='https://swenfang.github.io/',
     )
```

 也可参阅官方文档：

- [https://docs.python.org/2/dis...](https://docs.python.org/2/distutils/setupscript.html)

**注意：**

- 这里只是最基本的参考例子，执行打包会报警告，说缺少一些需要的文件，比如MANIFEST.in、readme.txt等等，暂时忽略即可。正式的项目中会复杂很多，甚至需要用到setuptools来扩展。这部分可以参考其他文档

- **为了保证效果，在打包之前我们可以验证setup.py的正确性。执行代码`python setup.py check`，输出一般是running check，如果有错误或者警告，就会在此之后显示.没有任何显示表示Distutils认可你这个setup.py文件**

- 执行 **`python setup.py sdist upload -r pypi`** 创建发布并上传,如果想先上传到测试平台，可以执行 **python setup.py sdist upload -r pypitest**，成功后再执行上面命令上传到正式平台。注意，这一步的配置文件里面由于pypi的发布机制更新导致有一些问题的出现。

- 410错误：这个是pypi上传机制变更导致的,我虽然参考的是最新的blog，但是还是过时了！！！（.pypirc的repository过时了，很多博客说的`repository: https://pypi.python.org/pypi`会导致后面步骤操作出现410错误）

  ​

## 打包发布到pypi

**基本流程：**

- 注册 pypi 账号，如果期望测试发布，同时需要注册[pypitest](https://testpypi.python.org/pypi?:action=register_form)账号（可以采用相同的用户名和密码）

  直接通过官网注册 [https://pypi.python.org/pypi?...](https://pypi.python.org/pypi?%3Aaction=register_form)，填写用户名、密码、确认密码、邮箱，

   但是需要验证邮件并确认激活。

- 创建配置文件, 该配置文件里面记录了你的账号信息以及要发布的平台信息，参考如下配置文件创建即可

  在自己的用户目录下新建一个空白文件命名为`.pypirc`，内容如下：

  ```python
  [distutils]
  index-servers=pypi

  [pypi]
  repository = https://upload.pypi.org/legacy/
  username = shenwenfang
  password = **************

  # 如果期望测试发布，同时需要
  #repository: https://test.pypi.org/legacy/
  #username= your_username
  #password= your_password

  ```

![mark](https://blogimg.nos-eastchina1.126.net/180101/jDEFKjA19C.png)

**相关命令：**

```python
python setup.py check #验证setup.py的正确性
```

```python
python setup.py sdist upload -r pypi #打包发布到pypi，返回 "Server response (200) : OK" 说明上传成功
```

**查看上传记录：**

如果你的包已经上传成功，那么当你登录PyPI网站后应该能在右侧导航栏看到管理入口。

![img](https://segmentfault.com/img/remote/1460000008663129?w=230&h=177)

## 测试代码块

```python
import swfswf;
swfswf.print_lol(参数1，参数2...)

```

### 管理你的包

如果你的包已经上传成功，那么当你登录PyPI网站后应该能在右侧导航栏看到管理入口。

![img](https://segmentfault.com/img/remote/1460000008663129?w=230&h=177)

点击包名进去后你可以对你的包进行管理，当然你也可以从这里删除这个包。

### 让别人使用你的包

包发布完成后，其他人只需要使用pip就可以安装你的包文件。比如：

```
pip install package-name
```

如果你更新了包，别人可以可以通过`--update`参数来更新：

```
pip install package-name --update
```

## 可能遇到的错误

1. **410错误**：.pypirc的repository过时了，很多博客说的`repository: https://pypi.python.org/pypi`会导致后面步骤操作出现410错误
2. **403错误**：是因为项目和已有 的项目重名了，可以先到 https://pypi.python.org/simple/ 上搜一下看看是否重名。解决的方法自然就是修改一下setup.py中setup函数中的name参数，删除之前生成的dist文件夹并重新生成，然后再upload

参考文章：

https://www.xncoding.com/2015/10/26/python/setuptools.html

https://segmentfault.com/a/1190000008663126

http://www.cnblogs.com/rongpmcu/p/7662821.html
