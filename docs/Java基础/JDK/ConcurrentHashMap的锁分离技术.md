---
tags:
	- 基础数据类型
categories: ConcurrentHashMap
title: ConcurrentHashMap 的锁定分离技术
---
# ConcurrentHashMap的锁分离技术

![img](https://images2015.cnblogs.com/blog/756320/201603/756320-20160321173412058-376309614.png)

​    对比上图，HashTable实现锁的方式是锁整个hash表，而ConcurrentHashMap的实现方式是**锁桶（**简单理解就是将整个hash表想象成一大缸水，现在将这大缸里的水分到了几个水桶里，hashTable每次都锁定这个大缸，而ConcurrentHashMap则每次只锁定其中一个 桶**）。**

​    ConcurrentHashMap将hash表分为16个桶（默认值），诸如get,put,remove等常用操作只锁当前需要用到的桶。试想，原来 只能一个线程进入，现在却能同时16个写线程进入，并发性的提升是显而易见的。

![img](https://images2015.cnblogs.com/blog/756320/201603/756320-20160321174511354-2060663727.png)

​    值得一提的是当对ConcurrentHashMap进行remove操作时，并不是进行简单的节点删除操作，对比上图，当对ConcurrentHashMap的一个segment也就是一个桶中的节点进行remove后，例如删除节点C，C节点实际并没有被销毁，而是将C节点前面的反转并拷贝到新的链表中，C节点后面的不需要被克隆。这样来保持并发的读线程不受并发的写线程的干扰。例如现在有一个读线程读到了A节点，写线程把C删掉了，但是看上图，读线程仍然可以继续读下去；当然，如果在删除C之前读线程读到的是D，那么更不会有影响。

​    根据上面所提到的ConcurrentHashMap中删除一个节点并不会立刻被读线程感受到的效果，就是传说中的**弱一致性**，所以ConcurrentHashMap的迭代器是弱一致性迭代器
