---
tags:
	- 开发工具
categories: webstorm
title: webstorm 配置 Apache 服务器
---
# webstorm配置Apache服务器

今天尝试了在 apache 上部署静态工程，顺便记录一下，希望能帮到需要的同学！
<!-- more -->

## 添加 apache 服务

![mark](https://blogimg.nos-eastchina1.126.net/180805/e4DLdAjEf3.png)

​    服务的类型选择 Local or mounted folder 。

## 配置 Connection	

![mark](https://blogimg.nos-eastchina1.126.net/180805/mDCbB44e4f.png)

## 配置 Mappings

![mark](https://blogimg.nos-eastchina1.126.net/180805/Ah1JILhi5B.png)

1处，选择你工程所在的位置，选择到工程直接访问的文件夹。（我的工程名称是 com.eyuninfo.web,但是我所有的页面都在 WebContent 的目录下）。

2处，htdocs 是你安装的 Apache 的目录下的一个目录，在这个目录下面放的是你在 webstorm 的 Tools->Deployment->Upload to Default Server 的所上传的文件，也就是部署到服务器路径下面的目录，如下：

![mark](https://blogimg.nos-eastchina1.126.net/180805/8mm36mckJ1.png)

3处，sunriseui 是工程的上下文。 访问的时候就是 http://127.0.0.1/sunriseui/sunrise/pages/home/tophome_w.html 这样了。



## 保存，完成。
