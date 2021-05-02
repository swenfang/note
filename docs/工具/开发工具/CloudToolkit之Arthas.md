---
tags:
	- 开发工具
categories: IntelliJ IDEA
title: CloudToolkit之Arthas
---
# CloudToolkit之Arthas

通过 Cloud Toolkit 插件，您可以在本地 IDE 中使用 Arthas 来实现本地诊断或远程诊断。

<!--more-->

## 3 分钟了解 Alibaba Cloud Toolkit

https://yq.aliyun.com/articles/689345?spm=a2c4e.11153940.0.0.33fa2708lyTUps

## 下载 Cloud Toolkit 插件

https://cn.aliyun.com/product/cloudtoolkit

![](http://blogimg.nos-eastchina1.126.net/wenfang20200321035510-67388.jpg)

https://plugins.jetbrains.com/plugin/11386-alibaba-cloud-toolkit?spm=5176.11997469.1327965.1.3a622fa85iZLDD

![](http://blogimg.nos-eastchina1.126.net/wenfang20200321035351-770731.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200321035430-159498.jpg)

## 安装教程

![](http://blogimg.nos-eastchina1.126.net/wenfang20200321035718-667923.jpg)

点击 InteIIi IDEA 可以查看 IDEA 怎么使用  Cloud Toolkit 。

我这里直接使用从阿里官网下载的 toolkit-intellij-2020.3.1.zip 导入到 IDEA 的插件中。 

1.Ctrl+Alt+s 打开 Settings 面板

![](http://blogimg.nos-eastchina1.126.net/wenfang20200321040928-575819.jpg)

2.选择从阿里官网下载下来的 toolkit-intellij-2020.3.1.zip 。

![](http://blogimg.nos-eastchina1.126.net/wenfang20200321041038-400151.jpg)

3.重启IDEA

![](http://blogimg.nos-eastchina1.126.net/wenfang20200321041107-54899.jpg)

4.重启之后会重新这个提示，并在 IDEA 的工具栏上多了一个 Alibaba Cloud Explorer

![](http://blogimg.nos-eastchina1.126.net/wenfang20200321040429-821322.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200321040723-784520.jpg)

## 可能遇到的问题

![](http://blogimg.nos-eastchina1.126.net/wenfang20200322081945-539564.jpg)

查询  C:\Users\shenw\.arthas\lib  目录是否是如下内容。

![](http://blogimg.nos-eastchina1.126.net/wenfang20200322082104-226590.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200322082050-418340.jpg)

## 使用手册

### 代码检查

https://help.aliyun.com/document_detail/145927.html?spm=a2c4g.11186623.6.593.48dec928DUf3GE

### Arthas 诊断

https://help.aliyun.com/document_detail/112975.html?spm=a2c4g.11186623.4.4.6bda7e11QVU8Jv