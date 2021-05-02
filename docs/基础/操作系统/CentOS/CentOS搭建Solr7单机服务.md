---
tags:
	- CentOS
categories: CentOS
title: CentOS搭建Solr7单机服务
---
# CentOS搭建Solr7单机服务

## 一.Solr安装环境

### 1.官方参考文档

Solr教程参考指南：<http://lucene.apache.org/solr/guide/7_4/solr-tutorial.html>

<!--more-->

### 2.Solr运行环境

系统要求：Java 8+      这里我们把solr服务部署到Tomacat服务器中，Tomcat安装过程参考：<https://swenfang.github.io/2019/04/19/CentOS/CentOS安装Tomcat/>

**在solr5以前solr的启动都有tomcat作为容器，但是从solr5以后solr内部集成jetty服务器，可以通过bin目录中脚本直接启动。就是从solr5以后跟solr4最大的区别是被发布成一个独立的应用。**

### 3.Solr下载

 下载地址：<http://archive.apache.org/dist/lucene/solr/>

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419093633-377825.jpg)

```
[admin@node21 software]$ wget http://archive.apache.org/dist/lucene/solr/7.4.0/solr-7.4.0.tgz
[admin@node21 software]$ ll
-rw-rw-r-- 1 admin admin 167346886 Jun 19 02:51 solr-7.4.0.tgz
```

## 二.Solr单机安装

### 1. 解压安装包

```
[admin@node21 software]$ tar zxvf solr-7.4.0.tgz 
[admin@node21 software]$ ls solr-7.4.0
bin CHANGES.txt contrib dist docs example licenses LICENSE.txt LUCENE_CHANGES.txt NOTICE.txt README.txt server
```

### 2.部署solr到tomcat下

注意，这里因为我用的是solr7.4最新版，所以跟solr4版本要拷贝*.war文件，然后再启动tomcat解压的操作是不一样的 ，

**1）复制并重命名solr目录里的server/solr-webapp/webapp文件夹到/usr/local/tomcat8/webapps/solr**

```
[admin@node21 software]$ sudo cp -r solr-7.4.0/server/solr-webapp/webapp /usr/local/tomcat8/webapps/solr
```

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419093852-618647.jpg)

**2）拷贝solr-7.4.0\server\lib\ext 下的jar包以及lib目录下gmetric4j-1.0.7.jar和metrics开头的jar包拷贝到 tomcat8\webapps\solr 项目的WEB-INF\lib下**

```
[admin@node21 software]$ sudo cp solr-7.4.0/server/lib/ext/* /usr/local/tomcat8/webapps/solr/WEB-INF/lib/
[admin@node21 software]$ sudo cp solr-7.4.0/server/lib/gmetric4j-1.0.7.jar /usr/local/tomcat8/webapps/solr/WEB-INF/lib/
[admin@node21 software]$ sudo cp solr-7.4.0/server/lib/metrics-*  /usr/local/tomcat8/webapps/solr/WEB-INF/lib/
```

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419093937-207098.jpg)

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419093955-399271.jpg)

3）**创建一个索引库solrhome**

拷贝solr-7.4.0\server 下的solr文件夹到其它非中文目录下，重命名为solrhome，我是建立到了/usr/local/tomcat8/solrhome下

```
[admin@node21 software]$ sudo cp -r solr-7.4.0/server/solr /usr/local/tomcat8/solrhome
```

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419094030-250401.jpg)

4）**关联solr及索引库solrhome，**需要修改tomcat里solr工程的web.xml文件

```
[admin@node21 software]$ sudo vi /usr/local/tomcat8/webapps/solr/WEB-INF/web.xml 
```

找到如下代码，打开注释，修改自己的solrhome的路径/put/your/solr/home/here，我的是 /usr/local/tomcat8/solrhome 路径。

```xml
   <!--
     <env-entry>
        <env-entry-name>solr/home</env-entry-name>
        <env-entry-value>/put/your/solr/home/here</env-entry-value>
        <env-entry-type>java.lang.String</env-entry-type>
     </env-entry>
    -->
```

如下图：

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419094153-478365.jpg)

然后到最下方，将这一段注释掉，不然会报403错误，完成后保存退出（solr4部署不用注释这个）

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419094222-739442.jpg)

**5）拷贝solr7.4.0\server\resources下的** log4j2.xml **到tomcat8/webapps/solr/WEB-INF\classes，如果WEB-INF下没有classes文件那么就创建一个classes文件夹** 

```
[admin@node21 tomcat8]$ sudo mkdir -p /usr/local/tomcat8/webapps/solr/WEB-INF/classes/
[admin@node21 tomcat8]$ sudo cp -r /opt/software/solr-7.4.0/server/resources/log4j2.xml /usr/local/tomcat8/webapps/solr/WEB-INF/classes/
```

**6）修改tomcat的bin目录下catalina.bat脚本，增加solr.log.dir系统变量，指定solr日志记录存放地址。**

```
[root@node21 solr]# vi /usr/local/tomcat8/bin/catalina.sh 
JAVA_OPTS="$JAVA_OPTS -Dsolr.log.dir=/usr/local/tomcat8/solrhome/logs"
```

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419094314-214585.jpg)

### 3.启动服务 

启动tomcat，访问需要完整路径，我的是 <http://47.106.180.186:8089/solr/index.html>

![](http://blogimg.nos-eastchina1.126.net/shenwf20190419094507-951150.jpg)

