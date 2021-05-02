---
tags:
	- python
categories: Python3 安装
title: Python3 安装
---

# Python3 安装

Linux下默认安装的是Python2.7，要使用Python3，需要自行安装。Python3最新的版本是Python3.6。

在这里介绍在CentOS，Debian和Windows上安装Python3.6。

小丽，你跳过前面的，只看Windows上安装Python3.6哈。

<!-- more -->

## CentOS安装Python3.6

更详细的安装过程查看：[CentOS7安装Python3.6](https://www.yuzhi100.com/tutorial/centos/centos-anzhuang-python36)

IUS软件源中包含了Python3.6，可以使用IUS软件源安装Python3.6，查看[如何安装使用IUS软件源](https://www.yuzhi100.com/article/centos7-ruhe-shiyong-ius-xinban-ruanjian)

1）安装IUS软件源

```
复制#安装EPEL依赖
sudo yum install epel-release

#安装IUS软件源
sudo yum install https://centos7.iuscommunity.org/ius-release.rpm
```

2）安装Python3.6

```
复制sudo yum install python36u
```

安装Python3完成后的shell命令为python3.6，为了使用方便，创建一个到python3的符号链接

```
复制sudo ln -s /bin/python3.6 /bin/python3
```

3）安装pip3

安装完成python36u并没有安装pip，安装pip

```
复制sudo yum install python36u-pip
```

安装pip完成后的shell命令为pip3.6，为了使用方便，创建一个到pip3的符号链接

```
复制sudo ln -s /bin/pip3.6 /bin/pip3
```

## Debian安装Python3.6

Debian8的软件源中包含了Python3.4，要安装Python3.6，需要下载源文件安装：

1）安装编译，安装Python源文件的依赖包，GCC编译器，Make编译程序，Zlib压缩库：

```
复制sudo aptitude -y install gcc make zlib1g-dev
```

2）运行如下命令安装Python3.6，以下命令依次为获取Python3.6源文件，解压，配置环境，编译，安装Python3.6

```
复制wget https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tar.xz
tar xJf Python-3.6.0.tar.xz
cd Python-3.6.0
sudo ./configure
sudo make
sudo make install
```

安装完成后，Python安装在了/usr/local文件夹中，可运行文件/usr/local/bin，库文件/usr/local/lib，可以使用如下命令查看安装位置和版本

3）查看安装位置：

```
复制anxin@bogon:~$ which python3

/usr/local/bin/python3
```

4）验证Python3.6版本

```
复制anxin@bogon:~$ python3 -V

Python 3.6.0
```

## Windows安装Python3.6

1）进入[Python下载页面](https://www.python.org/downloads/)下载对应版本的Python安装包，本例下载Python3.6 64位Windows安装包 python-3.6.2-amd64.exe

**如果你的电脑系统是32 位的,你就直接点击箭头的按钮下载就好了!** 

![mark](https://blogimg.nos-eastchina1.126.net/171220/em47IffHIC.png)

**如果你的电脑系统是64 位的,就按下面流程下载哈**

![mark](https://blogimg.nos-eastchina1.126.net/171220/Hhj4AC819m.png)

![mark](https://blogimg.nos-eastchina1.126.net/171220/lj0mE2G40B.png)

![mark](https://blogimg.nos-eastchina1.126.net/171220/LkaDhF964L.png)



下载下来：

![mark](https://blogimg.nos-eastchina1.126.net/171220/5CIc1GB4ii.png)

2）双击Python3.6安装包，开始安装Python3.6。

**1. 选择 Install Now 安装方式方式**

**其实你也可以一步就完成 Python3.6 的安装，如果选择 Install Now 安装方式**，以后下的步骤你可以全部不用做。

查看安装是否成功，打开cmd 输入 python 即可

![mark](https://blogimg.nos-eastchina1.126.net/171220/lGDD9DK26a.png)

**2. 选择选择自定义安装（Customize installation）方式**，可以选择安装的内容和目录：

![mark](https://blogimg.nos-eastchina1.126.net/171220/7AkhCB08fC.png)



注：安装Python3.6时，可以选中 Add Python 3.6 to PATH ，这样Python会自动把Python3.6的路径加入当前用户的PATH路径下，而不是系统的PATH路径下。

3）选择要安装的内容（如果你清除它们是什么），一般可以选择默认，即：所有内容，点击Next

![mark](https://blogimg.nos-eastchina1.126.net/171220/4g2BHFGJAC.png)

4）选择一些安装选择，一般选择默认，在这一步可以选择安装目录，**也可以直接输入目录地址**，点击“浏览（Browse）”按钮：

![mark](https://blogimg.nos-eastchina1.126.net/171220/IllKcK3K4k.png)

5）选择安装的目录，最好先创建好目录如 python36 ，点击“确定”按钮：

![mark](https://blogimg.nos-eastchina1.126.net/171220/K99deLGafF.png)

6）如图所示选择好Python3.6的安装目录，点击“Install”按钮，开始安装Python3.6

7）如不出什么以外，会提示你安装完成

![mark](https://blogimg.nos-eastchina1.126.net/171220/2h4lLL7Aah.png)

恭喜你，成功的在Windows安装了Python 3.6！

## 配置Python3.6环境变量

你也可以在安装完成Python3.6后，自己手动配置Python3.6的环境变量。

Win7进入控制面板-->系统和安全-->系统-->高级系统设置-->环境变量-->系统变量，选中Path，双击编辑Path环境变量，添加路径<python-home>\和<python-home>\Scripts\：

![配置Python3.6环境变量](https://www.yuzhi100.com/sites/default/files/inline-images/win7-install-python-362-7.jpg)

在本例中我们添加的Python3.6环境变量的路径为：;C:\python36\Scripts\;C:\python36\，注意：Windows的环境变量已 ; 分隔。

### 我们敲代码的地方

![mark](https://blogimg.nos-eastchina1.126.net/171220/Dh8J6JH3hB.png)

打开箭头指向的 IDLE ，就是我们的编辑器。
