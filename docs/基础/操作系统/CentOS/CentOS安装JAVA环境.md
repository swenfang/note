---
tags:
	- CentOS
categories: CentOS
title: CentOS安装JAVA环境（JDK 1.8）
---
# CentOS安装JAVA环境（JDK 1.8）

## 通过yum安装

```
如果服务器没有wget服务，使用yum -y install wget安装
yum install java-1.8.0-openjdk* -y
```

执行这一条命令就可以直接安装，并且无需配置就能使用。

## 下载tar包安装

<http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html>

我选择linux x64版本：

<!--more-->

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419101241-441593.jpg)



## 下载

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419101543-831303.jpg)

下载以后通过命令检查安装包大小是否符合

```
ls -lht
```

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419101640-785218.jpg)

## 安装

（1）创建安装目录

```
mkdir /usr/local/java/
```

（2）解压至安装目录

```
tar -zxvf jdk-8u171-linux-x64.tar.gz -C /usr/local/java/
```

## 设置环境变量

打开文件

```
vim /etc/profile
```

在末尾添加

```
export JAVA_HOME=/usr/local/java/jdk1.8.0_171
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

使环境变量生效

```
source /etc/profile
```

添加软链接

```
ln -s /usr/local/java/jdk1.8.0_171/bin/java /usr/bin/java
```

检查

```
java -version
```

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419102013-808980.jpg)

