---
tags:
	- Spring
categories: Spring
title: 第 17 章 Spring MVC
---
# 第 17 章 Spring MVC

## Spring MVC 的框架设计

<!--more-->

![Spring MVC 框架设计图](http://blogimg.nos-eastchina1.126.net/shenwf20190421060506-224411.jpg)

其中带有阿拉伯数字的说明，是 MVC 框架运行的流程。处理请求先到达控制器（Controller），控制器的作用是进行请求分发，这样它会根据请求的内容去访问模型层（Model）；现今互联网系统中，数据主要从数据库和 NoSQL 中来，而且对于数据库而言往往还存在事务的机制，为了适应这种变化，设计者会将模型层再分成两层，即服务层（Service）和（DAO）;当控制器获取到由模型层返回的数据后，就将数据渲染到视图中，这样就能够展现给用户了。

## Spring MVC 流程

![](http://blogimg.nos-eastchina1.126.net/shenwf20190421074655-629111.jpg)

它是 Spring MVC 运行的全流程，其中图中的阿拉伯数字是其执行流程。在启用 Spring MVC 时，它开始初始化一些重要组件，如 DispactherServlet、HandlerAdapter的实现类 RequestMappingHandlerAdapter 等组件对象。关于这些组件的初始化，可以看 spring-webmvc-xxx.jar 的属性文件 DispatcherServlet.properties，它定义的对象都是在 Spring MVC 开始时就初始化，并且存放在 Spring IOC 容器中。

DispatcherServlet.properties 源码如下：

```properties
# 国际化解析器
org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

# 主题解析器
org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

# HandlerMapping 实例
org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping
	
# 处理器适配器	
org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter

# 处理器异常解析器
org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

# 策略视图名称转换器，当你没有返回视图逻辑名称的时候，通过它可以生成默认的视图名称
org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

# 视图解析器
org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

# FlashMap 管理器。
org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

首先这些组件会在 Spring MVC 得到初始化。

其次是开发控制器（Controller），代码如下：

```java
@Controller
@RequestMapping("/user")
public class UserController {
    //注入用户服务类
    @Autowired
    private UserService userService = null;
    // 展示用户详情
    @RequestMapping("details")
    public ModelAndV工ewdetails(Long工d){
        // 访问模型层得到数据
        User user= userService.getUser(id); 
        // 模型和视图
        ModelAndView mv = new ModelAndView();
        // 定义模型视图
        mv.setViewName("user/details"); 
        // 加入数据模型
        mv.addObject("user",user);
        // 返回模型和视图
        return mv;
    }
}                             
```

这里的注解 @Controller 表明这是一个控制器，然后 @RequestMapping 代表请求路径和控制器（或其他方法）的映射关系，它会在Web 服务器启动 Spring MVC 时，被扫描到 HandlerMapping 的机制中存储，之后在用户发起请求被 DispatcherServlet 拦截后，通过 URI 和其他的条件，通过 HandlerMapper 机制就能找到对应的控制器（或其方法）进行响应。只是通过 HandlerMapping 返回的是一个 HandlerExecutionChain 对象，这个对象的源码如下。HandlerExecutionChain 对象包含一个处理器（handler）。这里的处理器是对控制器（controller）的包装，因为我们的控制器方法就可以读入 HTTP 和上下文的相关参数，然后再传递给控制器方法。而在控制器执行完成返回后，处理器又可以通过配置信息对控制器的返回结果进行处理。

处理器包含了控制器方法的逻辑，此外还有处理器的拦截器（interceptor），这样就能够通过拦截器进一步地增强处理器的功能。

```java
public class HandlerExecutionChain {
    // 日志
    private static final Log logger = LogFactory.getLog(HandlerExecutionChain.class);
    // 处理器
    private final Object handler;
    // 拦截器数组
    private HandlerInterceptor[] interceptors;
    // 拦截器列表
    private List<HandlerInterceptor> interceptorList;
    // 拦截器当前下标
    private int interceptorIndex;
    ...
}
```

得到了处理器（handler），还需要去运行，但是我们有普通HTTP请求，也有按 BeanName 的请求，甚至是 WebSocket ，所以还需要一个适配器去运行 HandlerExecutionChain 对象包含的处理器，就是 HandlerAdapter 接口定义的实现类，HttpRequestHandlerAdapter。