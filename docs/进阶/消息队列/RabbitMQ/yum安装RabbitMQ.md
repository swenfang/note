---
tags:
	- 消息队列
categories: RabbitMQ
title: yum安装RabbitMQ
---
# yum安装RabbitMQ

<!--more-->

进入/etc/yum.repos.d/ 文件夹

创建rabbitmq-erlang.repo 文件

内容如下：

```
[rabbitmq-erlang] 
name=rabbitmq-erlang
baseurl=https://dl.bintray.com/rabbitmq-erlang/rpm/erlang/21/el/7
gpgcheck=1
gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1
```



创建rabbitmq.repo 文件

内容如下：

```
[bintray-rabbitmq-server]
name=bintray-rabbitmq-rpm
baseurl=https://dl.bintray.com/rabbitmq/rpm/rabbitmq-server/v3.8.x/el/8/
gpgcheck=0
repo_gpgcheck=0
enabled=1
```



安装命令

```
yum install rabbitmq-server
```

rabbitmq相关命令

开启

```
service rabbitmq-server start
```

关闭

```
service rabbitmq-server stop
```

查看状态

```
service rabbitmq-server status
```

重启

```
service rabbitmq-server restart
```

启用插件页面管理

```
rabbitmq-plugins enable rabbitmq_management
```

创建用户

```
rabbitmqctl add_user admin mypassword
```

赋予权限

```
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```

浏览器访问http://机器IP:15672打开管理界面，使用上一步配置好的admin账号登录

访问不到管理后台的原因

如果无法访问到界面，那么有可能是服务器防火墙没有关闭的问题，解决这个问题有良好总方式：

关闭防火墙或者配置15672和5672 端口可以通过

关闭防火墙：systemctl stop firewalld 或者禁用 systemctl disable firewalld 开发或者测试环境。

配置防火墙端口：

15672（ui管理端口）：firewall-cmd --add-port=15672/tcp --permanent

5672（远程连接端口）：firewall-cmd --add-port=5672/tcp --permanent

最后 执行 firewall-cmd  --reload