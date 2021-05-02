---
tags:
	- 链路跟踪系统
categories: skywalking
title: skywalking是什么
---
# skywalking是什么

分布式追踪系统，也是**APM**（Application Performance Monitoring），即应用性能监控系统。尤其是为服务，云原生，以及基于容器（k8s, Mesos）架构而设计，skywalking就是这样一个**为了观察和监控分布式系统的解决方案**。

skywalking是一个收集，分析，聚合和可视化来自服务和云原生架构数据的开源观察平台，它提供了一个非常简单的方法，让你清晰认识你的分布式系统。

<!--more-->

skywalking完全由中国人开发，项目发起人是吴晟(https://github.com/wu-sheng)，并已经进入apache孵化。Github地址：https://github.com/apache/incubator-skywalking。

## skywalking与其他链路跟踪系统对比

|                     | cat            | pinpoint           | zipkin                   | jaeger                   | skywalking                        |
| ------------------- | -------------- | ------------------ | ------------------------ | ------------------------ | --------------------------------- |
| OpenTracing兼容     | 否             | 否                 | 是                       | 是                       | 是                                |
| 客户端支持语言      | java           | java、php          | java,c#,go,php等         | java,c#,go,php等         | Java, .NET Core, NodeJS and PHP   |
| 存储                | mysql , hdfs   | hbase              | ES，mysql,Cassandra,内存 | ES，kafka,Cassandra,内存 | ES，H2,mysql,TIDB,sharding sphere |
| 传输协议支持        |                | thrift             | http,MQ                  | udp/http                 | gRPC                              |
| ui丰富程度          | 低             | 高                 | 低                       | 中                       | 中                                |
| 实现方式-代码侵入性 | 拦截请求，侵入 | 字节码注入，无侵入 | 拦截请求，侵入           | 拦截请求，侵入           | 字节码注入，无侵入                |
| 扩展性              | 低             | 低                 | 高                       | 高                       | 中                                |
| trace查询           | 不支持         | 不支持             | 支持                     | 支持                     | 支持                              |
| 警告支持            | 支持           | 支持               | 不支持                   | 不支持                   | 支持                              |
| jvm监控             | 不支持         | 支持               | 不支持                   | 不支持                   | 支持                              |
| 性能损失            |                | 高                 | 中                       | 中                       | 低                                |

## skywalking架构图

![](http://blogimg.nos-eastchina1.126.net/wenfang20200316105700-136453.jpg)

架构图解读：

1. 架构图左边部分就是skywalking的`UI`，即提供给相关人员操作使用的界面。在UI上可以看到服务拓扑图，接口调用链路信息等。并且skywaling6.x通过新的GraphQL查询协议，不再与UI强绑定；
2. 架构图的右边是skywalking支持的`底层存储`。由图可知，skywalking支持es，mysql, TiDB, H2, ShardingSphere这些存储方式，用户可以根据自己的能力选择适合自己的存储（其中，H2仅限调研阶段使用，禁止使用在生产环境）。同时存储是开放的，用户可以自定义实现，例如使用HBase作为底层存储（如果你的自定义实现经过生产环境的考验，还可以将源码贡献给skywalking，成为apache顶级项目skywalking的contributor甚至committer）。
3. 架构图的上面部分，表示skywalking的`探针`（probe）。由图可知，skywalking支持采集zipkin（v1/v2）格式，skywalking自己的格式，Istio遥测格式等。 skywalking通过探针收集tracing和metrics数据并格式化，然后通过协议（gRPC或者HTTP）发送给skywalking核心部分即OAP观察分析平台。
4. 架构图中间部分就是skywalking的核心，称为skywalking`可观测分析平台`。并且由图可知，skywalking支持通过gRPC和http两种协议接收数据。且收集的数据主要分为两部分：tracing和metrics。

## skywalking的主要特性

| 服务，实例和端点的metrics分析 |      |
| :---------------------------- | ---- |
| 根因分析                      |      |
| 服务拓扑图分析                |      |
| 服务，实例和端点的依赖分析    |      |
| 慢服务和接口探测              |      |
| 性能优化                      |      |
| 分布式追踪以及上下文广播      |      |
| 告警                          |      |

根因分析

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314105104-950049.jpg)

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314105515-438915.jpg)

服务拓扑图分析

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314104339-583860.jpg)

服务，实例和端点的依赖分析

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314104359-853488.jpg)

性能优化

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314110743-594703.jpg)

服务，实例和端点的metrics分析

![](http://blogimg.nos-eastchina1.126.net/wenfang20200314052417-117986.jpg)



## 概念说明

- **服务（service）**：表示提供相同行为请求的一个`服务集合`或`服务组`工作负载，例如用户服务，评论服务等。当你用语言探针或SDK时，你可以定义服务名称。或是SkyWalking使用你在Istio这类平台中定义的名字。
- **服务实例（service instance）**：服务组中的每个工作负载都称作服务实例，例如部署的若干用户服务中的某个JVM实例。就像pods在Kubernetes(k8s)中一样，它不需要是OS中的单个进程。然而，如果您正在使用语言探针，一个服务实例则实际上是OS中的单个进程。
- **端点（endpoint）**：表示某个服务传入请求的路径，例如HTTP URI路径（/user/api/userinfo）或gRPC服务类+方法签名。



## 目标设计

skywalking核心设计目标如下：

- **保持可观察性**，无论待监控的目标系统如何部署，skywalking都能提供一个解决方案，或者集成方案保持对目标系统的观察。基于这个原因，skywalking提供了几个运行时形态和探针。
- **拓扑图，Metric和Trace**，了解分布式系统的第一步，应该是从拓扑图开始。它把整个复杂的系统可视化为一个简单的拓扑图。在拓扑图之下，我们需要更多关于服务，实例，端点和调用的metrics信息。例如当端点延迟变长，你需要查看最慢的地方从而找出延迟原因。正如你所见，它们是从一个大的图片到细节，这些都是必须的。SkyWalking整合并提供了许多特性，从而让其成为可能，并易于理解。
- **轻量级**. 主要有两个部分需要轻量级。 (1) 在探针上，我们仅依赖网络通信框架，优先gRPC。如此依赖，探针能尽可能的小，从而避免包冲突，和对VM的压力。 (2)作为一个可观察性平台，它在你的工程环境中是二级和三级系统。因此，skywalking用它们自己的轻量级框架构建后端核心，这样的话用户不需要部署大数据技术平台并维护它们，skywalking在技术栈上应该是简单的。
- **可插拔**，skywalking核心团队提供了很多默认实现，但是肯定还不够，也不可能覆盖每一种场景。所以，许多特性支持可插拔，例如底层存储。
- **可移植性**，skywalking能够运行在许多环境上，包括（1）使用传统的注册中心，比如Eureka。（2）用RPC框架包括服务发现，例如SpringCloud，dubbo。（3）使用ServiceMesh这样的现代基础架构。（4）使用云服务。（5）跨云部署。在这些环境下skywalking都应该能运行良好。
- **互操作**，可观察性是一个很大的landscape，skywalking不可能支持所有，即使通过它的社区也不行。所以，skywalking支持和其他的OSS系统相互协作。比如Zipkin，Jaeger，OpenTracing, OpenCensus。skywalking要能够接收并理解它们的数据格式。

## 为什么选择skywalking

`探针的性能消耗`

APM组件服务的影响应该做到足够小。服务调用埋点本身会带来性能损耗，这就需要调用跟踪的低损耗，实际中还会通过配置采样率的方式，选择一部分请求去分析请求路径。

`代码的侵入性`
即也作为业务组件，应当尽可能少入侵或者无入侵其他业务系统，对于使用方透明，减少开发人员的负担。

可扩展性
一个优秀的调用跟踪系统必须支持分布式部署，具备良好的可扩展性。

数据的分析
数据的分析要快 ，分析的维度尽可能多。界面能呈现出来的数据和报表需要强大。

`支持多种语言`
微服务经常不只有一种语言，比如公司就有PHP，Java，NodeJS等语言，支持多语言的就显得很必要。

探针的性能消耗、代码的侵入性、可扩展性、数据的分析、支持多种语言 这几个都可以作为链路跟踪系统的选型指标。其中标记颜色的几个指标就是我决定使用skywalking的原因。

