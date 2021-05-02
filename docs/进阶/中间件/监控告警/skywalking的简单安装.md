---
tags:
	- 链路跟踪系统
categories: skywalking
title: skywalking 的简单安装
---
# skywalking 的简单安装

win10 环境下来搭建 skywalking 基于 elasticsearch 。

<!--more-->

## 安装 JDK

由于本次使用的 elasticsearch 版本是 elasticsearch-7.6 ，skywalking 版本是 skywalking-6.6 ，所 JDK 版本要 1.8 或以上。

## 安装 elasticsearch 

1）下载

https://www.elastic.co/downloads/

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314101619-904969.jpg)

2) 解压

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314101748-472240.jpg)

3）修改配置文件

进入 elasticsearch 解压文件夹的 config目录,编辑 elasticsearch.yml 

改成服务地址

```yml
network.host: 127.0.0.1
```

服务名和agent的service_name相同

```yml
cluster.name: jccrc-pre-sales-service
```

elasticsearch的cluster.name和agent的service_name相同原因？假如是多个agent的service_name怎么配置？

4）启动

进入到解压文件夹的 bin 目录，双击 elasticsearch.bat 文件即可。

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314101909-835484.jpg)

4）访问

http://localhost:9200/

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314102017-596531.jpg)

这样说明启动 elasticsearch  成功。

5）关闭

## 安装 skywalking 

1）下载：

http://skywalking.apache.org/zh/downloads/

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314044638-849054.jpg)

2）解压

apache-skywalking-apm-es7-6.6.0.zip

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314044857-647089.jpg)

3）修改配置文件

在 skywalking 解压后 config 目录找到 elasticsearch.yml 文件，做如下修改：

skywalking  支持 h2、mysql、ElasticSearch 作为数据存储。官网好像是推荐使用ElasticSearch，为什么推荐？应该是ElasticSearch快。需要注意的是，ElasticSearch不是自带的，需要安装。

 Skywalking启用ElasticSearch，只需要配置文件设置如下：

放开 elasticsearch7 的注释同时记得注释掉 h2 。

```yml
  elasticsearch7:
    #nameSpace: ${SW_NAMESPACE:""}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
    trustStorePath: ${SW_SW_STORAGE_ES_SSL_JKS_PATH:"../es_keystore.jks"}
    trustStorePass: ${SW_SW_STORAGE_ES_SSL_JKS_PASS:""}
    user: ${SW_ES_USER:""}
    password: ${SW_ES_PASSWORD:""}
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    # Those data TTL settings will override the same settings in core module.
    recordDataTTL: ${SW_STORAGE_ES_RECORD_DATA_TTL:7} # Unit is day
    otherMetricsDataTTL: ${SW_STORAGE_ES_OTHER_METRIC_DATA_TTL:45} # Unit is day
    monthMetricsDataTTL: ${SW_STORAGE_ES_MONTH_METRIC_DATA_TTL:18} # Unit is month
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:1000} # Execute the bulk every 1000 requests
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
    resultWindowMaxSize: ${SW_STORAGE_ES_QUERY_MAX_WINDOW_SIZE:10000}
    metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:5000}
    segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
#  h2:
#    driver: ${SW_STORAGE_H2_DRIVER:org.h2.jdbcx.JdbcDataSource}
#    url: ${SW_STORAGE_H2_URL:jdbc:h2:mem:skywalking-oap-db}
#    user: ${SW_STORAGE_H2_USER:sa}
#    metadataQueryMaxSize: ${SW_STORAGE_H2_QUERY_MAX_SIZE:5000}
```

配置文件config/application.yml中的存储配置需要注意--clusterName的值一定要与es集群的cluster.name一致，还有clusterNodes的地址：

```yml
  elasticsearch7:
    #nameSpace: ${SW_NAMESPACE:""}
```

4）启动 skywalking 

在bin目录下执行 startup.bat

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314052219-70455.jpg)

执行startup.bat之后会启动如下两个服务：
（1）Skywalking-Collector：追踪信息收集器，通过 gRPC/Http 收集客户端的采集信息 ，Http默认端口 12800，gRPC默认端口 11800。
（2）Skywalking-Webapp：管理平台页面 默认端口 8080

5）访问管理后台：

http://localhost:8080/

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314052417-117986.jpg)

6）可能会遇到的问题

在bin目录下执行 startup.bat 时，startup.bat 闪退，不会启动 Skywalking-Collector 和 Skywalking-Webapp。

解决方案：将 skywalkinge 解压之后的文件夹放到 JDK 安装目录的上一级

7）关闭

## 配置要监控的服务

演示 jccrc-pre-sales-service 服务调用 jccrc-vehicle-service 服务的监控配置过程。

添加参数java启动参数，如下：

```yml
-javaagent:C:\skywalking-6.6\apache-skywalking-apm-bin-es7\agent\skywalking-agent.jar
-Dskywalking.agent.service_name=jccrc-vehicle-service
-Dskywalking.collector.backend_service=127.0.0.1:11800
```

skywalking-agent.jar 这个 jar 在 skywalking 解压目录 agent 下。

service_name 指定是你的服务名称（AppId）

backend_service 指的是 skywalking 收集客服端信息地址

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314111123-198691.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314111644-237065.jpg)

再次skywalking访问管理后台

http://localhost:8080/

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314052417-117986.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314104339-583860.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314104359-853488.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314104423-207177.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314104454-280635.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314104815-40387.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314105104-950049.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314105515-438915.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314110743-594703.jpg)
