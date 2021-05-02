---
tags:
	- 基础面试题
categories: 基础面试题
title: Jdk1.7与 jdk1.8的区别和最新的特征
---

## jdk7的新特性

jdk7的新特性方面主要有下面几方面的增强

<!--more-->

### 二进制变量的表示

>二进制变量的表示,支持将整数类型用二进制来表示，用0b开头。
>
>所有整数int、short、long、byte都可以用二进制表示：
>
>```java
>byte aByte = (byte) 0b00100001;
>```

### Switch语句支持String类型

### Try-with-resource语句

参考博客：[try-with-resources语句](http://www.cnblogs.com/aspirant/p/8621848.html) 

>try-with-resources语句是一种声明了一种或多种资源的try语句。资源是指在程序用完了之后必须要关闭的对象。try-with-resources语句保证了每个声明了的资源在语句结束的时候都会被关闭。任何实现了java.lang.AutoCloseable接口的对象，和实现了java.io.Closeable接口的对象，都可以当做资源使用。

### Catch多个异常

>在Java 7中，catch代码块得到了升级，用以在单个catch块中处理多个异常。如果你要捕获多个异常并且它们包含相似的代码，使用这一特性将会减少代码重复度。下面用一个例子来理解。
>
>```java
>catch(IOException | SQLException | Exception ex){
>     logger.error(ex);
>     throw new MyException(ex.getMessage());
>}
>```

### 数字类型的下划线表示 

>数字类型的下划线表示 更友好的表示方式，不过要注意下划线添加的一些标准。
>
>字面常量数字里加下划线的规则：下划线只能在数字之间，在数字的开始或结束一定不能使用下划线。
>
>```java
>public class UsingUnderscoreInNumericLiterals {
>    public static void main(String[] args) {
>        int int_num = 1_00_00_000;
>        System.out.println("int num:" + int_num);
>
>        long long_num = 1_00_00_000;
>        System.out.println("long num:" + long_num);
>
>        float float_num = 2.10_001F;
>        System.out.println("float num:" + float_num);
>
>        double double_num = 2.10_12_001;
>        System.out.println("double num:" + double_num);
>    }
>}
>```

### 泛型实例的创建简化

>泛型实例的创建可以通过类型推断来简化 可以去掉后面new部分的泛型类型，只用<>就可以了。

### 并发工具增强

>并发工具增强： fork-join框架最大的增强，充分利用多核特性，将大问题分解成各个子问题，由多个cpu可以同时解决多个子问题，最后合并结果，继承RecursiveTask，实现compute方法，然后调用fork计算，最后用join合并结果。	

参考自己写的例子：[Java7 Fork-Join 框架：任务切分，并行处理](http://www.cnblogs.com/aspirant/p/8622584.html)

## JDK1.8的新特性

![](http://blogimg.nos-eastchina1.126.net/shenwf20190506104625-683894.jpg)

### 接口的默认和静态方法

>Java 8允许我们给接口添加一个非抽象的方法实现，只需要使用 default关键字即可，这个特征又叫做扩展方法。
>
>```java
>public interface JDK8Interface {
>    // static修饰符定义静态方法  
>    static void staticMethod() {  
>        System.out.println("接口中的静态方法");  
>    }  
>    // default修饰符定义默认方法  
>    default void defaultMethod() {  
>        System.out.println("接口中的默认方法");  
>    }  
>}  
>```

### Lambda 表达式

>Lambda 表达式：(例如： (x, y) -> { return x + y; } ;λ表达式有三部分组成：参数列表，箭头（->），以及一个表达式或语句块。)
>
>参考博客: [lambda表达式详解](http://www.cnblogs.com/LvLoveYuForever/p/6697982.html); 
>
>[函数式接口和Lambda表达式深入理解](https://www.jianshu.com/p/40f833bf2c48)
>
>在Java 8 中你就没必要使用这种传统的匿名对象的方式了，Java 8提供了更简洁的语法，lambda表达式：
>
>```java
>Collections.sort(names, (String a, String b) -> {
>      return b.compareTo(a);
>});
>```

### 方法与构造函数引用 

>Java 8 允许你使用 : 关键字来传递方法或者构造函数引用，上面的代码展示了如何引用一个静态方法，我们也可以引用一个对象的方法：
>
>```java
>converter = something::startsWith;
>String converted = converter.convert("Java");
>System.out.println(converted);
>```

### 函数式接口

>所谓的函数式接口，当然首先是一个接口，然后就是在这个接口里面只能有一个抽象方法。

###  Annotation 注解

>Annotation 注解：支持多重注解：
>
>很多时候一个注解需要在某一位置多次使用。
>
>```java
>YourAnnotation
>@YourAnnotation
>public void test(){
>    //TODO
>}
>```

### 新的日期时间 API

>Java 8新的Date-Time API (JSR 310)受Joda-Time的影响，提供了新的java.time包，可以用来替代
>
>java.util.Date和java.util.Calendar。一般会用到Clock、LocaleDate、LocalTime、LocaleDateTime、ZonedDateTime、Duration这些类，对于时间日期的改进还是非常不错的。

### Base64编码

>Base64编码是一种常见的字符编码，可以用来作为电子邮件或Web Service附件的传输编码。
>
>在Java 8中，Base64编码成为了Java类库的标准。Base64类同时还提供了对URL、MIME友好的编码器与解码器。

### JavaScript引擎Nashorn

>Nashorn允许在JVM上开发运行JavaScript应用，允许Java与JavaScript相互调用。

### Stream的使用

>Stream API是把真正的函数式编程风格引入到Java中。其实简单来说可以把Stream理解为MapReduce，当然Google的MapReduce的灵感也是来自函数式编程。她其实是一连串支持连续、并行聚集操作的元素。从语法上看，也很像linux的管道、或者链式编程，代码写起来简洁明了，非常酷帅！

### Optional

>Java 8引入Optional类来防止空指针异常，Optional类最先是由Google的Guava项目引入的。Optional类实际上是个容器：它可以保存类型T的值，或者保存null。使用Optional类我们就不用显式进行空指针检查了。

### 扩展注解的支持

>Java 8扩展了注解的上下文，几乎可以为任何东西添加注解，包括局部变量、泛型类、父类与接口的实现，连方法的异常也能添加注解。

### 并行（parallel）数组

>支持对数组进行并行处理，主要是parallelSort()方法，它可以在多核机器上极大提高数组排序的速度。

### 编译器优化

>Java 8将方法的参数名加入了字节码中，这样在运行时通过反射就能获取到参数名，只需要在编译时使用-parameters参数。
>
>新的Java1.8对IO做了升级：
>
>关于IO/NIO 新IO的对比，请参考：[Java NIO：IO与NIO的区别 -阿里面试题](http://www.cnblogs.com/aspirant/p/8630283.html)
>
>还对CurrentHashMap做了升级，请参考：[ConcurrentHashMap原理分析（1.7与1.8）](http://www.cnblogs.com/aspirant/p/8623864.html)