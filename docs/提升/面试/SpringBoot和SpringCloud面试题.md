---
tags:
	- 基础面试题
categories: 基础面试题
title: SpringBoot和SpringCloud面试题
---
### Spring Boot 与 Spring 的区别

- Spring Boot可以建立独立的Spring应用程序；
- 内嵌了如Tomcat，Jetty和Undertow这样的容器，也就是说可以直接跑起来，用不着再做部署工作了。
- 无需再像Spring那样搞一堆繁琐的xml文件的配置；
- 可以自动配置Spring；
- 提供了一些现有的功能，如度量工具，表单数据验证以及一些外部配置这样的一些第三方功能；
- 提供的POM可以简化Maven的配置；

<!--more-->

### SpringBoot 的自动配置是怎么做的

> Spring boot 的所有自动化配置的实现都在 spring-boot-autoconfigure 依赖中，通过@EnableAutoConfiguration 核心注解初始化，并扫描 ClassPath 目录中自动配置类对应依赖。并对对应的组件依赖按一定规则获取默认配置并自动初始化所需要的 Bean。



先答为什么需要自动配置？
顾名思义，自动配置的意义是利用这种模式代替了配置 XML 繁琐模式。以前使用 Spring MVC ，需要进行配置组件扫描、调度器、视图解析器等，使用 Spring Boot 自动配置后，只需要添加 MVC 组件即可自动配置所需要的 Bean。所有自动配置的实现都在 spring-boot-autoconfigure 依赖中，包括 Spring MVC 、Data 和其它框架的自动配置。

接着答spring-boot-autoconfigure 依赖的工作原理?
spring-boot-autoconfigure 依赖的工作原理很简单，通过 @EnableAutoConfiguration 核心注解初始化，并扫描 ClassPath 目录中自动配置类对应依赖。比如工程中有木有添加 Thymeleaf 的 Starter 组件依赖。如果有，就按按一定规则获取默认配置并自动初始化所需要的 Bean。

ioc的思想最核心的地方在于，**资源不由使用资源的双方管理，而由不使用资源的第三方管理**，这可以带来很多好处： 

- 第一，资源集中管理，实现资源的可配置和易管理。 
- 第二，降低了使用资源双方的依赖程度，也就是我们说的耦合度。

### 什么是springboot

    用来简化spring应用的初始搭建以及开发过程 使用特定的方式来进行配置（properties或yml文件） 
    创建独立的spring引用程序 main方法运行
    嵌入的Tomcat 无需部署war文件
    简化maven配置
    自动配置spring添加对应功能starter自动化配置               

### springboot常用的starter有哪些

    spring-boot-starter-web 嵌入tomcat和web开发需要servlet与jsp支持
    spring-boot-starter-data-jpa 数据库支持
    spring-boot-starter-data-redis redis数据库支持
    spring-boot-starter-data-solr solr支持
    mybatis-spring-boot-starter 第三方的mybatis集成starter      

### springboot自动配置的原理

    在spring程序main方法中 添加@SpringBootApplication或者@EnableAutoConfiguration
    会自动去maven中读取每个starter中的spring.factories文件  该文件里配置了所有需要被创建spring容器中的bean

### springboot读取配置文件的方式

    springboot默认读取配置文件为application.properties或者是application.yml    

### springboot集成mybatis的过程

```xml
添加mybatis的starter maven依赖
   <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
             <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.2.0</version>
    </dependency>
在mybatis的接口中 添加@Mapper注解
在application.yml配置数据源信息
```


​        

### springboot如何添加【修改代码】自动重启功能

添加开发者工具集=====spring-boot-devtools

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-devtools</artifactId>
		<optional>true</optional>
	</dependency>
</dependencies>
```

SpringBoot2.0新增什么

```
1.支持 Java9

2.基于 Spring5 构建

3.Tomcat 升级到 8.5

4.Flyway 升级到 5

5.Hibernate 升级到 5.2

6.Thymeleaf 升级到 3
```

​        

### 什么是微服务

    以前的模式是 所有的代码在同一个工程中 部署在同一个服务器中 同一个项目的不同模块不同功能互相抢占资源
    微服务 将工程根据不同的业务规则拆分成微服务 微服务部署在不同的机器上 服务之间进行相互调用
    Java微服务的框架有 dubbo（只能用来做微服务），spring cloud（提供了服务的发现，断路器等）

### springcloud如何实现服务的注册和发现

    服务在发布时 指定对应的服务名（服务名包括了IP地址和端口） 将服务注册到注册中心（eureka或者zookeeper）
    这一过程是springcloud自动实现 只需要在main方法添加@EnableDisscoveryClient  同一个服务修改端口就可以启动多个实例调用方法：传递服务名称通过注册中心获取所有的可用实例 通过负载均衡策略调用（ribbon和feign）对应的服务


### ribbon和feign区别

    Ribbon添加maven依赖 spring-starter-ribbon 使用@RibbonClient(value="服务名称") 使用RestTemplate调用远程服务对应的方法feign添加maven依赖 spring-starter-feign 服务提供方提供对外接口 调用方使用 在接口上使用@FeignClient("指定服务名")

### Ribbon和Feign的区别：

    Ribbon和Feign都是用于调用其他服务的，不过方式不同。
    1.启动类使用的注解不同，Ribbon用的是@RibbonClient，Feign用的是@EnableFeignClients。
    2.服务的指定位置不同，Ribbon是在@RibbonClient注解上声明，Feign则是在定义抽象方法的接口中使用@FeignClient声明。
    3.调用方式不同，Ribbon需要自己构建http请求，模拟http请求然后使用RestTemplate发送给其他服务，步骤相当繁琐。Feign则是在Ribbon的基础上进行了一次改进，采用接口的方式，将需要调用的其他服务的方法定义成抽象方法即可，
    不需要自己构建http请求。不过要注意的是抽象方法的注解、方法签名要和提供服务的方法完全一致。

### springcloud断路器的作用

    当一个服务调用另一个服务由于网络原因或者自身原因出现问题时 调用者就会等待被调用者的响应 当更多的服务请求到这些资源时导致更多的请求等待 这样就会发生连锁效应（雪崩效应） 断路器就是解决这一问题
                断路器有完全打开状态
                        一定时间内 达到一定的次数无法调用 并且多次检测没有恢复的迹象 断路器完全打开，那么下次请求就不会请求到该服务
    
                半开
                        短时间内 有恢复迹象 断路器会将部分请求发给该服务 当能正常调用时 断路器关闭
                关闭
                        当服务一直处于正常状态 能正常调用 断路器关闭
### 服务注册发现组件Eureka工作原理

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412111608-386935.jpg)



### 服务网关组件Zuul工作原理

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412111643-879260.jpg)

### 跨域时序图

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412111757-854161.jpg)

### Eureka与Ribbon整合工作原理

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412111845-373920.jpg)

### 解决分布式一致性

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412111915-868350.jpg)

### 级联故障流程

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412111943-135952.jpg)

### 断路器组件Hystrix工作原理

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412112020-566111.jpg)

### 分布式追踪Sleuth工作原理

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412112049-558890.jpg)

### SpringBoot自动配置工作原理

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412112116-418572.jpg)

### 什么是SpringCloud

Spring cloud流应用程序启动器是基于Spring Boot的Spring集成应用程序，提供与外部系统的集成。Spring cloud Task，一个生命周期短暂的微服务框架，用于快速构建执行有限数据处理的应用程序。

### 使用Spring Cloud有什么优势

使用Spring Boot开发分布式微服务时，我们面临以下问题

- 与分布式系统相关的复杂性-这种开销包括网络问题，延迟开销，带宽问题，安全问题。
- 服务发现-服务发现工具管理群集中的流程和服务如何查找和互相交谈。它涉及一个服务目录，在该目录中注册服务，然后能够查找并连接到该目录中的服务。
- 冗余-分布式系统中的冗余问题。
- 负载平衡 –负载平衡改善跨多个计算资源的工作负荷，诸如计算机，计算机集群，网络链路，[中央处理单元](https://www.baidu.com/s?wd=%E4%B8%AD%E5%A4%AE%E5%A4%84%E7%90%86%E5%8D%95%E5%85%83&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，或磁盘驱动器的分布。
- 性能-问题 由于各种运营开销导致的性能问题。
- 部署复杂性-Devops技能的要求。

### 服务注册和发现是什么意思？Spring Cloud如何实现？

当我们开始一个项目时，我们通常在属性文件中进行所有的配置。随着越来越多的服务开发和部署，添加和修改这些属性变得更加复杂。有些服务可能会下降，而某些位置可能会发生变化。手动更改属性可能会产生问题。 Eureka服务注册和发现可以在这种情况下提供帮助。由于所有服务都在Eureka服务器上注册并通过调用Eureka服务器完成查找，因此无需处理服务地点的任何更改和处理。

### 负载平衡的意义什么

在计算中，负载平衡可以改善跨计算机，计算机集群，网络链接，中央处理单元或磁盘驱动器等多种计算资源的工作负载分布。负载平衡旨在优化资源使用，最大化吞吐量，最小化响应时间并避免任何单一资源的过载。使用多个组件进行负载平衡而不是单个组件可能会通过冗余来提高可靠性和可用性。负载平衡通常涉及专用软件或硬件，例如[多层交换机](https://www.baidu.com/s?wd=%E5%A4%9A%E5%B1%82%E4%BA%A4%E6%8D%A2%E6%9C%BA&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)或域名系统服务器进程。

### 什么是Hystrix？它如何实现容错？

Hystrix是一个延迟和容错库，旨在隔离远程系统，服务和第三方库的访问点，当出现故障是[不可避免](https://www.baidu.com/s?wd=%E4%B8%8D%E5%8F%AF%E9%81%BF%E5%85%8D&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)的故障时，停止级联故障并在复杂的分布式系统中实现弹性。

通常对于使用微服务架构开发的系统，涉及到许多微服务。这些微服务彼此协作。

思考以下微服务

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412112805-800691.jpg)

假设如果上图中的微服务9失败了，那么使用传统方法我们将传播一个异常。但这仍然会导致整个系统崩溃。

随着微服务数量的增加，这个问题变得更加复杂。微服务的数量可以高达1000.这是hystrix出现的地方 我们将使用Hystrix在这种情况下的Fallback方法功能。我们有两个服务employee-consumer使用由employee-consumer公开的服务。

简化图如下所示

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412112825-592667.jpg)

现在假设由于某种原因，employee-producer公开的服务会抛出异常。我们在这种情况下使用Hystrix定义了一个回退方法。这种后备方法应该具有与公开服务相同的返回类型。如果暴露服务中出现异常，则回退方法将返回一些值。

### 什么是Hystrix断路器？我们需要它吗？

由于某些原因，employee-consumer公开服务会引发异常。在这种情况下使用Hystrix我们定义了一个回退方法。如果在公开服务中发生异常，则回退方法返回一些默认值。

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412112904-253034.jpg)

如果firstPage method() 中的异常继续发生，则Hystrix电路将中断，并且员工使用者将一起跳过firtsPage方法，并直接调用回退方法。 断路器的目的是给第一页方法或第一页方法可能调用的其他方法留出时间，并导致异常恢复。可能发生的情况是，在负载较小的情况下，导致异常的问题有更好的恢复机会 。

![](http://blogimg.nos-eastchina1.126.net/shenwf20190412112922-392868.jpg)

### 什么是Netflix Feign？它的优点是什么？

Feign是受到Retrofit，JAXRS-2.0和WebSocket启发的java客户端联编程序。Feign的第一个目标是将约束分母的复杂性统一到http apis，而不考虑其稳定性。在employee-consumer的例子中，我们使用了employee-producer使用REST模板公开的REST服务。

但是我们必须编写大量代码才能执行以下步骤

- 使用功能区进行负载平衡。
- 获取服务实例，然后获取基本URL。
- 利用REST模板来使用服务。 前面的代码如下

```java
@Controller
public class ConsumerControllerClient {
 
@Autowired
private LoadBalancerClient loadBalancer;
 
public void getEmployee() throws RestClientException, IOException {
 
    ServiceInstance serviceInstance=loadBalancer.choose("employee-producer");
 
    System.out.println(serviceInstance.getUri());
 
    String baseUrl=serviceInstance.getUri().toString();
 
    baseUrl=baseUrl+"/employee";
 
    RestTemplate restTemplate = new RestTemplate();
    ResponseEntity<String> response=null;
    try{
    response=restTemplate.exchange(baseUrl,
            HttpMethod.GET, getHeaders(),String.class);
    }catch (Exception ex)
    {
        System.out.println(ex);
    }
    System.out.println(response.getBody());
}
```

之前的代码，有像NullPointer这样的例外的机会，并不是最优的。我们将看到如何使用Netflix Feign使呼叫变得更加轻松和清洁。如果Netflix Ribbon依赖关系也在类路径中，那么Feign默认也会负责负载平衡。

### 什么是Spring Cloud Bus？我们需要它吗？

考虑以下情况：我们有多个应用程序使用Spring Cloud Config读取属性，而Spring Cloud Config从GIT读取这些属性。

下面的例子中多个员工生产者模块从

### 什么是微服务架构

```
    1.微服务是系统架构上的一种设计风格，他的主旨是将一个原本独立的系统拆分成多个小型服务

	2.这些小型服务都在各自独立的进程中运行

	3.服务之间通过基于HTTP的RESTful API进行通信协作

	4.被拆分的每个微服务都围绕系统中的某一项或一些耦合度较高的业务功能进行构建

	5.并且每个微服务都维护着自身的数据存储，业务开发，自动化测试案例以及独立部署机制

	6.由于有了轻量级的通信协作基础，所以这些微服务可以使用不同的语言来编写
```

### 微服务之间是如何独立通讯的？

```
同步：RPC,REST等；使用HTTP的RESTful API或轻量级的消息发送协议，实现信息传递与服务调用的触发。

	异步：消息队列，要考虑可靠传输，高性能，以及编程模型的编号等；通过在轻量级的消息总线上传递消息，类似Rabbitmq等一些提供可靠异步

			交换的中间件。
```

### 谈谈你对SpringBoot,SpringCloud的理解

### 什么是服务熔断，什么是服务降级？

扇出与雪崩响应：

多位微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他微服务，这就是所谓的"扇处"。如果扇处的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃—所谓的"雪崩效应"。

服务熔断：

熔断机制是应对雪崩效应的一种微服务链路保护机制！！！当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回“错误”的响应信息。当检测到该节点微服务调用响应正常后恢复链路。在SpringCloud框架里熔断机制通过Hystrix实现。Hystrix会监控微服务调用的状况，当失败的调用到一定阈值就会启动熔断机制(默认是5秒内20次调用失败就会启动熔断机制)。但服务熔断来处理雪崩效应是非常可怕的，不可取的。第一如果有多个方法，对应的FallBack方法也会随之膨胀；第二异常处理与业务逻辑高耦合，完全不符合Spring AOP面向切面的思想。

服务降级：

整体资源快不够了，忍痛将某些服务先关掉，待渡过难关，再开启回来。服务降级处理是在客户端(消费者)完成的与服务端没有关系。

个人感觉：多个微服务之间进行调用，假设微服务A调用微服务B和微服务C,微服务B和微服务C又调用其他微服务，这就是所谓的“扇出”。如果扇出链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃---所谓的“雪崩效应”。

服务熔断和服务降级都是为了解决雪崩效应，只是二者实现方式不同：

服务熔断在服务提供端，进行服务降级，进而熔断该节点微服务的调用，快速返回“失败”的响应信息。当检测到 该节点微服务调用响应正常恢复链路。当失败的次数达到一定阈值就会启动熔断机制服务降级，处理方式在客户端完成与服务端有没有关系，整体资源快不够了，忍痛将某些服务先关掉，待度过难关再开启回来。

服务监控：htstrix dashbord

### 微服务的优缺点？

优点：

每个服务足够内聚，足够小，代码容易理解这样能聚焦一个指定的业务功能或业务需求

开发简单、开发效率提高，一个服务可能就是专一的只干一件事。

微服务能够被小团队独立开发，这个团队可以使2到5人的开发人员组成。

微服务是松耦合的，是有功能意义的服务，无论实在开发阶段或部署阶段都是独立的。

微服务能使用不同的语言开发。

微服务只是业务逻辑的代码，不会和HTML,CSS或其他界面组件混合。

每个微服务都有自己的存储能力，可以有自己的数据库。也可以有统一的数据库。

缺点：

开发人员要处理分布式系统的复杂性

多服务运维难度，随着服务的增加，运维的压力也在增加

系统部署依赖

服务间通信成本

数据一致性

系统集成测试

性能监控

### 分布式事务？

使用lcn

### 谈谈你所知道的微服务技术栈有哪些？

### 什么是springboot

Spring Boot的宗旨并非要重写Spring或是替代Spring,而是希望通过设计大量的自动化配置等方式来简化Spring原有样板化的配置，用来简化Spring初始构建以及开发工程的，使用特定的方式来进行配置

1. 大量的自动化配置来简化Spring原有样板化的配置，
2. 简化Maven配置，是依赖管理变得更加简单(添加对应功能的starter)，
3. 快速开发(创建独立的Spring引用程序，main方法运行)，
4. 轻松部署(内嵌的tomcat，不需要部署war文件，直接运行jar文件)

### SpringBoot自动配置的原理

在spring程序main方法中 添加@SpringBootApplication或者@EnableAutoConfiguration会自动去maven中读取每个starter中的spring.factories文件  该文件里配置了所有需要被创建spring容器中的bean

### springboot读取配置文件的方式

springboot默认读取配置文件为application.properties或者是application.yml

### springCloud如何实现服务的注册和发现的

服务在发布时 指定对应的服务名（服务名包括了IP地址和端口） 将服务注册到注册中心（eureka或者zookeeper）这一过程是springcloud自动实现 只需要在main方法添加@EnableDisscoveryClient  同一个服务修改端口就可以启动多个实例调用方法：传递服务名称通过注册中心获取所有的可用实例 通过负载均衡策略调用（ribbon和feign）对应的服务

### 对外服务接口带有Request参数的请求，带有Header信息的请求，带有RequestBody的请求以及请求响应体是一个对象的请求，Spring Cloud Feign接口调用如何绑定参数？

注意：在各参数绑定时，@RequestParam,@RequestHeader等可以指定参数名称的注解，它的value不能少。在SpringMVC程序中，这些注解会根据参数名作为默认值，但是在Feign中绑定参数必须通过value属性来指明具体的参数名，不然抛异常，value不能少。

###  Feign与Ribbon，Hystrix的关系

Feign的客户端负载均衡是通过SpringCloud Ribbon实现的，所以可以直接通过配置Ribbon客户端的方式来自定义各个服务客户端的参数。Feign引入了服务保护与容错的工具Hystix,默认情况下，Feign将所有Feign客户端方法都封装到Hystrix命令中进行服务保护。

### Ribbon全局配置，Ribbon指定服务配置，Ribbon重要的配置参数，Ribbon重试机制

Ribbon全局配置：

```
ribbon.ConnectTimeout = 500
ribbon.ReadTimeout = 5000
```

Ribbon指定服务配置：

```
在SpringCloud Feign中针对各个服务客户端进行个性化配置的方式与在SpringCloud Ribbon的配置方式是一样的，都采用<Client>.ribbon.key=value的格式进行设置。服务名就是@FeignClients指定的name或value。
```

Ribbon重要的配置参数：

```
#建立连接的超时时间
HELLO-SERVICE.ribbon.ConnectTimeout = 500
#连接超时时间
HELLO-SERVICE.ribbon.ReadTimeout = 2000
#所有操作都重试	
HELLO-SERVICE.ribbon.OkToRetryOnAllOperations = true
#重试发生，更换节点数最大值
HELLO-SERVICE.ribbon.MaxAutoRetriesNextServer = 2
#单个节点重试最大值
HELLO-SERVICE.ribbon.MaxAutoRetries = 1
```

重试机制：

```
SpringCloud Feign默认实现了重试机制，重试时间计算方式。Feign重试机制默认关闭，以免和ribbon冲突。
ribbon:
			ReadTimeout: 3000
			ConnectTimeout: 3000
			MaxAutoRetries: 1 #同一台实例最大重试次数,不包括首次调用
			MaxAutoRetriesNextServer: 1 #重试负载均衡其他的实例最大重试次数,不包括首次调用
			OkToRetryOnAllOperations: false  #是否所有操作都重试 
根据上面的参数计算重试的次数：MaxAutoRetries+MaxAutoRetriesNextServer+(MaxAutoRetries *MaxAutoRetriesNextServer) 即重试3次则一共产生4次调用。如果在重试期间，时间超过了hystrix的超时时间，便会立即执行熔断，fallback。所以要根据上面配置的参数计算hystrix的超时时间，使得在重试期间不能达到hystrix的超时时间，不然重试机制就会没有意义
hystrix超时时间的计算： 
(1 + MaxAutoRetries + MaxAutoRetriesNextServer) * ReadTimeout 即按照以上的配置 hystrix的超时时间应该配置为 （1+1+1）*3=9秒
当ribbon超时后且hystrix没有超时，便会采取重试机制。当OkToRetryOnAllOperations设置为false时，只会对get请求进行重试。如果设置为true，便会对所有的请求进行重试，如果是put或post等写操作，如果服务器接口没做幂等性，会产生不好的结果，所以OkToRetryOnAllOperations慎用。如果不配置ribbon的重试次数，默认会重试一次
注意：
	默认情况下,GET方式请求无论是连接异常还是读取异常,都会进行重试
	非GET方式请求,只有连接异常时,才会进行重试
```

###  Ribbon超时与Hystrix超时区别

让Hystrix超时时间大于Ribbon超时时间,否则Hystrix命令直接熔断，重试机制就没意义了

### 服务降级，服务容错，断路器三者关系

服务降级是服务容错的重要功能，服务降级的实现只需为Feign客户端定义接口编写一个具体的接口实现类，每个重写方法的实现逻辑都可以用来定义相应的服务降级逻辑。每一个服务接口的断路器实际就是实现类中重写函数的实现。

### Zuul网关服务主要解决的问题

从运维人员角度看：

客户端请求通过F5,Nginx等设施的路由和负载均衡分配后，被转发到不同的服务实例上。当有实例增减或是IP地址变动时，运维人员需要手工维护路由规则和服务实例。当系统复杂时，就需要一套机制有效降低维护路由规则与服务实例列表的难度。	

从开发人员角度看：

为了保证对外服务的安全性，所有对外的微服务接口往往都会有一定的权限校验机制。为了防止客户端在发起请求时被篡改等安全方面的考虑，需要签名校验，对校验的修改，扩展，优化，就需要解决各个前置校验的冗余问题。

API网关是整个微服务架构系统的入口，所有的外部客户端访问都需要经过他来进行调度和过滤。它处理要实现请求路由，负载均衡，校验过滤等功能之外，还需要更多的能力，比如与服务治理框架结合，请求转发时的熔断机制，服务的聚合等一系列高级功能。

### Zuul如何维护路由规则和服务实例列表，如何解决签名校验，登录校验在微服务架构中的冗余问题

服务实例列表的维护：Zuul与服务治理框架整合，将自身注册为服务治理下的应用，从服务中心获取所有其他微服务的实例信息。

路由规则的维护：Zuul默认会通过以服务名的作为contextPath的方式来创建路由映射。

校验逻辑在本质上与微服务应用自身的业务没有多大关系，完全可以把它们独立成一个单独的服务存在，只是这个微服务不是给各个微服务调用，而是在API网关服务上进行统一调用来对微服务接口做前置过滤，以实现对微服务接口的拦截和过滤。Zuul提供了一套过滤器机制，通过创建各种过滤器，然后指定哪些规则的请求需要执行校验逻辑，只有通过的校验才会被路由到具体的微服务接口，不然返回错误信息。

### Zuul的请求过滤功能(安全校验，权限控制)

通过继承ZuulFilter抽象类并重写了下面4个方法，来实现自定义的过滤器

filterType:过滤器类型，决定过滤器在请求的哪个生命周期中执行，具体如下：

pre:可以在请求被路由之前调用

routing:在路由请求时被调用

post:在routing和error过滤器之后被调用

error:处理请求时发生错误时被调用

filterOrder:过滤器执行顺序，当请求存在多个过滤器时，需要根据该方法返回值依次执行，数值越小优先级越高

shouldFilter:判断该过滤器是否需要被执行。实际中我们利用这个函数来制定过滤器的有效范围

run:过滤器具体的执行逻辑

```java
public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        logger.info("send {} request to {}" + request.getMethod(), request.getRequestURL().toString());
        Object accessToken = request.getParameter("accessToken");
        if (accessToken == null) {
            logger.warn("access token is empty!");
            ctx.setSendZuulResponse(false);	//不对请求进行路由
            ctx.setResponseStatusCode(401);	//设置错误码
            return null;					//请求返回值为null
        }
        logger.info("access token ok");
        return null;						//实际返回值路由成功后的返回值
    }
```

### Zuul两个核心功能

1.作为系统的统一入口，屏蔽了系统内部各个微服务的细节

2.可以与服务治理框架结合，实现自动化的服务实例以及负载均衡的路由转发功能

3.可以实现接口的权限校验与微服务业务逻辑的解耦。

4.通过网关中的过滤器，在各个生命周期中去校验请求的内容，将原本在对外服务层做的校验前移，保证了微服务的无状态性，同时降低了微服务的测试难度，让服务本身更专注业务逻辑的处理。

### Zuul完成路由工作原理

Zuul与服务治理框架整合，在其帮助下，API网关服务本身就已经维护了系统中所有serviceId与实例地址的映射关系。当有外部请求到达API网关的时候，根据请求的url路径找到最佳匹配的path规则，API网关就可以知道要将该请求路由到哪个具体的serviceId上去。由于在API网关中已经知道serviceId对应服务实例的地址清单，那么只需要通过Ribbon的负载均衡策略，直接在这些实例清单中选择一个具体的实例进行转发就能完成路由工作。

### Zuul对Cookie和头信息的处理

默认情况下，SpringCloud Zuul在请求路由时，会过滤掉HTTP请求头信息的一些敏感信息，防止他们被传递到下游的外部服务器。默认的敏感头信息通过zuul.sensitiveHeaders参数定义，包括Cookie，Set-Cookie，Authorization三个属性。Cookie在Zuul网关中默认是不会传递的，引发问题：Web应用无法实现登录和鉴权。

全局设置：

zuul.sensitive-headers=			#是所有经过zuul的敏感头信息有效

指定路由设置：

zuul.routes.<routeName>.sensitive-headers=				#对指定路由开启自定义敏感头

zuul.routes.<routeName>.custom-sensitive-headers=true	#将指定路由的敏感头设置为空

### Zuul异常处理

SpringBoot全局异常处理

实现SendErrorFilter过滤器的处理异常响应结果

### Zuul动态加载

微服务架构中，由于API网关服务担负着外部访问统一入口的重任，与其他应用不同，任务关闭应用和重启应用的操作都会使系统对外服务停止，对于高可用的系统来说绝对不能被允许。所以网关服务必须具备动态更新内部逻辑的能力，比如动态修改路由规则，动态添加/删除过滤器等。

动态路由：对于路由规则的控制几乎都可以在配置文件中完成，所以很自然的将它与SpringCloud Config动态刷新机制联系到一起。只需将API网关服务的配置文件通过Config连接的Git仓库存储和管理，就能轻松实现动态刷新路由规则的功能。

/routes	 接口来获取当前网关上的路由规则

/refresh 接口发送POST请求来刷新配置信息

动态过滤器；

### Config:分布式配置中心

### Config介绍

为分布式系统中的微服务应用提供集中化的外部配置支持，分为服务端与客户端。其中服务端也称为分布式配置中心，用来连接配置仓库并未客户端提供获取配置信心，加密/解密信息等访问接口；而客户端则是微服务架构中的各个微服务应用或基础设施，通过指定的配置中心来管理应用资源与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。

### 客户端应用从配置管理中获取配置信息的执行流程

1.客户端应用启动时，根据bootstrap.properties中配置的应用名{application},环境名{profile},分支名{label}，向Config Server请求获取配置信息

2.Config Server根据自己维护的Git仓库信息和客户端传递过来的配置定位信息去查找配置信息

3.通过git clone命令将找到的配置信息下载到Config Server的文件系统中

4.Config Server创建Spring的ApplicationContext实例，并从Git本地仓库中加载配置文件，最后将这些配置内容读取出来返回给客户端应用

5.客户端应用在获得外部配置文件后加载到客户端的ApplicationContext实例中，该配置内容的优先级大于客户端jar包内部的配置内容，所以jar包中重复的内容将不再被加载

### Git配置仓库，SVN配置仓库，本地仓库，高可用配置

高可用配置两种方式：传统模式(不需要注册中心),服务模式(将Config Server作为一个普通的微服务应用，微服务应用通过配置中心的服务名获取配置信息)

#### 健康监测属性覆盖

Config Server可以使用属性覆盖Git配置的属性信息，可以为SpringCloud应用配置一些共同属性或默认属性

### 安全保护，加密解密，高可用配置

加密解密：对称加密与非对称加密

配置文件中数据库的用户名与密码等敏感信息不应该对外暴露，就可以使用加密方式，对密码用户名进行加密。当微服务客户端加载配置时，配置中心会自动为带有{cipher}前缀的值进行解密。

高可用配置-服务端：

```yml
           server:
			  port: 13001
			spring:
			  application:
				name: config-server
			  cloud:
				config:
				  server:
					git:
					  uri: https://gitee.com/SunYanGang/cloud-config-server.git
					  search-paths: /SunYanGang/cloud-config-server
					  password: mayun_6688
					  username: 18811748164
```

高可用配置-客户端：

```yml
		    eureka:
			  client:
				fetch-registry: true
				register-with-eureka: true
				service-url:
				  defaultZone: http://eurekaserver01:8080/eureka,http://eurekaserver02:8081/eureka,http://eurekaserver03:8082/eureka
			server:
			  port: 12001
			spring:
			  application:
				name: api-gateway               #指定应用名称
			  cloud:
				config:
				  profile: dev                  #指定配置文件的环境
				  label: master                 #指定git
				  discovery:
					enabled: true               #开启通过服务来访问Config Server的功能
					service-id: config-server   #指定服务中心注册的服务名
			management:
			  security:
				enabled: false
			eureka:
			  client:
				service-url:
				  defaultZone: http://eurekaserver01:8080/eureka,http://eurekaserver02:8081/eureka,http://eurekaserver03:8082/eureka
				fetch-registry: true
				register-with-eureka: true
```

### 失败快速响应与重试

SpringCloud Config的客户端会预先加载很多其他信息，然后再开始连接Config Server进行属性的注入。连接Config Server发费较长的时间，所以我们希望快速知道当前应用是否能顺利地从Config Server获取到配置信息。还有重试机制，避免一些间歇性问题引起的失败导致客户端应用无法启动的情况。

### 获取远程配置

application:应用名

profile:应用环境

label:分支名

通过客户端配置方式加载的内容如下：

spring.application.name:对应配置文件中的{application}内容

spring.application.name:对应配置文件中的{application}内容

spring.cloud.config.label:对应分支内容，如不配置，默认为master

### 动态刷新配置

配置中心的客户端pom.xml文件新增 spring-boot-starter-actuator监控模块，其中包含了/refresh端点的实现，该端点将用于实现客户端应用配置信息的重新获取与刷新。

POST请求向客户端发送 http:*//配置中心客户端：端口/refresh*

### consul,eureka,ribbon,hystrix,feign,zuul,config,bus(rabbitmq)理解与问题

#### Eureka理论：

1.服务治理：为了解决微服务架构中的服务实例维护问题(传统的一个服务调用另一个服务，需要静态配置另一个服务的接口路径，一旦IP，端口等变化，需要手工修改)，产生了大量的服务治理框架和产品。这些框架和产品的实现都是围绕着服务注册和服务发现机制来完成对微服务应用实例的自动化管理。

服务注册：服务治理框架通常会构建一个注册中心，每个注册单元向注册中心登记自己提供的服务，将主机与端口号，版本号，协议等一些附加信息告知注册中心，注册中心按照服务名组织服务清单。当服务都注册到注册中心并启动后，注册中心还需要以心跳的方式检测清单中的服务是否可用，若不可用则从服务清单中剔除，达到排除故障服务的效果。

服务发现：由于在服务治理框架下运作，服务之间的调用不再通过具体的实例地址来实现，而是通过向服务名发起请求调用实现。所以服务调用方在调用其他服务的时候，并不知道具体的服务实例位置。因此调用方需要向注册中心咨询服务，并获取所有的服务实例清单，以实现对具体的服务实例的访问。调用其他服务的时候，可能对应多个服务实例，服务调用方从这多个实例中以某种轮询策略取出一个位置来进行服务调用，就是客户端负载均衡。实际框架为了性能等因素，不会采用每次想服务注册中心获取服务的方式，并且不同的应用场景在缓存和服务剔除等机制上也会有一些不同的实现策略。

2.Eureka:

Eureka服务端，也称为服务注册中心。支持高可用配置，提供良好的服务实例可用性，可以应对多种不同的故障场景。如果Eureka以集群模式部署，当集群中有分片出现故障时，那么Eureka就转入自我保护模式。它允许分片故障期间继续提供服务的发现与注册，当故障分片恢复运行时，集群中的分片会把他们的状态再次同步回来。

Eureka客户端，主要处理服务注册和发现的。(重点：服务续约，刷新服务状态)客户端服务通过注解和参数配置的方式，嵌入在客户端应用程序的代码中，在应用程序运行时，Eureka客户端向注册中心注册自身提供的服务并周期性地发送心跳来更新它的服务租约。同时，它也能从服务端查询当前注册的服务信息并把它们缓存到本地并周期性地刷新服务状态。

3.服务治理基础架构的三个核心要素：

服务注册中心：提供服务注册和发现的功能

服务提供者：提供服务的应用，自身注册到注册中心，以便其他服务发现和调用

服务消费者：从注册中心获取服务，从而使消费者可以知道去何处调用所需要的服务

4.服务提供者，服务消费者，服务注册中心各自重要的通信行为

注意服务续约，服务剔除，自我保护，服务获取，服务同步，服务续约和服务剔除有关系，服务剔除和服务注册中心自我保护有关系获取服务缓存到本地有个更新时间间隔。

4.1 服务提供者：

服务注册：服务提供者在启动服务时发送Rest请求将自己注册到服务注册中心，同时带上自身的一些元数据信息。注册中心接到Rest请求后，将元数据信息存储到双层Map中，其中第一层的key是服务名，第二层的key是具体服务的实例名。

服务同步：指"高可用的注册中心"集群中多个注册中心之间的同步，当服务提供者发送注册请求到一个注册中心时，该注册中心会将请求转发给集群中相连的其他注册中心，从而实现注册中心之间服务的同步。

服务续约：服务提供者会维护一个心跳用来持续告诉注册中心自己还活着，防止服务注册中心将该服务实例从服务列表中剔除。有两个比较重要的属性：续租任务的调用时间间隔，默认30s（客户端定义）;服务失效的时间间隔，默认90s（服务端定义）

4.2 服务消费者：

获取服务：启动服务消费者的时候，它会发送一个REST请求给服务注册中心，来获取上面注册的服务清单。为了性能考虑，服务注册注册中心会维护一个只读的服务清单来返回给客户端，同时客户端缓存清单会每隔30s更新一次。	

服务调用：服务消费者在获取到服务清单后，通过服务名可以获取具体提供服务的实例名和该实例的元数据信息。在Ribbon中会采用默认的轮询方式进行调用，从而实现客户端的负载均衡。

服务下线：关闭和重启服务实例，包括正常关闭和非正常关闭。

4.3 服务注册中心：

失效剔除：服务注册中心启动时创建定时任务，每隔一段时间（默认60s）将清单中超时(默认90s)没有续约的服务剔除出去。对于服务正常下线，会告诉注册中心，而服务由于内存溢出，网络故障等原因使得服务不能正常工作，注册中心并不会收到“服务下线”的请求。

自我保护：服务注册中心在运行期间统计心跳失败比例，若是网络延迟不稳定导致，则会将当前实例注册信息保护起来，不让服务实例过期而被服务剔除。 但会引发问题：消费者拿到已不存在发服务实例报错，因此客户端必须要有容错机制，比如请求重试，断路器等机制。

4.4 Eureka Client负责下面的任务：

向Eureka Server注册服务实例

向Eureka Server服务租约

当服务关闭时向Eureka Server取消租约

查询Eureka Server中的服务实例列表

