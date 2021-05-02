---
tags:
	- 基础数据类型
categories: ConcurrentHashMap
title: ConcurrentHashMap的使用
---

# ConcurrentHashMap的使用

缓存的使用

- 高性能本地缓存：对系统中常用到的业务数据放到缓存中以提高系统性能，限制是单服务器模式
- 分布式缓存：常用分布式缓存技术memcached、redis等

ConcurrentHashMap就是常用的高并发下的缓存对象。
<!-- more -->
接下来直接上例子：
```java
public class ConcurrentMapTest {
  
	public static ConcurrentMap<String, Future<String>> cMap 
      = new ConcurrentHashMap<>();
  
	public static ConcurrentHashMap<String, String> cMap2 
      = new ConcurrentHashMap<>();
  
	public static int index = 0;

	public static void main(String[] args) {
		for (int i = 0; i < 5; i++) {
			new Thread(new Runnable() {
				@Override
				public void run() {
					concurrentMap2("3");
					try {
						concurrentMap("123");
					} catch (InterruptedException | 
                             ExecutionException | TimeoutException e) {
						e.printStackTrace();
					}
				}
			}).start();
		}
	}

	/**
	 * 解决并发写的线程安全问题。但是高并发可读取会造成重复写的问题...
	 * 如果put的业务计算复杂将耗费不必要的资源
	 * 解决缓存读取问题，但可能会出现缓存重复写
	 */
	private static void concurrentMap2(String key) {
		System.out.println(Thread.currentThread().getName() + " start ....");
		String f = cMap2.get(key);
		// ConcurrentMap读不加锁，写加锁。
        // 当并发量高时会出现重复compute的操作，然后才put到map中
		if (f == null) {
			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			cMap2.put(key, "dataTest" + index);
			System.out.println(Thread.currentThread().getName() 
                               + " compute , index================== " + index++);
		}
		System.out.println(Thread.currentThread().getName() + " end .... " 
                           + cMap2.get(key));

	}

	/**
	 * 解决并发写问题，同时避免了重复put计算的问题 解决缓存读写的问题
	 * 
	 * @param key
	 * @throws InterruptedException
	 * @throws ExecutionException
	 * @throws TimeoutException
	 */
	private static void concurrentMap(String key) throws InterruptedException, ExecutionException, TimeoutException {
		System.out.println(Thread.currentThread().getName() + " start ....");
		Future<String> f = null;
		f = cMap.get(key);
		if (f == null) {
			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			FutureTask<String> fTask = new FutureTask<String>
              (new Callable<String>() {
				public String call() throws Exception {

					try {
						Thread.sleep(2000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println(Thread.currentThread().getName() 
                                       + " compute , index=============== "
                                       + index++);
					return "456789123";
				}
			});
			f = cMap.putIfAbsent(key, fTask); // 相当于get-if-absent-compute，
             //而且是原子执行，解决了并发读的问题。（FutureTask解决compute步骤）		
			if (f == null) {
				f = fTask;
				// f的值是FutureTask对象引用，解决了call的重复调用问题,
                 // 只用一个线程会执行run()方法
				fTask.run();
			}
		}
		System.out.println("end ===========");
		// get会等待FutureTask的计算结果，可以设置等待超时事件,超时会抛出超时异常
		System.out.println(Thread.currentThread().getName() 
                           + " end ....====== " 
                           + f.get(3000, TimeUnit.MILLISECONDS));
		// get会等待FutureTask的计算结果，永久等待
		System.out.println(Thread.currentThread().getName() 
                           + " end .... " + f.get());
	}

}
```

concurrentMap2的执行结果

Thread-0 start ....
Thread-2 start ....
Thread-4 start ....
Thread-1 start ....
Thread-3 start ....
Thread-4 compute , index================== 0
Thread-0 compute , index================== 2
Thread-0 end .... dataTest0
Thread-2 compute , index================== 1
Thread-2 end .... dataTest0
Thread-4 end .... dataTest0
Thread-3 compute , index================== 3
Thread-1 compute , index================== 4
Thread-1 end .... dataTest3
Thread-3 end .... dataTest3

 可以看到put方法被重复执行....

concurrentMap的执行结果

Thread-1 start ....
Thread-3 start ....
Thread-0 start ....
Thread-2 start ....
Thread-4 start ....
end ===========
end ===========
end ===========
end ===========
Thread-1 compute , index=============== 0
Thread-3 end ....====== 456789123
Thread-3 end .... 456789123
end ===========
Thread-1 end ....====== 456789123
Thread-1 end .... 456789123
Thread-0 end ....====== 456789123
Thread-0 end .... 456789123
Thread-2 end ....====== 456789123
Thread-2 end .... 456789123
Thread-4 end ....====== 456789123
Thread-4 end .... 456789123

可以看到put运算只执行一次....

总结：

1. 如果缓存对象可以在系统启动时进行初始化加载，可以不使用ConcurrentHashMap
2. 如果缓存在put时计算比较复杂，那么推荐直接使用concurrentMap写法
3. ConcurrentHashMap缺陷就是缓存无法回收，导致内存溢出问题。此问题在google发布Guava的Cache很好的进行了处理，可查看另一篇文章[Guava Cache的使用](http://itfish.net/article/64821.html#)
