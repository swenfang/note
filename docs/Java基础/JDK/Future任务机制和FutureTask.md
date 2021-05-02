---
tags:
	- 多线程
categories: 多线程
title: Future 任务机制和 FutureTask
---
# Future 任务机制和 FutureTask

## 前言
今天在完成功能的时候，使用到 Future 在这里记录一下，自己所了解的到知识，希望可以帮到需要的朋友。
<!-- more -->

## Future 类

Future 类就是对于具体的 Runnable 或者 Callable 任务的执行结果进行取消、查询是否已经完成、获取结果。必要时可以通过 get 方法获取执行结果，该方法会阻塞直到任务返回结果。Future 位于 java.util.concurren 包下，它也是一个接口，如下：

```java
public interface Future<V> {
    /*
     * 用来取消任务,如果取消任务成功，则返回 true，失败则返回 false 。参数 mayInterrypIfRunning      
     * 表示是否允许取消正在执行却没有执行完毕的任务，如果设置 true ，则表示可以取消正在执行中的任务 。
     * 如果任务已经完成，则无论 mayInterruptIfRunning 为 ture 还是 false ，都返回 false，即如果
     * 取消已经完成的任务会返回 false ；如果任务正在执行，若 mayInterrupIfRunning 设置为 true 则
     * 返回 true ，设置为 false 则返回 false；如果任务还没有执行，都返回false。
     */
    boolean cancel(boolean mayInterruptIfRunning);
    /*
     * 表示任务是否已经完成，若任务完成则返回 true
     */
    boolean isDone();
    /*
     * 用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回。
     */
    V get() throws InterruptedException,ExecutionException;
    /*
     * 用来获取执行结果，如果在指定时间内，还没有获取到返回结果，就直接返回 null
     */
    V get(long timeout,TimeUnit unit) 
        throws InterruptedException,ExecutionException,TimeoutException;
    	
}
```

也就是说 Future 提供了三种功能：

- 判断任务是否完成。
- 能够中断任务。
- 能够获取任务执行结果。

## FutureTask 类

因为 Future 只是一个接口，所以无法直接用来创建对象使用的，因此就有了 FutureTask 。

FutureTask 目前是 Future 接口的一个唯一实现类：

![](http://blogimg.nos-eastchina1.126.net/shenwf20190128111337-690350.jpg)

 FutureTask 类

```java
public class FutureTask<V> implements RunnableFuture<V> {
    ...
}
```

RunableFuture 类

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}

```

可以看出 RunnableFuture 继承了 Runnable 和 Future 接口，而 FutureTask  实现了 RunnableFuture 接口。所以 FutureTask 既可以作为 Runnable 被线程执行，又可以作为 Future 得到 Callable 的返回值。

FutureTask 提供了2个构造器：

```java
// 创建一个 FutureTask ，一旦运行就执行给定的 Callable
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
// 创建一个 FutureTask ，一旦运行就执行给定的 Runnable ，并安排成功时 get 返回给定的结果。
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

## 使用场景 

在实际工作中，可能需要统计各种类型的报表呈现结果，可能一个大的报表需要依赖很多很小的模块的运算结果，一个线程做可能比较慢，就可拆分成 N 多个小线程，然后将结果合并起来作为大的报表呈现结果。Fork/Join 就是基于 Future 实现的



