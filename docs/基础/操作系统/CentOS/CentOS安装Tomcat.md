---
tags:
	- CentOS
categories: CentOS
title: CentOS安装Tomcat
---
# CentOS安装Tomcat

## 一.tomcat的简介

这是**Apache Tomcat** Servlet / JSP容器的文档包的顶级入口点 。的Apache Tomcat 8.0版实现了Servlet 3.1和JavaServer Pages 2.3 [规范](https://wiki.apache.org/tomcat/Specifications)从 [Java社区进程](https://www.jcp.org/)，并包含许多额外的功能，使开发和部署Web应用程序和Web服务的有用平台
<!--more-->
## 二.tomcat的安装

### 1.tomcat下载

官网地址：[http://tomcat.apache.org/](https://www.cnblogs.com/frankdeng/p/%E5%AE%98%E7%BD%91%E5%9C%B0%E5%9D%80%EF%BC%9Ahttp://tomcat.apache.org/)

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419094746-454034.jpg)

```
[admin@node21 software]$ wget http://mirrors.shu.edu.cn/apache/tomcat/tomcat-8/v8.0.53/bin/apache-tomcat-8.0.53.tar.gz
[admin@node21 software]$ ll
-rw-rw-r-- 1 admin admin   9455895 Jun 30 00:39 apache-tomcat-8.0.53.tar.gz
```

### 2.tomcat安装

查看是否安装 JDK

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419094915-822740.jpg)

1）解压缩安装包

```
[admin@node21 software]$ tar zxvf apache-tomcat-8.0.53.tar.gz 
```

2）移动安装包到/usr/local/tomcat目录下，也可以不移动设置tomcat环境变量

```![](http://blogimg.nos-eastchina1.126.net/shenwf20190419094956-281396.jpg)
[admin@node21 software]$ sudo mv apache-tomcat-8.0.53 /usr/local/tomcat8
```

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419094956-281396.jpg)

### 3.启动tomcat

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419095022-266452.jpg)

```
[admin@node21 bin]$ pwd
/usr/local/tomcat8/bin
[admin@node21 bin]$ ./startup.sh 
```

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419095051-652902.jpg)

### 4.测试内部是否启动成功

```
curl "http://47.106.180.186:8089/"
```

![1555682470740](C:\Users\shenwenfang\AppData\Roaming\Typora\typora-user-images\1555682470740.png)

### 5.WebUI访问

tomcat默认端口8080，访问地址：<http://47.106.180.186:8089/>，默认页面如下

![1555682033090](C:\Users\shenwenfang\AppData\Roaming\Typora\typora-user-images\1555682033090.png)

### 6.停止tomcat

```
[admin@node21 webapps]$ /usr/local/tomcat8/bin/shutdown.sh 
```

## 三.Tomcat服务部署web应用

**第一种方式：利用Tomcat自动部署**

​        利用Tomcat自动部署方式是最简单的、最常用的方式。若一个web应用结构为**D:\workspace\WebApp\AppName\WEB-INF\*，只要将一个Web应用的WebContent级的AppName**直接扔进%Tomcat_Home%\webapps文件夹下，系统会把该web应用直接部署到Tomcat中。**所以这里不再赘述**

**第二种方式：手动部署修改%Tomcat_Home%\conf\server.xml文件来部署web应用**

打开**%Tomcat_Home%\conf\server.xml**文件并在其中<host>标签里增加以下元素：

```
<Context docBase="D:\workspace\WebApp\AppName" path="/XXX" debug="0" reloadable="false" /> 
```

然后启动Tomcat即可。

`注意：`

​      （1）以上代码中的**workDir**表示将该Web应用部署后置于的工作目录（Web应用中JSP编译成的Servlet都可在其中找到）。如果自定义web部署文件XXX.xml中未指明workdir，则web应用将默认部署在

```
%Tomcat_Home%\work\Catalina\localhost
```

路径下新建的以XXX命名的文件夹下。（Web应用中JSP编译成的Servlet都可在其中找到）

​      （2）**Context path**即指定web应用的虚拟路径名。**docBase**指定要部署的Web应用的源路径。

## 四.解决中文乱码及测试访问页

### 1.测试修改访问页面

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419100432-389841.jpg)

```html
<html>
    <body>
       <h1>Hello,世界!</h1>
    </body>
</html>
```

再次启动tomcat，输入：http://47.106.180.186:8089/hello/index.html，出现下图，发现有中文乱码现象。

### 2.解决中文乱码

乱码原因：tomcat8之前，URL中参数的默认解码是ISO-8859-1，而tomcat8的默认解码为utf-8。ISO-8859-1并未包括中文字符，中文字符不能被正确解析了。