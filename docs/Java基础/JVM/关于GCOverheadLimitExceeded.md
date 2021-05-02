---
tags:
	- Java JVM
categories: Java JVM
title: 关于GC overhead limit exceeded
mathjax: true
---
# 关于GC overhead limit exceeded

> 我遇到这样的问题，本地部署时抛出异常java.lang.OutOfMemoryError：GC overhead limit exceeded导致服务起不来，查看日志发现加载了太多资源到内存，本地的性能也不好，gc时间消耗的较多。<!--more-->解决这种问题两种方法是，增加参数，-XX:-UseGCOverheadLimit，关闭这个特性，同时增加heap大小，-Xmx1024m。坑填了，but why？
>
> OOM大家都知道，就是JVM内存溢出了，那GC overhead limit exceed呢？
>
> GC overhead limt exceed检查是Hotspot VM 1.6定义的一个策略，通过统计GC时间来预测是否要OOM了，提前抛出异常，防止OOM发生。Sun 官方对此的定义是：“并行/并发回收器在GC回收时间过长时会抛出OutOfMemroyError。过长的定义是，超过98%的时间用来做GC并且回收了不到2%的堆内存。用来避免内存过小造成应用不能正常工作。“
>
> 听起来没啥用...预测OOM有啥用？起初开来这玩意只能用来Catch住释放内存资源，避免应用挂掉。后来发现一般情况下这个策略不能拯救你的应用，但是可以在应用挂掉之前做最后的挣扎，比如数据保存或者保存现场（Heap Dump）。
>
> 而且有些时候这个策略还会带来问题，比如加载某个大的内存数据时频繁OOM。
>
> 假如你也生产环境中遇到了这个问题，在不知道原因时不要简单的猜测和规避。可以通过**-verbose:gc -XX:+PrintGCDetails**看下到底什么原因造成了异常。**通常原因都是因为old区占用过多导致频繁Full GC，最终导致GC overhead limit exceed**。如果gc log不够可以借助于JProfile等工具查看内存的占用，old区是否有内存泄露。分析内存泄露还有一个方法**-XX:+HeapDumpOnOutOfMemoryError**，这样OOM时会自动做Heap Dump，可以拿MAT来排查了。还要留意young区，如果有过多短暂对象分配，可能也会抛这个异常。

日志的信息不难理解，就是每次gc时打条日志，记录GC的类型，前后大小和时间。举个例子。

33.125: [GC [DefNew: 16000K->16000K(16192K), 0.0000574 secs][Tenured: 2973K->2704K(16384K), 0.1012650 secs] 18973K->2704K(32576K), 0.1015066 secs]

100.667:[Full GC [Tenured: 0K->210K(10240K), 0.0149142 secs] 4603K->210K(19456K), [Perm : 2999K->2999K(21248K)], 0.0150007 secs]

>GC和Full GC代表gc的停顿类型，Full GC代表stop-the-world。箭头两边是gc前后的区空间大小，分别是young区、tenured区和perm区，括号里是该区的总大小。冒号前面是gc发生的时间，单位是秒，从jvm启动开始计算。DefNew代表Serial收集器，为Default New Generation的缩写，类似的还有PSYoungGen，代表Parallel Scavenge收集器。这样可以通过分析日志找到导致GC overhead limit exceeded的原因，通过调节相应的参数解决问题。
>
>文中涉及到的名词解释，
>
>Eden Space：堆内存池，大多数对象在这里分配内存空间。
>
>Survivor Space：堆内存池，存储在Eden Space的gc中存活下来的对象。
>
>Tenured Generation：堆内存池，存储Survivor Space中存活过几次gc的对象。
>
>Permanent Generation：非堆空间，存储的是class和method对象。
>
>Code Cache：非堆空间，JVM用来存储编译和存储native code。

最后附上GC overhead limit exceed HotSpot的实现：

```java
bool print_gc_overhead_limit_would_be_exceeded = false;
if (is_full_gc) {
  if (gc_cost() > gc_cost_limit &&
    free_in_old_gen < (size_t) mem_free_old_limit &&
    free_in_eden < (size_t) mem_free_eden_limit) {
    inc_gc_overhead_limit_count();
    if (UseGCOverheadLimit) {
      if (gc_overhead_limit_count() >=
          AdaptiveSizePolicyGCTimeLimitThreshold){
        // All conditions have been met for throwing an out-of-memory
        set_gc_overhead_limit_exceeded(true);
        // Avoid consecutive OOM due to the gc time limit by resetting
        // the counter.
        reset_gc_overhead_limit_count();
      } else {
        // The required consecutive collections which exceed the
        // GC time limit may or may not have been reached. We
        // are approaching that condition and so as not to
        // throw an out-of-memory before all SoftRef's have been
        // cleared, set _should_clear_all_soft_refs in CollectorPolicy.
        // The clearing will be done on the next GC.
        bool near_limit = gc_overhead_limit_near();
        if (near_limit) {
          collector_policy->set_should_clear_all_soft_refs(true);
          if (PrintGCDetails && Verbose) {
            gclog_or_tty->print_cr("  Nearing GC overhead limit, "
              "will be clearing all SoftReference");
          }
        }
      }
    }
    // Set this even when the overhead limit will not
    // cause an out-of-memory.  Diagnostic message indicating
    // that the overhead limit is being exceeded is sometimes
    // printed.
    print_gc_overhead_limit_would_be_exceeded = true;
 
  } else {
    // Did not exceed overhead limits
    reset_gc_overhead_limit_count();
  }
}
```



