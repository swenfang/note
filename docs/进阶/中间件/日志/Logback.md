---
tags:
	- LogBack
categories: LogBack
title: LogBack 使用
---
![此处输入图片的描述][1]
**26 Seven 2017**
    
## 前言
之前项目一直使用 logback ,现在大概写下了 logback 基础配置。

<!--more-->

## 简介
<code>LogBack</code>是一个日志框架，它是Log4j作者Ceki的又一个日志组件。

<code>LogBack,Slf4j,Log4j</code>之间的关系

slf4j是The Simple Logging Facade for Java的简称，是一个简单日志门面抽象框架，它本身只提供了日志Facade API和一个简单的日志类实现，一般常配合<code>Log4j</code>，<code>LogBack</code>，<code>java.util.logging</code>使用。Slf4j作为应用层的Log接入时，程序可以根据实际应用场景动态调整底层的日志实现框架(Log4j/LogBack/JdkLog...)；

LogBack和Log4j都是开源日记工具库，LogBack是Log4j的改良版本，比Log4j拥有更多的特性，同时也带来很大性能提升。

LogBack官方建议配合Slf4j使用，这样可以灵活地替换底层日志框架。

LogBack的结构
LogBack分为3个组件，logback-core, logback-classic 和 logback-access。
其中logback-core提供了LogBack的核心功能，是另外两个组件的基础。
logback-classic则实现了Slf4j的API，所以当想配合Slf4j使用时，则需要引入这个包。
logback-access是为了集成Servlet环境而准备的，可提供HTTP-access的日志接口。

Log的行为级别：
OFF、
FATAL、
ERROR、
WARN、
INFO、
DEBUG、
ALL
从下向上，当选择了其中一个级别，则该级别向下的行为是不会被打印出来。
举个例子，当选择了INFO级别，则INFO以下的行为则不会被打印出来。


## 获取 Logger 对象

![此处输入图片的描述][2]

我们先从获取 logger 对象开始
``` java
Logger logger = LoggerFactory.getLogger(xxx.class.getName());
```
LoggerFactory 是 slf4j 的日志工厂，获取 logger 方法就来自这里。
``` java
以下代码摘自：org.slf4j.LoggerFactory

public static Logger getLogger(String name) {
    ILoggerFactory iLoggerFactory = getILoggerFactory();
    return iLoggerFactory.getLogger(name);
}
```
这个方法里面有分为两个过程。第一个过程是获取ILoggerFactory，就是真正的日志工厂。第二个过程就是从真正的日志工厂中获取logger。
接下来我们看下到底是怎么获取的
``` java
以下代码摘自：org.slf4j.LoggerFactory

public static ILoggerFactory getILoggerFactory() {
if (INITIALIZATION_STATE == UNINITIALIZED) {
  INITIALIZATION_STATE = ONGOING_INITIALIZATION;
  // 第一次调用会去加载 StaticLoggerBinder.class 文件来决定 LoggerFactory 的实现类
  // （补充下：不同日志包下都有 StaticLoggerBinder 这个类文件，这个会决定 LoggerFactory 初始化那种类型的日志）
  performInitialization();
}
// INITIALIZATION_STATE 值判断是否有加载初始化过 ILoggerFactory 实例。
switch (INITIALIZATION_STATE) {
  case SUCCESSFUL_INITIALIZATION:
    // 返回对应的 ILoggerFactory 实例
    return StaticLoggerBinder.getSingleton().getLoggerFactory();
  case NOP_FALLBACK_INITIALIZATION:
    // 当加载不到一个 StaticLoggerBinder 时，会走这里
    // 返回一个 NOPLoggerFactory 实例
    return NOP_FALLBACK_FACTORY;
  case FAILED_INITIALIZATION:
    // 初始化异常
    throw new IllegalStateException(UNSUCCESSFUL_INIT_MSG);
  case ONGOING_INITIALIZATION:
    // support re-entrant behavior.
    // See also http://bugzilla.slf4j.org/show_bug.cgi?id=106
    return TEMP_FACTORY;
}
throw new IllegalStateException("Unreachable code");
}
```
接下来我们来看下是怎么加载 StaticLoggerBinder.class 文件的

``` java
以下代码摘自：org.slf4j.LoggerFactory

 private final static void bind() {
    try {
      // 加载 StaticLoggerBinder
      Set staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();
      reportMultipleBindingAmbiguity(staticLoggerBinderPathSet);
      // 最后会随机选择一个StaticLoggerBinder.class来创建一个单例
      StaticLoggerBinder.getSingleton();
      // 改变 INITIALIZATION_STATE 值，表示成功初始化 Factory。
      // 并且以后在获取 Logger 的时候并不会再次加载该方法
      INITIALIZATION_STATE = SUCCESSFUL_INITIALIZATION;
      reportActualBinding(staticLoggerBinderPathSet);
      emitSubstituteLoggerWarning();
    } catch (NoClassDefFoundError ncde) {
      String msg = ncde.getMessage();
      if (messageContainsOrgSlf4jImplStaticLoggerBinder(msg)) {
        // 加载不到 StaticLoggerBinder 文件
        INITIALIZATION_STATE = NOP_FALLBACK_INITIALIZATION;
      } else {
        failedBinding(ncde);
        throw ncde;
      }
    } catch (java.lang.NoSuchMethodError nsme) {
      String msg = nsme.getMessage();
      if (msg != null && msg.indexOf("org.slf4j.impl.StaticLoggerBinder.getSingleton()") != -1) {
        // 初始化异常
        INITIALIZATION_STATE = FAILED_INITIALIZATION;
      }
      throw nsme;
    } catch (Exception e) {
      failedBinding(e);
      throw new IllegalStateException("Unexpected initialization failure", e);
    }
  }
```
 加载 StaticLoggerBinder.class
``` java
以下代码摘自：org.slf4j.LoggerFactory.findPossibleStaticLoggerBinderPathSet

private static String STATIC_LOGGER_BINDER_PATH = "org/slf4j/impl/StaticLoggerBinder.class";

···
if (loggerFactoryClassLoader == null) {
    paths = ClassLoader.getSystemResources(STATIC_LOGGER_BINDER_PATH);
  } else {
    paths = loggerFactoryClassLoader
            .getResources(STATIC_LOGGER_BINDER_PATH);
  }
  while (paths.hasMoreElements()) {
    URL path = (URL) paths.nextElement();
    staticLoggerBinderPathSet.add(path);
  }
···
```

当项目中存在多个StaticLoggerBinder.class文件时，运行项目会出现以下日志：
（这里获取的规则就近原则，如果在 maven 先配置 slf4j 而后面在配置 logback，则这里初始化的是 slf4j）
``` java
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/E:/OperSource/org/slf4j/slf4j-log4j12/1.7.12/slf4j-log4j12-1.7.12.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/E:/OperSource/ch/qos/logback/logback-classic/1.1.3/logback-classic-1.1.3.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
```
返回实例
``` java
以下代码摘自：org.slf4j.LoggerFactory.getILoggerFactory

StaticLoggerBinder.getSingleton().getLoggerFactory();
```

## LogBack 配置
``` xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 项目名称配置 -->
    <contextName>example</contextName>
    <!-- 属性 -->
    <property name="APP_Name" value="example" />
    <!-- 统一的时间格式，用于日志头输出 -->
    <timestamp key="timeStyle" datePattern="yyyy-MM-dd HH:mm:ss.SSS"/>
    <!--配置控制台输出,开发环境有-->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoder 默认配置为PatternLayoutEncoder -->
        <encoder>
            <pattern>[%d{timeStyle}] [%cn] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>
    <!--文件输出配置-->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>../logs/${APP_Name}_run.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <fileNamePattern>../logs/${APP_Name}_run.%i.log.zip</fileNamePattern>
            <minIndex>1</minIndex>
            <maxIndex>10</maxIndex>
        </rollingPolicy>
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>10MB</maxFileSize>
        </triggeringPolicy>
        <encoder>
            <pattern>[%d{timeStyle}] [%cn] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="BUSINESS_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>../logs/${APP_Name}_business.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <fileNamePattern>../logs/${APP_Name}_business.%i.business.zip</fileNamePattern>
            <minIndex>1</minIndex>
            <maxIndex>10</maxIndex>
        </rollingPolicy>
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>10MB</maxFileSize>
        </triggeringPolicy>
        <encoder>
            <pattern>[%d{timeStyle}] [%cn] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印错误日志 -->
            <level>WARE</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- 异步输出 -->
    <appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>256</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref="FILE"/>
    </appender>

    <appender name="BUSINESS_ASYNC" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>256</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref="BUSINESS_FILE"/>
    </appender>

    <!-- 为数据库开启显示sql -->
    <logger name="com.cjf.example.repository" level="DEBUG"></logger>

    <logger name="com.cjf" level="INFO"></logger>
    <!--日志的root目录，用于定位日志输出级别-->
    <root level="INFO">
        <appender-ref ref="ASYNC"/>
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```
上面就是一个常用的日志配置模版，下面就从跟节点来解析每个节点

**1.configuration**  
scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
scanPeriod：监测配置文件是否有修改的时间间隔，默认 60s。
debug：是否打印 logback 内部日志，默认 false.
**2.contextName** 项目名称
logger 上下文容器名称，默认 ‘default’，用于区分不同应用程序的记录。（可以在日志输出的时候将项目名称打印处理方便系统间交互 比如上面配置的 <code>%cn</code>）
**3.property** 设置变量
定义变量后，可以使<code>${}</code>来使用变量。
**4.timestamp** 设置时间戳格式
key:标识此<timestamp> 的名字；datePattern：设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循Java.txt.SimpleDateFormat的格式。
**5.logger** 
用来设置某一个包或者具体的某一个类的日志打印级别、以及指定<appender>。<loger>仅有一个name属性，一个可选的level和一个可选的addtivity属性。
name: 用来指定受此loger约束的某一个包或者具体的某一个类。
level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。如果未设置此属性，那么当前loger将会继承上级的级别。
addtivity:是否向上级loger传递打印信息。默认是true。
loger可以包含零个或多个appender-ref元素，标识这个appender将会添加到这个loger。
**6.root** 
也是 loger 元素，但是它是根 loger。只有一个 level 属性，应为已经被命名为 <code>"root"</code>.
root 可以包含零个或多个 appender-ref 元素，标识这个 appender 将会添加到这个 loger。

### 什么是 Appender？
logback 将日志记录事件写入到名为 appender 的组件的任务,不同 appender 决定了日志的记录方式。
appender 有多种实现的方式，下面简单介绍几种比较常用的配置。 [更多详情请参考官方文档][3]

**ConsoleAppender**
把日志添加到控制台，有以下子节点：
  encoder：对日志进行格式化
  target：字符串 System.out 或者 System.err ，默认 System.out;
  withJansi：日志彩色输出


例如
``` xml
<configuration>  
  
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">  
    <encoder>  
      <pattern>[%d{timeStyle}] [%cn] %-5level %logger{35} - %msg%n</pattern>  
    </encoder>  
  </appender>  
  
  <root level="DEBUG">  
    <appender-ref ref="STDOUT" />  
  </root>  
</configuration>
```

**RollingFileAppender**
滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。
file：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
append：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。 
encoder：对记录事件进行格式化。
rollingPolicy：当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名。
triggeringPolicy：告知 RollingFileAppender 合适激活滚动。
prudent：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空。

　
rollingPolicy 有两种类型，分别是 TimeBasedRollingPolicy，FixedWindowRollingPolicy。

例如 TimeBasedRollingPolicy 配置
``` xml
<!--输出到文件-->
<appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${log.path}</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
		<fileNamePattern>logback.%d{yyyy-MM-dd}.log</fileNamePattern>
        <maxHistory>30</maxHistory>
		<totalSizeCap>1GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```
fileNamePattern 定义了日志的切分方式——把每一天的日志归档到一个文件中
maxHistory 表示只保留最近30天的日志，以防止日志填满整个磁盘空间。
totalSizeCap 用来指定日志文件的上限大小，例如设置为1GB的话，那么到了这个值，就会删除旧的日志。

例如 FixedWindowRollingPolicy 配置
``` xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>../logs/${APP_Name}_run.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
        <fileNamePattern>../logs/${APP_Name}_run.%i.log.zip</fileNamePattern>
        <minIndex>1</minIndex>
        <maxIndex>10</maxIndex>
    </rollingPolicy>
    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
        <maxFileSize>10MB</maxFileSize>
    </triggeringPolicy>
    <encoder>
        <pattern>[%d{timeStyle}] [%cn] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
</appender>
```
这里多了一个 SizeBasedTriggeringPolicy 触发机制，但 log 文件大于 10MB 的时候，就开始执行 rolling。
minIndex 和 maxIndex 表示保存日志数量，但大于 maxIndex 的时候，会开始删除旧的日志。

> 必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为
> mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz
> 或者 没有log%i.log.zip


### AsyncAppender 异步输出
这里就不写了，可以[参考][4]文章。

## 总结
由于没有深入去了解 logback ,许多内容都是网上摘来的，写文章的时候也很费劲。思路不清晰。


  [1]: https://logback.qos.ch/images/logos/lblogo.jpg
  [2]: /images/example.png
  [3]: https://logback.qos.ch/manual/appenders.html
  [4]: http://blog.csdn.net/ChenJie2000/article/details/8902727
