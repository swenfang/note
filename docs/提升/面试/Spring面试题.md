---
tags:
	- 基础面试题
categories: 基础面试题
title: Spring 面试题
---

## Spring IOC、AOP的理解、实现的原理，以及优点

> Spring的IoC容器是Spring的核心，Spring AOP是spring框架的重要组成部分
>
> **IOC**
>
> - **我的理解**
>   - 正常的情况下，比如有一个类，在类里面有方法（不是静态的方法），调用类里面的方法，创建类的对象，使用对象调用方法，创建类对象的过程，需要new出来对象
>   - 通过控制反转，把对象的创建不是通过new方式实现，而是交给Spring配置创建类对象
>   - IOC的意思是控件反转也就是由容器控制程序之间的关系，这也是spring的优点所在，把控件权交给了外部容器，之前的写法，由程序代码直接操控，而现在控制权由应用代码中转到了外部容器，控制权的转移是所谓反转。换句话说之前用new的方式获取对象，现在由spring给你至于怎么给你就是di了。
> - **Spring IOC实现原理**
>   - 创建xml配置文件，配置要创建的对象类
>   - 通过反射创建实例
>   - 获取需要注入的接口实现类并将其赋值给该接口
> - **优点**
>   - 解耦合，开发更方便组织分工
>   - 高层不依赖于底层（依赖倒置）
>   - 是应用更容易测试
>   - 因为把对象生成放在了XML里定义，所以当我们需要换一个实现子类将会变成很简单（一般这样的对象都是现实于某种接口的），只要修改XML就可以了，这样我们甚至可以实现对象的热插拨
>
> **AOP**(面向切面编程,主要的作用是不需要修改源代码的基础扩展功能)
>
> - **我的理解**
>   - AOP（Aspect Oriented Programming ）称为面向切面编程，扩展功能不是修改源代码实现，在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等待，Struts2的拦截器设计就是基于AOP的思想，是个比较经典的例子。
>   - 面向切面编程（aop）是对面向对象编程（oop）的补充
>   - 面向切面编程提供声明式事务管理
>   - AOP就是典型的代理模式的体现
> - **Spring AOP实现原理**
>   - 动态代理（利用**反射和动态编译**将代理模式变成动态的）
>   - JDK的动态代理
>     - JDK内置的Proxy动态代理可以在运行时动态生成字节码，而没必要针对每个类编写代理类
>     - JDKProxy返回动态代理类，是目标类所实现接口的另一个实现版本，它实现了对目标类的代理（如同UserDAOProxy与UserDAOImp的关系）
>   - cglib动态代理
>     - CGLibProxy返回的动态代理类，则是目标代理类的一个子类（代理类扩展了UserDaoImpl类）
>     - cglib继承被代理的类，重写方法，织入通知，动态生成字节码并运行
> - **优点**
>   - 各个步骤之间的良好隔离性
>   - 源代码无关性
>   - 松耦合
>   - 易扩展
>   - 代码复用

## 项目中Spring AOP用在什么地方，为什么这么用，切点，织入，通知用自己的话描述一下

>- Joinpoint（连接点）（重要）
>  - 类里面可以被增强的方法，这些方法称为连接点
>- Pointcut（切入点）（重要）
>  - 所谓切入点是指我们要对哪些Joinpoint进行拦截的定义
>- Advice（通知/增强）（重要）
>  - 所谓通知是指拦截到Joinpoint之后所要做的事情就是通知.通知分为前置通知，后置通知，异常通知，最终通知，环绕通知（切面要完成的功能）
>- Aspect（切面）
>  - 是切入点和通知（引介）的结合
>- Introduction（引介）
>  - 引介是一种特殊的通知在不修改类代码的前提下， Introduction可以在运行期为类动态地添加一些方法或Field.
>- Target（目标对象）
>  - 代理的目标对象（要增强的类）
>- Weaving（织入）
>  - 是把增强应用到目标的过程，把advice 应用到 target的过程
>- Proxy（代理）
>  - 一个类被AOP织入增强后，就产生一个结果代理类
>
>AOP（Aspect Oriented Programming ）称为面向切面编程，扩展功能不是修改源代码实现，在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等待，Struts2的拦截器设计就是基于AOP的思想，是个比较经典的例子。

## AOP动态代理2种实现原理，他们的区别是什么？

> 动态代理与cglib实现的区别
>
> - JDK动态代理只能对实现了接口的类生成代理，而不能针对类
> - cglib是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法因为是继承，所以该类或方法最好不要声明成final
> - JDK代理是不需要以来第三方的库，只要JDK环境就可以进行代理
> - cglib必须依赖于cglib的类库，但是它需要类来实现任何接口代理的是指定的类生成一个子类，覆盖其中的方法，是一种继承

## Spring事物的七种事物传播属性行为及五种隔离级别

> Spring七个事物传播属性：
>
> 1. PROPAGATION_REQUIRED -- 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
> 2. PROPAGATION_SUPPORTS -- 支持当前事务，如果当前没有事务，就以非事务方式执行。
> 3. PROPAGATION_MANDATORY -- 支持当前事务，如果当前没有事务，就抛出异常。
> 4. PROPAGATION_REQUIRES_NEW -- 新建事务，如果当前存在事务，把当前事务挂起。
> 5. PROPAGATION_NOT_SUPPORTED -- 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
> 6. PROPAGATION_NEVER -- 以非事务方式执行，如果当前存在事务，则抛出异常。
> 7. PROPAGATION_NESTED -- 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。
>
> 五个隔离级别：
>
> 1. ISOLATION_DEFAULT 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别.另外四个与JDBC的隔离级别相对应；
> 2. ISOLATION_READ_UNCOMMITTED 这是事务最低的隔离级别，它充许别外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。**(读未提交)**
> 3. ISOLATION_READ_COMMITTED 保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。**(读已提交)**
> 4. ISOLATION_REPEATABLE_READ 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。它除了保证 一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生 **(不可重复读)**。
> 5. ISOLATION_SERIALIZABLE 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻像读。**(串行化)**

## Spring 工作流程描述

>1. 用户向服务器发送请求，请求被Spring 前端控制Servelt DispatcherServlet捕获；
>2. DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI）。然后根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回；
>3. DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler(...)方法）
>4. 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)。 在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：ttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息；数据转换：对请求消息进行数据转换。如String转换成Integer、Double等；数据根式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等；数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中；
>5. Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象；
>6. 根据返回的ModelAndView，选择一个适合的ViewResolver（必须是已经注册到Spring容器中的ViewResolver)返回给DispatcherServlet ；
>7. ViewResolver 结合Model和View，来渲染视图
>8. 将渲染结果返回给客户端。

## 为什么Spring只使用一个Servlet(DispatcherServlet)来处理所有请求？

>Spring工作流程描述
>    为什么Spring只使用一个Servlet(DispatcherServlet)来处理所有请求？
>提供一个集中的请求处理机制，所有的请求都将由一个单一的处理程序处理。该处理程序可以做认证/授权/记录日志，或者跟踪请求，然后把请求传给相应的处理程序。以下是这种设计模式的实体。
>    Spring为什么要结合使用HandlerMapping以及HandlerAdapter来处理Handler?
>    符合面向对象中的单一职责原则，代码架构清晰，便于维护，最重要的是代码可复用性高。如HandlerAdapter可能会被用于处理多种Handler。

## Spring有哪些优点

> 1.轻量级：Spring在大小和透明性方面绝对属于轻量级的，基础版本的Spring框架大约只有2MB。
> 2.控制反转(IOC)：Spring使用控制反转技术实现了松耦合。依赖被注入到对象，而不是创建或寻找依赖对象。
> 3.面向切面编程(AOP)： Spring支持面向切面编程，同时把应用的业务逻辑与系统的服务分离开来。
> 4.容器：Spring包含并管理应用程序对象的配置及生命周期。
> 5.MVC框架：Spring的web框架是一个设计优良的web MVC框架，很好的取代了一些web框架。
> 6.事务管理：Spring对下至本地业务上至全局业务(JAT)提供了统一的事务管理接口。
> 7.异常处理：Spring提供一个方便的API将特定技术的异常(由JDBC, Hibernate, 或JDO抛出)转化为一致的、Unchecked异常。

## 什么是控制反转(IoC)？什么是依赖注入？

> IoC，是 Inversion of Control 的缩写，即控制反转。
>
> - 上层模块不应该依赖于下层模块，它们共同依赖于一个抽象
> - 抽象不能依赖于具体实现，具体实现依赖于抽象。
>
> 注：又称为依赖倒置原则。这是设计模式六大原则之一。
>
> DI，是 Dependency Injection 的缩写，即依赖注入。
>
> - 依赖注入是 IoC 的最常见形式。
> - 容器全权负责的组件的装配，它会把符合依赖关系的对象通过 JavaBean 属性或者构造函数传递给需要的对象。
>
> 依赖注入三种形式：1.构造器注入；2.setter 方法注入；3.接口注入

## Spring 中的 IoC

> BeanFactory 是 Spring IoC 容器的具体实现，用来包装和管理前面提到的各种 bean。BeanFactory 接口是 Spring IoC 容器的核心接口。IOC:把对象的创建、初始化、销毁交给 spring 来管理，而不是由开发者控制，实现控制反转。

## 什么是Spring的依赖注入

> 依赖注入，是IOC的一个方面，是个通常的概念，它有多种解释。这概念是说你不用创建对象，而只需要描述它如何被创建。你不在代码里直接组装你的组件和服务，但是要在配置文件里描述哪些组件需要哪些服务，之后一个容器（IOC容器）负责把他们组装起来。

## 有哪些不同类型的IOC（依赖注入）方式

>- **构造器依赖注入：**构造器依赖注入通过容器触发一个类的构造器来实现的，该类有一系列参数，每个参数代表一个对其他类的依赖。
>- **Setter方法注入：**Setter方法注入是容器通过调用无参构造器或无参static工厂 方法实例化bean之后，调用该bean的setter方法，即实现了基于setter的依赖注入。
>
>两种依赖方式都可以使用，构造器注入和Setter方法注入。最好的解决方案是用构造器参数实现强制依赖，setter方法实现可选依赖。

## IOC容器是什么其优点

>Spring IOC负责创建对象、管理对象(通过依赖注入)、整合对象、配置对象以及管理这些对象的生命周期。
>优点:
>IOC或依赖注入减少了应用程序的代码量。它使得应用程序的测试很简单，因为在单元测试中不再需要单例或JNDI查找机制。简单的实现以及较少的干扰机制使得松耦合得以实现。IOC容器支持勤性单例及延迟加载服务。

## BeanFactory和ApplicationContext有什么区别

> BeanFactory 包含了种 bean 的定义，以便在接收到客户端请求时将对应的 bean 实例化。BeanFactory 还能在实例化对象的时生成协作类之间的关系。BeanFactory 还包含了 bean 生命周期的控制，调用客户端的初始化方法（initialization methods）和销毁方法（destruction methods）。
>
> ApplicationContext 扩展了 BeanFactory：1.提供了支持国际化的文本消息2.统一的资源文件读取方式3.已在监听器中注册的 bean 的事件
>
> 三种较常见的 ApplicationContext 实现：
>
> - **FileSystemXmlApplicationContext ：**此容器从一个XML文件中加载beans的定义，XML Bean 配置文件的全路径名必须提供给它的构造函数。
> - **ClassPathXmlApplicationContext：**此容器也从一个XML文件中加载beans的定义，这里，你需要正确设置classpath因为这个容器将在classpath里找bean配置。
> - **WebXmlApplicationContext：**此容器加载一个XML文件，此文件定义了一个WEB应用的所有bean。
>
> 如果使用ApplicationContext，如果配置的bean是singleton，那么不管你有没有或想不想用它，它都会被实例化。好处是可以预先加载，坏处是浪费内存。BeanFactory，当使用BeanFactory实例化对象时，配置的bean不会马上被实例化，而是等到你使用该bean的时候（getBean）才会被实例化。好处是节约内存，坏处是速度比较慢。多用于移动设备的开发。

## Spring有几种配置方式？

>将Spring配置到应用开发中有以下三种方式：
>
>1. 基于XML的配置
>2. 基于注解的配置
>3. 基于Java的配置

## 请解释Spring Bean的生命周期？

对于普通的Java对象，当new的时候创建对象，当它没有任何引用的时候被垃圾回收机制回收。而由Spring IoC容器托管的对象，它们的生命周期完全由容器控制。Spring中每个Bean的生命周期如下：

![](http://blogimg.nos-eastchina1.126.net/shenwf20190420095615-102292.jpg)

1. **实例化Bean：**对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。 容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入。 实例化对象被包装在BeanWrapper对象中，BeanWrapper提供了设置对象属性的接口，从而避免了使用反射机制设置属性。
2. **设置对象属性（依赖注入）：**实例化后的对象被封装在BeanWrapper对象中，并且此时对象仍然是一个原生的状态，并没有进行依赖注入。 紧接着，Spring根据BeanDefinition中的信息进行依赖注入。并且通过BeanWrapper提供的设置属性的接口完成依赖注入。
3. **注入Aware接口：**紧接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean。
4. **BeanPostProcessor：**当经过上述几个步骤后，bean对象已经被正确构造，但如果你想要对象被使用前再进行一些自定义的处理，就可以通过BeanPostProcessor接口实现。 
   该接口提供了两个函数：
   - postProcessBeforeInitialzation( Object bean, String beanName ) 当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 这个函数会先于InitialzationBean执行，因此称为前置处理。 所有Aware接口的注入就是在这一步完成的。
   - postProcessAfterInitialzation( Object bean, String beanName ) 当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 这个函数会在InitialzationBean完成后执行，因此称为后置处理。
5. **InitializingBean与init-method：**当BeanPostProcessor的前置处理完成后就会进入本阶段。 InitializingBean接口只有一个函数：afterPropertiesSet()这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行afterPropertiesSet函数。当然，Spring为了降低对客户代码的侵入性，给bean的配置提供了init-method属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。init-method本质上仍然使用了InitializingBean接口。
6. **DisposableBean和destroy-method：**和init-method一样，通过给destroy-method指定函数，就可以在bean销毁前执行指定的逻辑。

##  解释Spring支持的几种bean的作用域

> Spring框架支持以下五种bean的作用域：
>
> - **singleton :** bean在每个Spring ioc 容器中只有一个实例。
> - **prototype**：一个bean的定义可以有多个实例。
> - **request**：每次http请求都会创建一个bean，该作用域仅在基于web的Spring ApplicationContext情形下有效。
> - **session**：在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
> - **global-session**：在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
>
> 缺省的Spring bean 的作用域是Singleton.

## Spring框架中的单例Beans是线程安全的么？

>Spring框架中的单例bean不是线程安全的。Spring框架并没有对单例bean进行任何多线程的封装处理。关于单例bean的线程安全和并发问题需要开发者自行去搞定。最浅显的解决办法就是将多态bean的作用域由**“singleton**”变更为“**prototype**”。

## 请举例说明如

## 何在Spring中注入一个Java Collection？

> Spring提供以下几种集合的配置元素：
>
> - **<list>** :   该标签用来装配可重复的list值。
> - **<set>** :    该标签用来装配没有重复的set值。
> - **<map>**:   该标签可用来注入键和值可以为任何类型的键值对。
> - **<props>** : 该标签支持注入键和值都是字符串类型的键值对。

## 什么是bean装配

> 装配，或bean 装配是指在Spring 容器中把bean组装到一起，前提是容器需要知道bean的依赖关系，如何通过依赖注入来把它们装配到一起。

## 什么是bean的自动装配

> Spring 容器能够自动装配相互合作的bean，这意味着容器不需要<constructor-arg>和<property>配置，能通过Bean工厂自动处理bean之间的协作。

## 解释不同方式的自动装配

> 有五种自动装配的方式，可以用来指导Spring容器用自动装配方式来进行依赖注入。
>
> - **no**：默认的方式是不进行自动装配，通过显式设置ref 属性来进行装配。
> - **byName：**通过参数名 自动装配，Spring容器在配置文件中发现bean的autowire属性被设置成byname，之后容器试图匹配、装配和该bean的属性具有相同名字的bean。
> - **byType:：**通过参数类型自动装配，Spring容器在配置文件中发现bean的autowire属性被设置成byType，之后容器试图匹配、装配和该bean的属性具有相同类型的bean。如果有多个bean符合条件，则抛出错误。
> - **constructor：这个方式类似于**byType， 但是要提供给构造器参数，如果没有确定的带参数的构造器参数类型，将会抛出异常。
> - **autodetect：**首先尝试使用constructor来自动装配，如果无法工作，则使用byType方式。

## 自动装配有哪些局限性

> 自动装配的局限性是：
>
> - **重写**： 你仍需用 <constructor-arg>和 <property> 配置来定义依赖，意味着总要重写自动装配。
> - **基本数据类型**：你不能自动装配简单的属性，如基本数据类型，String字符串，和类。
> - **模糊特性：**自动装配不如显式装配精确，如果有可能，建议使用显式装配。

## 什么是基于Java的Spring注解配置? 给一些注解的例子

> 基于Java的配置，允许你在少量的Java注解的帮助下，进行你的大部分Spring配置而非通过XML文件。以@Configuration 注解为例，它用来标记类可以当做一个bean的定义，被Spring IOC容器使用。另一个例子是@Bean注解，它表示此方法将要返回一个对象，作为一个bean注册进Spring应用上下文。

## 什么是基于注解的容器配置

> 相对于XML文件，注解型的配置依赖于通过字节码元数据装配组件，而非尖括号的声明。开发者通过在相应的类，方法或属性上使用注解的方式，直接组件类中进行配置，而不是使用xml表述bean的装配关系。

## @Required  注解

> 这个注解表明bean的属性必须在配置的时候设置，通过一个bean定义的显式的属性值或通过自动装配，若@Required注解的bean属性未被设置，容器将抛出BeanInitializationException。

## @Autowired 注解

> @Autowired 注解提供了更细粒度的控制，包括在何处以及如何完成自动装配。它的用法和@Required一样，修饰setter方法、构造器、属性或者具有任意名称和/或多个参数的PN方法。

## @Qualifier 注解

> 当有多个相同类型的bean却只有一个需要自动装配时，将@Qualifier 注解和@Autowire 注解结合使用以消除这种混淆，指定需要装配的确切的bean。

## 构造方法注入和设值注入有什么区别？

> 请注意以下明显的区别：
>
> 1. 在设值注入方法支持大部分的依赖注入，如果我们仅需要注入int、string和long型的变量，我们不要用设值的方法注入。对于基本类型，如果我们没有注入的话，可以为基本类型设置默认值。在构造方法注入不支持大部分的依赖注入，因为在调用构造方法中必须传入正确的构造参数，否则的话为报错。
> 2. 设值注入不会重写构造方法的值。如果我们对同一个变量同时使用了构造方法注入又使用了设置方法注入的话，那么构造方法将不能覆盖由设值方法注入的值。很明显，因为构造方法尽在对象被创建时调用。
> 3. 在使用设值注入时有可能还不能保证某种依赖是否已经被注入，也就是说这时对象的依赖关系有可能是不完整的。而在另一种情况下，构造器注入则不允许生成依赖关系不完整的对象。
> 4. 在设值注入时如果对象A和对象B互相依赖，在创建对象A时Spring会抛出sObjectCurrentlyInCreationException异常，因为在B对象被创建之前A对象是不能被创建的，反之亦然。所以Spring用设值注入的方法解决了循环依赖的问题，因对象的设值方法是在对象被创建之前被调用的。

## Spring框架中有哪些不同类型的事件？

> Spring 提供了以下5中标准的事件：
>
> 1. **上下文更新事件（ContextRefreshedEvent）**：该事件会在ApplicationContext被初始化或者更新时发布。也可以在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。
> 2. **上下文开始事件（ContextStartedEvent）**：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。
> 3. **上下文停止事件（ContextStoppedEvent）**：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。
> 4. **上下文关闭事件（ContextClosedEvent）**：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。
> 5. **请求处理事件（RequestHandledEvent）**：在Web应用中，当一个http请求（request）结束触发该事件。

## BeanFactory 和 ApplicationContext 有什么区别

> BeanFactory 可以理解为含有bean集合的工厂类。BeanFactory 包含了种bean的定义，以便在接收到客户端请求时将对应的bean实例化。
> BeanFactory还能在实例化对象的时生成协作类之间的关系。此举将bean自身与bean客户端的配置中解放出来。BeanFactory还包含了bean生命周期的控制，调用客户端的初始化方法（initializationmethods）和销毁方法（destruction methods）。
> 从表面上看，applicationcontext如同beanfactory一样具有bean定义、bean关联关系的设置，根据请求分发bean的功能。但applicationcontext在此基础上还提供了其他的功能。
> 提供了支持国际化的文本消息统一的资源文件读取方式已在监听器中注册的bean的事件

## Spring Bean 的生命周期

> Spring Bean的生命周期简单易懂。在一个bean实例被初始化时，需要执行一系列的初始化操作以达到可用的状态。同样的，当一个bean不在被调用时需要进行相关的析构操作，并从bean容器中移除。Spring bean factory 负责管理在spring容器中被创建的bean的生命周期。Bean的生命周期由两组回调（call back）方法组成。
> 初始化之后调用的回调方法。
> 销毁之前调用的回调方法。
> Spring框架提供了以下四种方式来管理bean的生命周期事件： InitializingBean和DisposableBean回调接口
> 针对特殊行为的其他Aware接口
> Bean配置文件中的Custom init()方法和destroy()方法
> @PostConstruct和@PreDestroy注解方式

## Spring IOC 如何实现

> Spring中的 org.springframework.beans包和org.springframework.context包构成了Spring框架IoC容器的基础。
> BeanFactory 接口提供了一个先进的配置机制，使得任何类型的对象的配置成为可能。ApplicationContex接口对BeanFactory（是一个子接口）进行了扩展，在BeanFactory的基础上添加了其他功能，比如与Spring的AOP更容易集成，也提供了处理messageresource的机制（用于国际化）、事件传播以及应用层的特别配置，比如针对Web应用的WebApplicationContext。
> org.springframework.beans.factory.BeanFactory是SpringIoC容器的具体实现，用来包装和管理前面提到的各种bean。BeanFactory接口是Spring IoC 容器的核心接口。

## 说说 Spring AOP

> 面向切面编程，在我们的应用中，经常需要做一些事情，但是这些事情与核心业务无关，比如，要记录所有update方法的执行时间时间，操作人等等信息，记录到日志，通过spring的AOP技术，就可以在不修改update的代码的情况下完成该需求。

## Spring AOP 实现原理

> Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理。JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。JDK动态代理的核心是InvocationHandler接口和Proxy类。 如果目标类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

## 动态代理（cglib 与 JDK）

> JDK 动态代理类和委托类需要都实现同一个接口。也就是说只有实现了某个接口的类可以使用Java动态代理机制。但是，事实上使用中并不是遇到的所有类都会给你实现一个接口。因此，对于没有实现接口的类，就不能使用该机制。而CGLIB则可以实现对类的动态代理。

## 有几种不同类型的自动代理

> BeanNameAutoProxyCreator
>
> DefaultAdvisorAutoProxyCreator
>
> Metadata autoproxying

## AOP与OOP的区别

> OOP面向对象编程，针对业务处理过程的实体及其属性和行为进行抽象封装，以获得更加清晰高效的逻辑单元划分。而AOP则是针对业务处理过程中的切面进行提取，它所面对的是处理过程的某个步骤或阶段，以获得逻辑过程的中各部分之间低耦合的隔离效果。这两种设计思想在目标上有着本质的差异。
>
> 举例：对于“雇员”这样一个业务实体进行封装，自然是OOP的任务，我们可以建立一个“Employee”类，并将“雇员”相关的属性和行为封装其中。而用AOP 设计思想对“雇员”进行封装则无从谈起。同样，对于“权限检查”这一动作片段进行划分，则是AOP的目标领域。OOP面向名次领域，AOP面向动词领域。总之AOP可以通过预编译方式和运行期动态代理实现在不修改源码的情况下，给程序动态同意添加功能的一项技术。

## Aspect 切面

> AOP核心就是切面，它将多个类的通用行为封装成可重用的模块，该模块含有一组API提供横切功能。比如，一个日志模块可以被称作日志的AOP切面。根据需求的不同，一个应用程序可以有若干切面。在Spring AOP中，切面通过带有@Aspect注解的类实现。

## Spring 事务实现方式

> 1、编码方式
> 所谓编程式事务指的是通过编码方式实现事务，即类似于JDBC编程实现事务管理。
> 2、声明式事务管理方式
> 声明式事务管理又有两种实现方式：基于xml配置文件的方式；另一个实在业务方法上进行@Transaction注解，将事务规则应用到业务逻辑中

## Spring 事务底层原理

> a、划分处理单元——IOC
> 由于spring解决的问题是对单个数据库进行局部事务处理的，具体的实现首相用spring中的IOC划分了事务处理单元。并且将对事务的各种配置放到了ioc容器中（设置事务管理器，设置事务的传播特性及隔离机制）。
> b、AOP拦截需要进行事务处理的类
> Spring事务处理模块是通过AOP功能来实现声明式事务处理的，具体操作（比如事务实行的配置和读取，事务对象的抽象），用TransactionProxyFactoryBean接口来使用AOP功能，生成proxy代理对象，通过TransactionInterceptor完成对代理方法的拦截，将事务处理的功能编织到拦截的方法中。读取ioc容器事务配置属性，转化为spring事务处理需要的内部数据结构（TransactionAttributeSourceAdvisor），转化为TransactionAttribute表示的数据对象。
> c、对事物处理实现（事务的生成、提交、回滚、挂起）
> spring委托给具体的事务处理器实现。实现了一个抽象和适配。适配的具体事务处理器：DataSource数据源支持、hibernate数据源事务处理支持、JDO数据源事务处理支持，JPA、JTA数据源事务处理支持。这些支持都是通过设计PlatformTransactionManager、AbstractPlatforTransaction一系列事务处理的支持。 为常用数据源支持提供了一系列的TransactionManager。
> d、结合
> PlatformTransactionManager实现了TransactionInterception接口，让其与TransactionProxyFactoryBean结合起来，形成一个Spring声明式事务处理的设计体系。

## spring的事务有几种它的隔离级别和传播行为

> 声明式事务和编程式事务
>
> 隔离级别：
>
> 1. DEFAULT(default)使用数据库默认的隔离级别
> 2. READ_UNCOMMITTED(read_uncommitted)会出现脏读，不可重复读和幻影读问题
> 3. READ_COMMITTED(read_committed)会出现重复读和幻影读
> 4. REPEATABLE_READ(repeatable_read)会出现幻影读
> 5. SERIALIZABLE(serialzable)最安全，但是代价最大，性能影响极其严重
>
> 传播行为：
>
> 1. REQUIRED(required)存在事务就融入该事务，不存在就创建事务
> 2. SUPPORTS(supports)存在事务就融入事务，不存在则不创建事务
> 3. MANDATORY(mandatory)存在事务则融入该事务，不存在，抛异常
> 4. REQUIRES_NEW(requirse_new)总是创建新事务
> 5. NOT_SUPPORTED(not_supported)存在事务则挂起，一直执行非事务操作
> 6. NEVER(never)总是执行非事务，如果当前存在事务则抛异常
> 7. NESTED(nested)嵌入式事务

## 如何自定义注解实现功能

> 创建自定义注解和创建一个接口相似，但是注解的interface关键字需要以@符号开头。
> 注解方法不能带有参数；
> 注解方法返回值类型限定为：基本类型、String、Enums、Annotation或者是这些类型的数组；
> 注解方法可以有默认值；
> 注解本身能够包含元注解，元注解被用来注解其它注解。

## Spring MVC 运行流程

> 1.spring mvc将所有的请求都提交给DispatcherServlet,它会委托应用系统的其他模块负责对请求 进行真正的处理工作。
> 2.DispatcherServlet查询一个或多个HandlerMapping,找到处理请求的Controller.
> 3.DispatcherServlet请请求提交到目标Controller
> 4.Controller进行业务逻辑处理后，会返回一个ModelAndView
> 5.Dispathcher查询一个或多个ViewResolver视图解析器,找到ModelAndView对象指定的视图对象
> 6.视图对象负责渲染返回给客户端。

## Spring MVC 启动流程

> 在 web.xml 文件中给SpringMVC的Servlet配置了load-on-startup,所以程序启动的时候会初始化 Spring MVC，在 HttpServletBean 中将配置的 contextConfigLocation属性设置到 Servlet 中，然后在FrameworkServlet 中创建了 WebApplicationContext,DispatcherServlet根据contextConfigLocation 配置的 classpath 下的 xml 文件初始化了Spring MVC 总的组件。

## Spring 的单例实现原理

> Spring 对 Bean 实例的创建是采用单例注册表的方式进行实现的，而这个注册表的缓存是 ConcurrentHashMap 对象。

## Spring 框架中用到了哪些设计模式

> 代理模式—在AOP和remoting中被用的比较多。
> 单例模式—在spring配置文件中定义的bean默认为单例模式。
> 模板方法—用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。
> 前端控制器—Spring提供了DispatcherServlet来对请求进行分发。
> 视图帮助(View Helper)—Spring提供了一系列的JSP标签，高效宏来辅助将分散的代码整合在视图里。
> 依赖注入—贯穿于BeanFactory / ApplicationContext接口的核心理念。
> 工厂模式—BeanFactory用来创建对象的实例。

## 动态代理与cglib实现的区别

> • JDK动态代理只能对实现了接口的类生成代理，而不能针对类.
> • CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法因为是继承，所以该类或方法最好不要声明成final。
> • JDK代理是不需要以来第三方的库，只要JDK环境就可以进行代理
> • CGLib 必须依赖于CGLib的类库，但是它需要类来实现任何接口代理的是指定的类生成一个子类，覆盖其中的方法，是一种继承.

## Spring MVC的工作原理Spring MVC的工作原理

> • 1 客户端的所有请求都交给前端控制器DispatcherServlet来处理，它会负责调用系统的其他模块来真正处理用户的请求。
> • 2 DispatcherServlet收到请求后，将根据请求的信息(包括URL、HTTP协议方法、请求头、请求参数、Cookie等)以及HandlerMapping的配置找到处理该请求的Handler(任何一个对象都可以作为请求的Handler)。
> • 3在这个地方Spring会通过HandlerAdapter对该处理进行封装。
> • 4 HandlerAdapter是一个适配器，它用统一的接口对各种Handler中的方法进行调用。
> • 5 Handler完成对用户请求的处理后，会返回一个ModelAndView对象给DispatcherServlet,ModelAndView顾名思义，包含了数据模型以及相应的视图的信息。
> • 6 ModelAndView的视图是逻辑视图，DispatcherServlet还要借助ViewResolver完成从逻辑视图到真实视图对象的解析工作。
> • 7 当得到真正的视图对象后，DispatcherServlet会利用视图对象对模型数据进行渲染。
> • 8 客户端得到响应，可能是一个普通的HTML页面，也可以是XML或JSON字符串，还可以是一张图片或者一个PDF文件。

## 你分析过SpringMVC的源码吗？

### 1. MVC使用

> 在研究源码之前，先来回顾以下springmvc 是如何配置的，这将能使我们更容易理解源码。

#### 1.1 web.xml

```xml
<servlet>
    <servlet-name>mvc-dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 配置springMVC需要加载的配置文件
        spring-dao.xml,spring-service.xml,spring-web.xml
        Mybatis - > spring -> springmvc
     -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring/spring-*.xml</param-value>
    </init-param>
</servlet>
<servlet-mapping>
    <servlet-name>mvc-dispatcher</servlet-name>
    <!-- 默认匹配所有的请求 -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

值的注意的是`contextConfigLocation`和`DispatcherServlet`(用此类来拦截请求)的引用和配置。

#### 1.2 spring-web.xml

```xml
<!-- 配置SpringMVC -->
<!-- 1.开启SpringMVC注解模式 -->
<!-- 简化配置： 
    (1)自动注册DefaultAnootationHandlerMapping,AnotationMethodHandlerAdapter 
    (2)提供一些列：数据绑定，数字和日期的format @NumberFormat, @DateTimeFormat, xml,json默认读写支持 
-->
<mvc:annotation-driven />

<!-- 2.静态资源默认servlet配置
    (1)加入对静态资源的处理：js,gif,png
    (2)允许使用"/"做整体映射
 -->
 <mvc:default-servlet-handler/>

 <!-- 3.配置jsp 显示ViewResolver -->
 <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
     <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
     <property name="prefix" value="/WEB-INF/jsp/" />
     <property name="suffix" value=".jsp" />
 </bean>

 <!-- 4.扫描web相关的bean -->
 <context:component-scan base-package="com.xxx.fantj.web" />
```

值的注意的是`InternalResourceViewResolver`，它会在`ModelAndView`返回的试图名前面加上`prefix`前缀，在后面加载`suffix`指定后缀。

### SpringMvc主支源码分析

![](http://blogimg.nos-eastchina1.126.net/shenwf20190421121846-805191.jpg)



上图流程总体来说可分为三大块：

1. `Map`的建立(并放入`WebApplicationContext`)
2. `HttpRequest`请求中Url的请求拦截处理(DispatchServlet处理)
3. 反射调用`Controller`中对应的处理方法，并返回视图

本文将围绕这三块进行分析。

#### 1. Map的建立

> 在容器初始化时会建立所有 url 和 Controller 的对应关系,保存到 Map中，那是如何保存的呢。

###### ApplicationObjectSupport #setApplicationContext方法

```java
// 初始化ApplicationContext
@Override
public void initApplicationContext() throws ApplicationContextException {
    super.initApplicationContext();
    detectHandlers();
}
```

###### AbstractDetectingUrlHandlerMapping #detectHandlers()方法：

```java
/**
 * 建立当前ApplicationContext 中的 所有Controller 和url 的对应关系
 */
protected void detectHandlers() throws BeansException {
    if (logger.isDebugEnabled()) {
        logger.debug("Looking for URL mappings in application context: " + getApplicationContext());
    }
    // 获取容器中的beanNames
    String[] beanNames = (this.detectHandlersInAncestorContexts ?
            BeanFactoryUtils.beanNamesForTypeIncludingAncestors(getApplicationContext(), Object.class) :
            getApplicationContext().getBeanNamesForType(Object.class));
    // 遍历 beanNames 并找到对应的 url
    // Take any bean name that we can determine URLs for.
    for (String beanName : beanNames) {
        // 获取bean上的url(class上的url + method 上的 url)
        String[] urls = determineUrlsForHandler(beanName);
        if (!ObjectUtils.isEmpty(urls)) {
            // URL paths found: Let's consider it a handler.
            // 保存url 和 beanName 的对应关系
            registerHandler(urls, beanName);
        }
        else {
            if (logger.isDebugEnabled()) {
                logger.debug("Rejected bean name '" + beanName + "': no URL paths identified");
            }
        }
    }
}
```

##### determineUrlsForHandler()方法：

> 该方法在不同的子类有不同的实现，我这里分析的是`DefaultAnnotationHandlerMapping`类的实现，该类主要负责处理`@RequestMapping`注解形式的声明。

```java
/**
 * 获取@RequestMaping注解中的url
 */
@Override
protected String[] determineUrlsForHandler(String beanName) {
    ApplicationContext context = getApplicationContext();
    Class<?> handlerType = context.getType(beanName);
    // 获取beanName 上的requestMapping
    RequestMapping mapping = context.findAnnotationOnBean(beanName, RequestMapping.class);
    if (mapping != null) {
        // 类上面有@RequestMapping 注解
        this.cachedMappings.put(handlerType, mapping);
        Set<String> urls = new LinkedHashSet<String>();
        // mapping.value()就是获取@RequestMapping注解的value值
        String[] typeLevelPatterns = mapping.value();
        if (typeLevelPatterns.length > 0) {
            // 获取Controller 方法上的@RequestMapping
            String[] methodLevelPatterns = determineUrlsForHandlerMethods(handlerType);
            for (String typeLevelPattern : typeLevelPatterns) {
                if (!typeLevelPattern.startsWith("/")) {
                    typeLevelPattern = "/" + typeLevelPattern;
                }
                for (String methodLevelPattern : methodLevelPatterns) {
                    // controller的映射url+方法映射的url
                    String combinedPattern = getPathMatcher().combine(typeLevelPattern, methodLevelPattern);
                    // 保存到set集合中
                    addUrlsForPath(urls, combinedPattern);
                }
                addUrlsForPath(urls, typeLevelPattern);
            }
            // 以数组形式返回controller上的所有url
            return StringUtils.toStringArray(urls);
        }
        else {
            // controller上的@RequestMapping映射url为空串,直接找方法的映射url
            return determineUrlsForHandlerMethods(handlerType);
        }
    }
    // controller上没@RequestMapping注解
    else if (AnnotationUtils.findAnnotation(handlerType, Controller.class) != null) {
        // 获取controller中方法上的映射url
        return determineUrlsForHandlerMethods(handlerType);
    }
    else {
        return null;
    }
}
```

更深的细节代码就比较简单了，有兴趣的可以继续深入。

到这里，Controller和Url的映射就装配完成，下来就分析请求的处理过程。

#### 2. url的请求处理

> 我们在xml中配置了`DispatcherServlet`为调度器，所以我们就来看它的代码，可以
> 从名字上看出它是个`Servlet`,那么它的核心方法就是`doService()`

##### DispatcherServlet #doService():

```java
/**
 * 将DispatcherServlet特定的请求属性和委托 公开给{@link #doDispatch}以进行实际调度。
 */
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    if (logger.isDebugEnabled()) {
        String requestUri = new UrlPathHelper().getRequestUri(request);
        logger.debug("DispatcherServlet with name '" + getServletName() + "' processing " + request.getMethod() +
                " request for [" + requestUri + "]");
    }

    //在包含request的情况下保留请求属性的快照，
    //能够在include之后恢复原始属性。
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        logger.debug("Taking snapshot of request attributes before include");
        attributesSnapshot = new HashMap<String, Object>();
        Enumeration attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith("org.springframework.web.servlet")) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    // 使得request对象能供 handler处理和view处理 使用
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    try {
        doDispatch(request, response);
    }
    finally {
        // 如果不为空，则还原原始属性快照。
        if (attributesSnapshot != null) {
            restoreAttributesAfterInclude(request, attributesSnapshot);
        }
    }
}
```

可以看到，它将请求拿到后，主要是给request设置了一些对象，以便于后续工作的处理(Handler处理和view处理)。比如`WebApplicationContext`，它里面就包含了我们在第一步完成的`controller`与`url`映射的信息。

##### DispatchServlet # doDispatch()

```java
/**
 * 控制请求转发
 */
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    int interceptorIndex = -1;

    try {

        ModelAndView mv;
        boolean errorView = false;

        try {
            // 1. 检查是否是上传文件
            processedRequest = checkMultipart(request);

            // 2. 获取handler处理器，返回的mappedHandler封装了handlers和interceptors
            mappedHandler = getHandler(processedRequest, false);
            if (mappedHandler == null || mappedHandler.getHandler() == null) {
                // 返回404
                noHandlerFound(processedRequest, response);
                return;
            }

            // 获取HandlerInterceptor的预处理方法
            HandlerInterceptor[] interceptors = mappedHandler.getInterceptors();
            if (interceptors != null) {
                for (int i = 0; i < interceptors.length; i++) {
                    HandlerInterceptor interceptor = interceptors[i];
                    if (!interceptor.preHandle(processedRequest, response, mappedHandler.getHandler())) {
                        triggerAfterCompletion(mappedHandler, interceptorIndex, processedRequest, response, null);
                        return;
                    }
                    interceptorIndex = i;
                }
            }

            // 3. 获取handler适配器 Adapter
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
            // 4. 实际的处理器处理并返回 ModelAndView 对象
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            // Do we need view name translation?
            if (mv != null && !mv.hasView()) {
                mv.setViewName(getDefaultViewName(request));
            }

            // HandlerInterceptor 后处理
            if (interceptors != null) {
                for (int i = interceptors.length - 1; i >= 0; i--) {
                    HandlerInterceptor interceptor = interceptors[i];
                    // 结束视图对象处理
                    interceptor.postHandle(processedRequest, response, mappedHandler.getHandler(), mv);
                }
            }
        }
        catch (ModelAndViewDefiningException ex) {
            logger.debug("ModelAndViewDefiningException encountered", ex);
            mv = ex.getModelAndView();
        }
        catch (Exception ex) {
            Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
            mv = processHandlerException(processedRequest, response, handler, ex);
            errorView = (mv != null);
        }

        if (mv != null && !mv.wasCleared()) {
            render(mv, processedRequest, response);
            if (errorView) {
                WebUtils.clearErrorRequestAttributes(request);
            }
        }
        else {
            if (logger.isDebugEnabled()) {
                logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +
                        "': assuming HandlerAdapter completed request handling");
            }
        }

        // 请求成功响应之后的方法
        triggerAfterCompletion(mappedHandler, interceptorIndex, processedRequest, response, null);
    }

    catch (Exception ex) {
        // Trigger after-completion for thrown exception.
        triggerAfterCompletion(mappedHandler, interceptorIndex, processedRequest, response, ex);
        throw ex;
    }
    catch (Error err) {
        ServletException ex = new NestedServletException("Handler processing failed", err);
        // Trigger after-completion for thrown exception.
        triggerAfterCompletion(mappedHandler, interceptorIndex, processedRequest, response, ex);
        throw ex;
    }

    finally {
        // Clean up any resources used by a multipart request.
        if (processedRequest != request) {
            cleanupMultipart(processedRequest);
        }
    }
}
```

该方法主要是

1. 通过request对象获取到`HandlerExecutionChain`，`HandlerExecutionChain`对象里面包含了拦截器interceptor和处理器handler。如果获取到的对象是空，则交给`noHandlerFound`返回404页面。
2. 拦截器预处理，如果执行成功则进行3
3. 获取handler适配器 Adapter
4. 实际的处理器处理并返回 ModelAndView 对象

> 下面是该方法中的一些核心细节：

`DispatchServlet #doDispatch # noHandlerFound`核心源码：

```
response.sendError(HttpServletResponse.SC_NOT_FOUND);
```

`DispatchServlet #doDispatch #getHandler`方法事实上调用的是`AbstractHandlerMapping #getHandler`方法,我贴出一个核心的代码：

```java
// 拿到处理对象
Object handler = getHandlerInternal(request);
...
String handlerName = (String) handler;
handler = getApplicationContext().getBean(handlerName);
...
// 返回HandlerExecutionChain对象
return getHandlerExecutionChain(handler, request);
```

可以看到，它先从request里获取handler对象，这就证明了之前`DispatchServlet #doService`为什么要吧`WebApplicationContext`放入request请求对象中。

最终返回一个`HandlerExecutionChain`对象.

#### 3. 反射调用处理请求的方法，返回结果视图

> 在上面的源码中，实际的处理器处理并返回 ModelAndView 对象调用的是`mv = ha.handle(processedRequest, response, mappedHandler.getHandler());`这个方法。该方法由`AnnotationMethodHandlerAdapter #handle() #invokeHandlerMethod()`方法实现.

##### `AnnotationMethodHandlerAdapter #handle() #invokeHandlerMethod()`

```java
/**
 * 获取处理请求的方法,执行并返回结果视图
 */
protected ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {

    // 1.获取方法解析器
    ServletHandlerMethodResolver methodResolver = getMethodResolver(handler);
    // 2.解析request中的url,获取处理request的方法
    Method handlerMethod = methodResolver.resolveHandlerMethod(request);
    // 3. 方法调用器
    ServletHandlerMethodInvoker methodInvoker = new ServletHandlerMethodInvoker(methodResolver);
    ServletWebRequest webRequest = new ServletWebRequest(request, response);
    ExtendedModelMap implicitModel = new BindingAwareModelMap();
    // 4.执行方法（获取方法的参数）
    Object result = methodInvoker.invokeHandlerMethod(handlerMethod, handler, webRequest, implicitModel);
    // 5. 封装成mv视图
    ModelAndView mav =
            methodInvoker.getModelAndView(handlerMethod, handler.getClass(), result, implicitModel, webRequest);
    methodInvoker.updateModelAttributes(handler, (mav != null ? mav.getModel() : null), implicitModel, webRequest);
    return mav;
}
```

这个方法有两个重要的地方，分别是`resolveHandlerMethod`和`invokeHandlerMethod`。

##### resolveHandlerMethod 方法

`methodResolver.resolveHandlerMethod(request)`:获取controller类和方法上的`@requestMapping value`,与request的url进行匹配,找到处理request的controller中的方法.最终拼接的具体实现是`org.springframework.util.AntPathMatcher#combine`方法。

##### invokeHandlerMethod方法

> 从名字就能看出来它是基于反射，那它做了什么呢。

解析该方法上的参数,并调用该方法。

```java
//上面全都是为解析方法上的参数做准备
...
// 解析该方法上的参数
Object[] args = resolveHandlerArguments(handlerMethodToInvoke, handler, webRequest, implicitModel);
// 真正执行解析调用的方法
return doInvokeMethod(handlerMethodToInvoke, handler, args);
```

###### invokeHandlerMethod方法#resolveHandlerArguments方法

> 代码有点长，我就简介下它做了什么事情吧。

- 如果这个方法的参数用的是注解，则解析注解拿到参数名，然后拿到request中的参数名，两者一致则进行赋值(详细代码在`HandlerMethodInvoker#resolveRequestParam`)，然后将封装好的对象放到args[]的数组中并返回。
- 如果这个方法的参数用的不是注解，则需要asm框架(底层是读取字节码)来帮助获取到参数名，然后拿到request中的参数名，两者一致则进行赋值，然后将封装好的对象放到args[]的数组中并返回。

###### invokeHandlerMethod方法#doInvokeMethod方法

```javascript
private Object doInvokeMethod(Method method, Object target, Object[] args) throws Exception {
    // 将一个方法设置为可调用，主要针对private方法
    ReflectionUtils.makeAccessible(method);
    try {
        // 反射调用
        return method.invoke(target, args);
    }
    catch (InvocationTargetException ex) {
        ReflectionUtils.rethrowException(ex.getTargetException());
    }
    throw new IllegalStateException("Should never get here");
}
```

到这里,就可以对request请求中url对应的controller的某个对应方法进行调用了。

### 总结：

> 看完后脑子一定很乱，有时间的话还是需要自己动手调试一下。本文只是串一下整体思路，所以功能性的源码没有全部分析。

其实理解这些才是最重要的。

1. 用户发送请求至前端控制器DispatcherServlet
2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。
3. 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
4. DispatcherServlet通过HandlerAdapter处理器适配器调用处理器
5. HandlerAdapter执行处理器(handler，也叫后端控制器)。
6. Controller执行完成返回ModelAndView
7. HandlerAdapter将handler执行结果ModelAndView返回给DispatcherServlet
8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器
9. ViewReslover解析后返回具体View对象
10. DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。
11. DispatcherServlet响应用户