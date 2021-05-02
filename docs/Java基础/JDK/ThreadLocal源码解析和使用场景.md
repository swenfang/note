---
tags:
	- Java 源码解读
categories: ThreadLocal
title: ThreadLocal 源码解析和使用场景
---
# ThreadLocal 源码解析和使用场景

## ThreadLocal 主要用途

ThreadLocal 是在 JDK 包里面提供的，它提供了线程本地变量，也就是如果你创建了一个 ThreadLocal 变量，那么访问这个变量的每个线程都会有这个变量的一个本地拷贝，多个线程操作这个变量的时候，实际是操作的自己本地内存里面的变量，从而避免了线程安全问题，创建一个ThreadLocal变量后每个线程会拷贝一个变量到自己本地内存，如下图：

![mark](https://blogimg.nos-eastchina1.126.net/180316/6IE9Kh6dAe.png)

- 从JAVA官方对 ThreadLocal 类的说明定义（定义在示例代码中）：ThreadLocal  类用来提供线程内部的局部变量。这种变量在多线程环境下访问（通过 get 和 set 方法访问）时能保证各个线程的变量相对独立于其他线程内的变量。ThreadLocal 实例通常来说都是 private static 类型的，用于关联线程和线程上下文。
- 我们可以得知 ThreadLocal 的作用是：ThreadLocal 的作用是提供线程内的局部变量，不同的线程之间不会相互干扰，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或组件之间一些公共变量的传递的复杂度。
- 上述可以概述为：ThreadLocal 提供线程内部的局部变量，在本线程内随时随地可取，隔离其他线程。

## ThreadLocal使用实例

使用例子来调试，看下ThreadLocal如何使用，从而加深理解。例子开启了两个线程，每个线程内部设置了本地变量的值，然后调用print函数打印当前本地变量的值，如果打印后调用了本地变量额remove方法则会删除本地内存中的该变量，代码如下：

```java
public class ThreadLocalTest {

    //(1)打印函数
    static void print(String str){
        //1.1  打印当前线程本地内存中localVariable变量的值
        System.out.println(str + ":" +localVariable.get());
        //1.2 清除当前线程本地内存中localVariable变量
        //localVariable.remove();
    }
    //(2) 创建ThreadLocal变量
    static ThreadLocal<String> localVariable = new ThreadLocal<>();
    public static void main(String[] args) {

        //(3) 创建线程one
        Thread threadOne = new Thread(new  Runnable() {
            public void run() {
                //3.1 设置线程one中本地变量localVariable的值
                localVariable.set("threadOne local variable");
                //3.2 调用打印函数
                print("threadOne");
                //3.3打印本地变量值
                System.out.println("threadOne remove after" + ":" +localVariable.get());

            }
        });
        //(4) 创建线程two
        Thread threadTwo = new Thread(new  Runnable() {
            public void run() {
                //4.1 设置线程one中本地变量localVariable的值
                localVariable.set("threadTwo local variable");
                //4.2 调用打印函数
                print("threadTwo");
                //4.3打印本地变量值
                System.out.println("threadTwo remove after" + ":" +localVariable.get());

            }
        });
        //(5)启动线程
        threadOne.start();
        threadTwo.start();
    }

```

运行结果：

```
threadOne:threadOne local variable
threadTwo:threadTwo local variable
threadOne remove after:threadOne local variable
threadTwo remove after:threadTwo local variable

```

- 代码（2）创建了一个ThreadLocal变量
- 代码（3）（4）分别创建了线程one和two
- 代码（5）启动了两个线程。
- 线程one中代码3.1通过set方法设置了localVariable的值，这个设置的其实是线程one本地内存中的一个拷贝，这个拷贝线程two是访问不了的。然后代码3.2调用了print函数，代码1.1通过get函数获取了当前线程（线程one）本地内存中localVariable的值。
- 线程two执行类似线程one

解开代码1.2的注释后，再次运行，运行结果为：

```
threadOne:threadOne local variable
threadOne remove after:null
threadTwo:threadTwo local variable
threadTwo remove after:null

```

## ThreadLocal实现原理

首先看下ThreadLocal相关的类的类图结构

![img](https://user-gold-cdn.xitu.io/2018/1/3/160baaa075e68be6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如上类图可知 Thread 类中有一个 threadLocals 和 inheritableThreadLocals 都是 ThreadLocalMap 类型的变量，而 **ThreadLocalMap 是一个定制化的 HashMap**，默认每个线程中这个两个变量都为null。

只有当前线程第一次调用了 ThreadLocal 的set或者get方法时候才会进行创建。其实每个线程的本地变量不是存放到 ThreadLocal 实例里面的，而是存放到调用线程的 threadLocals 变量里面。

**也就是说 ThreadLocal 类型的本地变量是存放到具体的线程内存空间的。**

**ThreadLocal 就是一个工具壳，它通过 set 方法把 value 值放入调用线程的 threadLocals 里面存放起来，当调用线程调用它的 get 方法时候再从当前线程的 threadLocals 变量里面拿出来使用。**

如果调用线程一直不终止那么这个本地变量会一直存放到调用线程的 threadLocals 变量里面，所以当不需要使用本地变量时候可以通过调用ThreadLocal 变量的 remove 方法，从当前线程的 threadLocals 里面删除该本地变量。

另外 Thread 里面的threadLocals 为何设计为 map 结构那？很明显是因为每个线程里面可以关联多个 ThreadLocal 变量。

![mark](https://blogimg.nos-eastchina1.126.net/180301/mb7C7FllE1.png)

### 为什么使用了弱引用

ThreadLocalMap 中的存储实体 Entry 使用 ThreadLocal 作为 key，但这个 Entry 是继承弱引用 WeakReference 的，为什么要这样设计，使用了弱引用 WeakReference 会造成内存泄露问题吗？

- 首先，回答这个问题之前，我需要解释一下什么是强引用，什么是弱引用。

我们在正常情况下，普遍使用的是强引用：

```
A a = new A();

B b = new B();
```

当  a = null;b = null; 时，一段时间后，JAVA垃圾回收机制GC会将 a 和 b 对应所分配的内存空间给回收。

但考虑这样一种情况：

```
C c = new C(b);
b = null;
```

当 b 被设置成 null 时，那么是否意味这一段时间后GC工作可以回收 b 所分配的内存空间呢？答案是否定的，因为即使 b 被设置成 null ，但 c 仍然持有对 b 的引用，而且还是强引用，所以GC不会回收 b 原先所分配的空间，既不能回收，又不能使用，这就造成了 内存泄露。

**那么我们该如何处理呢？**

可以通过 c = null;，也可以使用弱引用 WeakReference w = new WeakReference(b); 。因为使用了弱引用 WeakReference，GC 是可以回收 b 原先所分配的空间的。

上述解释主要参考自：[对ThreadLocal实现原理的一点思考](https://link.juejin.im/?target=http%3A%2F%2Fwww.jianshu.com%2Fp%2Fee8c9dccc953)

- 回到  ThreadLocal  的层面上，ThreadLocalMap  使用 ThreadLocal  的弱引用作为key，如果一个ThreadLocal 没有外部强引用来引用它，那么系统 GC 的时候，这个 ThreadLocal 势必会被回收，这样一来，ThreadLocalMap 中就会出现 key 为 null 的 Entry，就没有办法访问这些 key 为 null 的 Entry 的 value，如果当前线程再迟迟不结束的话，这些 key 为 null 的 Entry 的 value 就会一直存在一条强引用链：`Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value` 永远无法回收，造成内存泄漏。

其实，ThreadLocalMap的设计中已经考虑到这种情况，也加上了一些防护措施：在 ThreadLocal 的get(), set(), remove() 的时候都会清除线程 ThreadLocalMap 里所有 key 为null 的 value。

但是这些被动的预防措施并不能保证不会内存泄漏：

- 使用static的ThreadLocal，延长了ThreadLocal的生命周期，可能导致的内存泄漏（参考[ThreadLocal 内存泄露的实例分析](https://link.juejin.im/?target=http%3A%2F%2Fblog.xiaohansong.com%2F2016%2F08%2F09%2FThreadLocal-leak-analyze%2F)）。
- 分配使用了ThreadLocal又不再调用get(),set(),remove()方法，那么就会导致内存泄漏。

从表面上看内存泄漏的根源在于使用了弱引用。网上的文章大多着重分析 ThreadLocal 使用了弱引用会导致内存泄漏，但是另一个问题也同样值得思考：为什么使用弱引用而不是强引用？

我们先来看看官方文档的说法：

```
To help deal with very large and long-lived usages, 
the hash table entries use WeakReferences for keys.
```

为了应对非常大和长时间的用途，哈希表使用弱引用的 key。

下面我们分两种情况讨论：

- key 使用强引用：引用的 ThreadLocal 的对象被回收了，但是 ThreadLocalMap 还持有 ThreadLocal 的强引用，如果没有手动删除，ThreadLocal 不会被回收，导致 Entry 内存泄漏。
- key  使用弱引用：引用的 ThreadLocal 的对象被回收了，由于 ThreadLocalMap 持有 ThreadLocal 的弱引用，即使没有手动删除，ThreadLocal 也会被回收。value 在下一次 ThreadLocalMap 调用get() ,set(),remove()的时候会被清除。
- 比较两种情况，我们可以发现：由于 ThreadLocalMap 的生命周期跟 Thread 一样长，如果都没有手动删除对应 key ，都会导致内存泄漏，但是使用弱引用可以多一层保障：弱引用 ThreadLocal  不会内存泄漏，对应的 value 在下一次 ThreadLocalMap 调用 get() ,set(),remove()的时候会被清除。

因此， ThreadLocal 内存泄漏的根源是：由于 ThreadLocalMap 的生命周期跟 Thread 一样长，如果没有手动删除对应 key 就会导致内存泄漏，而不是因为弱引用。

综合上面的分析，我们可以理解ThreadLocal内存泄漏的前因后果，那么怎么避免内存泄漏呢？

每次使用完ThreadLocal，都调用它的 remove() 方法，清除数据。

在使用线程池的情况下，没有及时清理 ThreadLocal，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用 ThreadLocal 就跟加锁完要解锁一样，用完就清理。

上述解释主要参考自：[深入分析 ThreadLocal 内存泄漏问题](https://link.juejin.im/?target=http%3A%2F%2Fblog.xiaohansong.com%2F2016%2F08%2F06%2FThreadLocal-memory-leak%2F)

### 常用操作的底层实现原理

根据上面的例子，我们进行调试，看看一下几个常用操作的实现原理。

#### get() 方法

```java
    /**
     * 返回当前线程对应的ThreadLocal的初始值
     * 此方法的第一次调用发生在，当线程通过{@link #get}方法访问此线程的ThreadLocal值时
     * 除非线程先调用了 {@link #set}方法，在这种情况下，
     * {@code initialValue} 才不会被这个线程调用。
     * 通常情况下，每个线程最多调用一次这个方法，
     * 但也可能再次调用，发生在调用{@link #remove}方法后，
     * 紧接着调用{@link #get}方法。
     *
     * <p>这个方法仅仅简单的返回null {@code null};
     * 如果程序员想ThreadLocal线程局部变量有一个除null以外的初始值，
     * 必须通过子类继承{@code ThreadLocal} 的方式去重写此方法
     * 通常, 可以通过匿名内部类的方式实现
     *
     * @return 当前ThreadLocal的初始值
     */
    protected T initialValue() {
        return null;
    }

    /**
     * 创建一个ThreadLocal
     * @see #withInitial(java.util.function.Supplier)
     */
    public ThreadLocal() {
    }

    /**
     * 返回当前线程中保存ThreadLocal的值
     * 如果当前线程没有此ThreadLocal变量，
     * 则它会通过调用{@link #initialValue} 方法进行初始化值
     *
     * @return 返回当前线程对应此ThreadLocal的值
     */
    public T get() {
        // 获取当前线程对象
        Thread t = Thread.currentThread();
        // 获取此线程对象中维护的ThreadLocalMap对象
        ThreadLocalMap map = getMap(t);
        // 如果此map存在
        if (map != null) {
            // 以当前的ThreadLocal 为 key，调用getEntry获取对应的存储实体e
            ThreadLocalMap.Entry e = map.getEntry(this);
            // 找到对应的存储实体 e 
            if (e != null) {
                @SuppressWarnings("unchecked")
                // 获取存储实体 e 对应的 value值
                // 即为我们想要的当前线程对应此ThreadLocal的值
                T result = (T)e.value;
                return result;
            }
        }
        // 如果map不存在，则证明此线程没有维护的ThreadLocalMap对象
        // 调用setInitialValue进行初始化
        return setInitialValue();
    }

    /**
     * set的变样实现，用于初始化值initialValue，
     * 用于代替防止用户重写set()方法
     *
     * @return the initial value 初始化后的值
     */
    private T setInitialValue() {
        // 调用initialValue获取初始化的值
        T value = initialValue();
        // 获取当前线程对象
        Thread t = Thread.currentThread();
        // 获取此线程对象中维护的ThreadLocalMap对象
        ThreadLocalMap map = getMap(t);
        // 如果此map存在
        if (map != null)
            // 存在则调用map.set设置此实体entry
            map.set(this, value);
        else
            // 1）当前线程Thread 不存在ThreadLocalMap对象
            // 2）则调用createMap进行ThreadLocalMap对象的初始化
            // 3）并将此实体entry作为第一个值存放至ThreadLocalMap中
            createMap(t, value);
        // 返回设置的值value
        return value;
    }

    /**
     * 获取当前线程Thread对应维护的ThreadLocalMap 
     * 
     * @param  t the current thread 当前线程
     * @return the map 对应维护的ThreadLocalMap 
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```

- 调用 get() 操作获取 ThreadLocal 中对应当前线程存储的值时，进行了如下操作：

  1 ) 获取当前线程 Thread 对象，进而获取此线程对象中维护的 ThreadLocalMap 对象。

  2 ) 判断当前的 ThreadLocalMap 是否存在：

- 如果存在，则以当前的ThreadLocal为 key，调用ThreadLocalMap中的getEntry方法获取对应的存储实体 e。找到对应的存储实体 e，获取存储实体 e 对应的 value 值，即为我们想要的当前线程对应此ThreadLocal的值，返回结果值。

- 如果不存在，则证明此线程没有维护的 ThreadLocalMap 对象，调用 setInitialValue 方法进行初始化。返回setInitialValue 初始化的值。

- setInitialValue 方法的操作如下：

  1 ) 调用initialValue获取初始化的值。

  2 ) 获取当前线程Thread对象，进而获取此线程对象中维护的ThreadLocalMap对象。

  3 ) 判断当前的ThreadLocalMap是否存在：

- 如果存在，则调用map.set 设置此实体entry。

- 如果不存在，则调用createMap 进行 ThreadLocalMap 对象的初始化，并将此实体 entry 作为第一个值存放至 ThreadLocalMap 中。



#### set() 方法

```java
    /**
     * 设置当前线程对应的ThreadLocal的值
     * 大多数子类都不需要重写此方法，
     * 只需要重写 {@link #initialValue}方法代替设置当前线程对应的ThreadLocal的值
     *
     * @param value 将要保存在当前线程对应的 ThreadLocal 的值
     *  
     */
    public void set(T value) {
        // 获取当前线程对象
        Thread t = Thread.currentThread();
        // 获取此线程对象中维护的 ThreadLocalMap 对象
        ThreadLocalMap map = getMap(t);
        // 如果此map存在
        if (map != null)
            // 存在则调用map.set设置此实体entry
            map.set(this, value);
        else
            // 1）当前线程Thread 不存在ThreadLocalMap对象
            // 2）则调用createMap进行ThreadLocalMap对象的初始化
            // 3）并将此实体entry作为第一个值存放至ThreadLocalMap中
            createMap(t, value);
    }

    /**
     * 为当前线程Thread 创建对应维护的ThreadLocalMap. 
     *
     * @param t the current thread 当前线程
     * @param firstValue 第一个要存放的ThreadLocal变量值
     */
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

- 调用 set(T value) 操作设置ThreadLocal中对应当前线程要存储的值时，进行了如下操作：

  1 ) 获取当前线程 Thread 对象，进而获取此线程对象中维护的 ThreadLocalMap 对象。

  2 ) 判断当前的 ThreadLocalMap 是否存在：

- 如果存在，则调用 map.set 设置此实体 entry。

- 如果不存在，则调用 createMap 进行 ThreadLocalMap 对象的初始化，并将此实体 entry 作为第一个值存放至 ThreadLocalMap 中。



#### remove() 方法

```java
  /**
   * 删除当前线程中保存的ThreadLocal对应的实体entry
   * 如果此ThreadLocal变量在当前线程中调用 {@linkplain #get read}方法
   * 则会通过调用{@link #initialValue}进行再次初始化，
   * 除非此值value是通过当前线程内置调用 {@linkplain #set set}设置的
   * 这可能会导致在当前线程中多次调用{@code initialValue}方法
   *
   * @since 1.5
   */
   public void remove() {
      // 获取当前线程对象中维护的ThreadLocalMap对象
       ThreadLocalMap m = getMap(Thread.currentThread());
      // 如果此map存在
       if (m != null)
          // 存在则调用map.remove
          // 以当前ThreadLocal为key删除对应的实体entry
           m.remove(this);
   }
```

调用 remove() 操作删除ThreadLocal中对应当前线程已存储的值时，进行了如下操作：

1.  获取当前线程 Thread 对象，进而获取此线程对象中维护的 ThreadLocalMap 对象。
2.  判断当前的 ThreadLocalMap 是否存在， 如果存在，则调用 map.remove ，以当前 ThreadLocal 为 key 删除对应的实体 entry。

### ThreadLocalMap的内部底层实现

对 ThreadLocal 的常用操作实际是对线程Thread中的ThreadLocalMap进行操作，核心是ThreadLocalMap这个哈希表，接着我们来谈谈ThreadLocalMap的内部底层实现。

源代码：

```java
    /**
     * ThreadLocalMap 是一个定制的自定义 hashMap 哈希表，只适合用于维护
     * 线程对应ThreadLocal的值. 此类的方法没有在ThreadLocal 类外部暴露，
     * 此类是私有的，允许在 Thread 类中以字段的形式声明 ，     
     * 以助于处理存储量大，生命周期长的使用用途，
     * 此类定制的哈希表实体键值对使用弱引用WeakReferences 作为key， 
     * 但是, 一旦引用不在被使用，
     * 只有当哈希表中的空间被耗尽时，对应不再使用的键值对实体才会确保被 移除回收。
     */
    static class ThreadLocalMap {

        /**
         * 实体entries在此hash map中是继承弱引用 WeakReference, 
         * 使用ThreadLocal 作为 key 键.  请注意，当key为null（i.e. entry.get()
         * == null) 意味着此key不再被引用,此时实体entry 会从哈希表中删除。
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** 当前 ThreadLocal 对应储存的值value. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

        /**
         * 初始容量大小 16 -- 必须是2的n次方.
         */
        private static final int INITIAL_CAPACITY = 16;

        /**
         * 底层哈希表 table, 必要时需要进行扩容.
         * 底层哈希表 table.length 长度必须是2的n次方.
         */
        private Entry[] table;

        /**
         * 实际存储键值对元素个数 entries.
         */
        private int size = 0;

        /**
         * 下一次扩容时的阈值
         */
        private int threshold; // 默认为 0

        /**
         * 设置触发扩容时的阈值 threshold
         * 阈值 threshold = 底层哈希表table的长度 len * 2 / 3
         */
        private void setThreshold(int len) {
            threshold = len * 2 / 3;
        }

        /**
         * 获取该位置i对应的下一个位置index
         */
        private static int nextIndex(int i, int len) {
            return ((i + 1 < len) ? i + 1 : 0);
        }

        /**
         * 获取该位置i对应的上一个位置index
         */
        private static int prevIndex(int i, int len) {
            return ((i - 1 >= 0) ? i - 1 : len - 1);
        }

    }
```

- ThreadLocalMap 的底层实现是一个定制的自定义 HashMap 哈希表，核心组成元素有：

  1 ) Entry[] table; ：底层哈希表 table, 必要时需要进行扩容，底层哈希表 table.length 长度必须是2的n次方。

  2 ) int size;：实际存储键值对元素个数 entries

  3 ) int threshold;：下一次扩容时的阈值，阈值 threshold = len * 2 / 3 (底层哈希表table的长度)。当`size >= threshold`时，遍历 table 并删除 key 为 null 的元素，如果删除后`size >= threshold*3/4`时，需要对table 进行扩容

- 其中 Entry[] table; 哈希表存储的核心元素是 Entry ，Entry 包含：

  1 ) **ThreadLocal<?> k**；：当前存储的 ThreadLocal 实例对象

  2 ) **Object value**;：当前 ThreadLocal 对应储存的值value

- 需要注意的是，此 Entry 继承了弱引用  WeakReference ，所以在使用 ThreadLocalMap 时，发现`key == null`，则意味着此 key ThreadLocal 不在被引用，需要将其从 ThreadLocalMap 哈希表中移除。



```java
        /**
         * 用于创建一个新的hash map包含 (firstKey, firstValue).
         * ThreadLocalMaps 构造方法是延迟加载的,所以我们只会在至少有一个
         * 实体entry存放时，才初始化创建一次（仅初始化一次）。
         */
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            // 初始化 table 初始容量为 16
            table = new Entry[INITIAL_CAPACITY];
            // 计算当前entry的存储位置
            // 存储位置计算等价于：
            // ThreadLocal 的 hash 值 threadLocalHashCode  % 哈希表的长度 length
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            // 存储当前的实体，key 为 : 当前ThreadLocal  value：真正要存储的值
            table[i] = new Entry(firstKey, firstValue);
            // 设置当前实际存储元素个数 size 为 1
            size = 1;
            // 设置阈值，为初始化容量 16 的 2/3。
            setThreshold(INITIAL_CAPACITY);
        }
```

ThreadLocalMap 的构造方法是延迟加载的，也就是说，只有当线程需要存储对应的 ThreadLocal 的值时，才初始化创建一次（仅初始化一次）。初始化步骤如下：

1） 初始化底层数组 table 的初始容量为 16。

2） 获取 ThreadLocal 中的 threadLocalHashCode ，通过`threadLocalHashCode & (INITIAL_CAPACITY - 1)`，即ThreadLocal 的 hash 值 threadLocalHashCode % 哈希表的长度 length 的方式计算该实体的存储位置。

3） 存储当前的实体，key 为 : 当前ThreadLocal value：真正要存储的值

4）设置当前实际存储元素个数 size 为 1

5）设置阈值`setThreshold(INITIAL_CAPACITY)`，为初始化容量 16 的 2/3。



源代码：

```java
        /**
         * 根据key 获取对应的实体 entry.  此方法快速适用于获取某一存在key的
         * 实体 entry，否则，应该调用getEntryAfterMiss方法获取，这样做是为
         * 了最大限制地提高直接命中的性能
         *
         * @param  key 当前thread local 对象
         * @return the entry 对应key的 实体entry, 如果不存在，则返回null
         */
        private Entry getEntry(ThreadLocal<?> key) {
            // 计算要获取的entry的存储位置
            // 存储位置计算等价于：
            // ThreadLocal 的 hash 值 threadLocalHashCode  % 哈希表
            的长度 length
            int i = key.threadLocalHashCode & (table.length - 1);
            // 获取到对应的实体 Entry 
            Entry e = table[i];
            // 存在对应实体并且对应key相等，即同一ThreadLocal
            if (e != null && e.get() == key)
                // 返回对应的实体Entry 
                return e;
            else
                // 不存在 或 key不一致，则通过调用getEntryAfterMiss继续查找
                return getEntryAfterMiss(key, i, e);
        }

        /**
         * 当根据key找不到对应的实体entry 时，调用此方法。
         * 直接定位到对应的哈希表位置
         *
         * @param  key 当前thread local 对象
         * @param  i 此对象在哈希表 table中的存储位置 index
         * @param  e the entry 实体对象
         * @return the entry 对应key的 实体entry, 如果不存在，则返回null
         */
        private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;
            // 循环遍历当前位置的所有实体entry
            while (e != null) {
                // 获取当前entry 的 key ThreadLocal
                ThreadLocal<?> k = e.get();
               // 比较key是否一致，一致则返回
                if (k == key)
                    return e;
                // 找到对应的entry ，但其key 为 null，则证明引用已经不存在
                // 这是因为Entry继承的是WeakReference，这是弱引用带来的坑
                if (k == null)
                    // 删除过期(stale)的entry
                    expungeStaleEntry(i);
                else
                    // key不一致 ，key也不为空，则遍历下一个位置，继续查找
                    i = nextIndex(i, len);
                // 获取下一个位置的实体 entry
                e = tab[i];
            }
            // 遍历完毕，找不到则返回null
            return null;
        }


        /**
         * 删除对应位置的过期实体，并删除此位置后对应相关联位置key = null的实体
         *
         * @param staleSlot 已知的key = null 的对应的位置索引
         * @return 对应过期实体位置索引的下一个key = null的位置
         * (所有的对应位置都会被检查)
         */
        private int expungeStaleEntry(int staleSlot) {
            // 获取对应的底层哈希表 table
            Entry[] tab = table;
            // 获取哈希表长度
            int len = tab.length;

            // 擦除这个位置上的脏数据
            tab[staleSlot].value = null;
            tab[staleSlot] = null;
            size--;

            // 直到我们找到 Entry e = null，才执行rehash操作
            // 就是遍历完该位置的所有关联位置的实体
            Entry e;
            int i;
            // 查找该位置对应所有关联位置的过期实体，进行擦除操作
            for (i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();
                if (k == null) {
                    e.value = null;
                    tab[i] = null;
                    size--;
                } else {
                    int h = k.threadLocalHashCode & (len - 1);
                    if (h != i) {
                        tab[i] = null;

                        // 我们必须一直遍历直到最后
                        // 因为还可能存在多个过期的实体
                        while (tab[h] != null)
                            h = nextIndex(h, len);
                        tab[h] = e;
                    }
                }
            }
            return i;
        }

        /**
         * 删除所有过期的实体
         */
        private void expungeStaleEntries() {
            Entry[] tab = table;
            int len = tab.length;
            for (int j = 0; j < len; j++) {
                Entry e = tab[j];
                if (e != null && e.get() == null)
                    expungeStaleEntry(j);
            }
        }
```

- ThreadLocal 的 get() 操作实际是调用 ThreadLocalMap 的 getEntry(ThreadLocal<?> key) 方法,此方法快速适用于获取某一存在 key 的实体 entry，否则，应该调用`getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e)`方法获取，这样做是为了最大限制地提高直接命中的性能，该方法进行了如下操作：

  1 ) 计算要获取的 entry 的存储位置，存储位置计算等价于：ThreadLocal 的 hash 值 threadLocalHashCode % 哈希表的长度 length。

  2 ) 根据计算的存储位置，获取到对应的实体 Entry。判断对应实体 Entry 是否存在 并且 key 是否相等：

- 存在对应实体 Entry 并且对应 key 相等，即同一 ThreadLocal ，返回对应的实体 Entry。

- 不存在对应实体 Entry 或者  key 不相等，则通过调用`getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e)`方法继续查找。

- `getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e)`方法操作如下：

  1 ) 获取底层哈希表数组 table，循环遍历对应要查找的实体 Entry 所关联的位置。

  2 ) 获取当前遍历的 entry 的  key ThreadLocal ，比较 key 是否一致，一致则返回。

  3 ) 如果 key 不一致 并且 key 为 null，则证明引用已经不存在，这是因为 Entry 继承的是 WeakReference，这是弱引用带来的坑。调用`expungeStaleEntry(int staleSlot)`方法删除过期的实体 Entry（此方法不单独解释，请查看示例代码，有详细注释说明）。

  4 ) key不一致 ，key也不为空，则遍历下一个位置，继续查找。

  5 ) 遍历完毕，仍然找不到则返回null。

  ​

源代码：

```java
/**
     * 设置对应ThreadLocal的值
     *
     * @param key 当前thread local 对象
     * @param value 要设置的值
     */
    private void set(ThreadLocal<?> key, Object value) {

        // 我们不会像get()方法那样使用快速设置的方式，
        // 因为通常很少使用set()方法去创建新的实体
        // 相对于替换一个已经存在的实体, 在这种情况下,
        // 快速设置方案会经常失败。

        // 获取对应的底层哈希表 table
        Entry[] tab = table;
        // 获取哈希表长度
        int len = tab.length;
        // 计算对应threalocal的存储位置
        int i = key.threadLocalHashCode & (len-1);

        // 循环遍历table对应该位置的实体，查找对应的threadLocal
        for (Entry e = tab[i];e != null;e = tab[i = nextIndex(i, len)]) {
            // 获取当前位置的ThreadLocal
            ThreadLocal<?> k = e.get();
            // 如果key threadLocal一致，则证明找到对应的threadLocal
            if (k == key) {
                // 赋予新值
                e.value = value;
                // 结束
                return;
            }
            // 如果当前位置的key threadLocal为null
            if (k == null) {
                // 替换该位置key == null 的实体为当前要设置的实体
                replaceStaleEntry(key, value, i);
                // 结束
                return;
            }
        }
        // 当前位置的k ！= key  && k != null
        // 创建新的实体，并存放至当前位置i
        tab[i] = new Entry(key, value);
        // 实际存储键值对元素个数 + 1
        int sz = ++size;
        // 由于弱引用带来了这个问题，所以先要清除无用数据，才能判断现在的size有没有达到阀值threshhold
        // 如果没有要清除的数据，存储元素个数仍然 大于 阈值 则扩容
        if (!cleanSomeSlots(i, sz) && sz >= threshold)
            // 扩容
            rehash();
    }

    /**
     * 当执行set操作时，获取对应的key threadLocal，并替换过期的实体
     * 将这个value值存储在对应key threadLocal的实体中，无论是否已经存在体
     * 对应的key threadLocal
     *
     * 有一个副作用, 此方法会删除该位置下和该位置nextIndex对应的所有过期的实体
     *
     * @param  key 当前thread local 对象
     * @param  value 当前thread local 对象对应存储的值
     * @param  staleSlot 第一次找到此过期的实体对应的位置索引index
     *         .
     */
    private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                   int staleSlot) {
        // 获取对应的底层哈希表 table
        Entry[] tab = table;
        // 获取哈希表长度
        int len = tab.length;
        Entry e;

        // 往前找，找到table中第一个过期的实体的下标
        // 清理整个table是为了避免因为垃圾回收带来的连续增长哈希的危险
        // 也就是说，哈希表没有清理干净，当GC到来的时候，后果很严重

        // 记录要清除的位置的起始首位置
        int slotToExpunge = staleSlot;
        // 从该位置开始，往前遍历查找第一个过期的实体的下标
        for (int i = prevIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = prevIndex(i, len))
            if (e.get() == null)
                slotToExpunge = i;

        // 找到key一致的ThreadLocal或找到一个key为 null的
        for (int i = nextIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = nextIndex(i, len)) {
            ThreadLocal<?> k = e.get();

            // 如果我们找到了key，那么我们就需要把它跟新的过期数据交换来保持哈希表的顺序
            // 那么剩下的过期Entry呢，就可以交给expungeStaleEntry方法来擦除掉
            // 将新设置的实体放置在此过期的实体的位置上
            if (k == key) {
                // 替换，将要设置的值放在此过期的实体中
                e.value = value;
                tab[i] = tab[staleSlot];
                tab[staleSlot] = e;

                // 如果存在，则开始清除之前过期的实体
                if (slotToExpunge == staleSlot)
                    slotToExpunge = i;
                // 在这里开始清除过期数据
                cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
                return;
            }

            // / 如果我们没有在往后查找中找没有找到过期的实体，
            // 那么slotToExpunge就是第一个过期Entry的下标了
            if (k == null && slotToExpunge == staleSlot)
                slotToExpunge = i;
        }

        // 最后key仍没有找到，则将要设置的新实体放置
        // 在原过期的实体对应的位置上。
        tab[staleSlot].value = null;
        tab[staleSlot] = new Entry(key, value);

        // 如果该位置对应的其他关联位置存在过期实体，则清除
        if (slotToExpunge != staleSlot)
            cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
    }
```


```java
    /**
     * 启发式的扫描查找一些过期的实体并清除，
     * 此方法会再添加新实体的时候被调用, 
     * 或者过期的元素被清除时也会被调用.
     * 如果实在没有过期数据，那么这个算法的时间复杂度就是O(log n)
     * 如果有过期数据，那么这个算法的时间复杂度就是O(n)
     * 
     * @param i 一个确定不是过期的实体的位置，从这个位置i开始扫描
     *
     * @param n 扫描控制: 有{@code log2(n)} 单元会被扫描,
     * 除非找到了过期的实体, 在这种情况下
     * 有{@code log2(table.length)-1} 的格外单元会被扫描.
     * 当调用插入时, 这个参数的值是存储实体的个数，
     * 但如果调用 replaceStaleEntry方法, 这个值是哈希表table的长度
     * (注意: 所有的这些都可能或多或少的影响n的权重
     * 但是这个版本简单，快速，而且似乎执行效率还可以）
     *
     * @return true 返回true，如果有任何过期的实体被删除。
     */
    private boolean cleanSomeSlots(int i, int n) {
        boolean removed = false;
        Entry[] tab = table;
        int len = tab.length;
        do {
            i = nextIndex(i, len);
            Entry e = tab[i];
            if (e != null && e.get() == null) {
                n = len;
                removed = true;
                i = expungeStaleEntry(i);
            }
        } while ( (n >>>= 1) != 0);
        return removed;
    }
```


```java
    /**
     * 哈希表扩容方法
     * 首先扫描整个哈希表table，删除过期的实体
     * 缩小哈希表table大小 或 扩大哈希表table大小，扩大的容量是加倍.
     */
    private void rehash() {
        // 删除所有过期的实体
        expungeStaleEntries();

        // 使用较低的阈值threshold加倍以避免滞后
        // 存储实体个数 大于等于 阈值的3/4则扩容
        if (size >= threshold - threshold / 4)
            resize();
    }

    /**
     * 扩容方法，以2倍的大小进行扩容
     * 扩容的思想跟HashMap很相似，都是把容量扩大两倍
     * 不同之处还是因为WeakReference带来的
     */
    private void resize() {
        // 记录旧的哈希表
        Entry[] oldTab = table;
        // 记录旧的哈希表长度
        int oldLen = oldTab.length;
        // 新的哈希表长度为旧的哈希表长度的2倍
        int newLen = oldLen * 2;
        // 创建新的哈希表
        Entry[] newTab = new Entry[newLen];
        int count = 0;
        // 逐一遍历旧的哈希表table的每个实体，重新分配至新的哈希表中
        for (int j = 0; j < oldLen; ++j) {
            // 获取对应位置的实体
            Entry e = oldTab[j];
            // 如果实体不会null
            if (e != null) {
                // 获取实体对应的ThreadLocal
                ThreadLocal<?> k = e.get(); 
                // 如果该ThreadLocal 为 null
                if (k == null) {
                    // 则对应的值也要清除
                    // 就算是扩容，也不能忘了为擦除过期数据做准备
                    e.value = null; // Help the GC
                } else {
                    // 如果不是过期实体，则根据新的长度重新计算存储位置
                    int h = k.threadLocalHashCode & (newLen - 1);
                   // 将该实体存储在对应ThreadLocal的最后一个位置
                    while (newTab[h] != null)
                        h = nextIndex(h, newLen);
                    newTab[h] = e;
                    count++;
                }
            }
        }
        // 重新分配位置完毕，则重新计算阈值Threshold
        setThreshold(newLen);
        // 记录实际存储元素个数
        size = count;
        // 将新的哈希表赋值至底层table
        table = newTab;
    }
```
ThreadLocal 的`set(T value)`操作实际是调用 ThreadLocalMap 的`set(ThreadLocal<?> key, Object value)`方法，该方法进行了如下操作：

1 ) 获取对应的底层哈希表 table ，计算对应 threalocal 的存储位置。

2 ) 循环遍历 table 对应该位置的实体，查找对应的 threadLocal。

3 ) 获取当前位置的 threadLocal，如果 key threadLocal 一致，则证明找到对应的 threadLocal，将新值赋值给找到的当前实体 Entry 的 value 中，结束。

4 ) 如果当前位置的 key threadLocal 不一致，并且 key threadLocal 为 null，则调用`replaceStaleEntry(ThreadLocal<?> key, Object value,int staleSlot)`方法（此方法不单独解释，请查看示例代码，有详细注释说明），替换该位置`key == null` 的实体为当前要设置的实体，结束。

5 ) 如果当前位置的 key threadLocal 不一致，并且 key threadLocal 不为 null ，则创建新的实体，并存放至当前位置 i `tab[i] = new Entry(key, value);`，实际存储键值对元素个数`size + 1`，由于弱引用带来了这个问题，所以要调用`cleanSomeSlots(int i, int n)`方法清除无用数据（此方法不单独解释，请查看示例代码，有详细注释说明），才能判断现在的 size 有没有达到阀值 threshhold ，如果没有要清除的数据，存储元素个数仍然 大于 阈值 则调用 rehash 方法进行扩容（此方法不单独解释，请查看示例代码，有详细注释说明）。



```java
        /**
         * 移除对应ThreadLocal的实体
         */
        private void remove(ThreadLocal<?> key) {
            // 获取对应的底层哈希表 table
            Entry[] tab = table;
            // 获取哈希表长度
            int len = tab.length;
            // 计算对应threalocal的存储位置
            int i = key.threadLocalHashCode & (len-1);
            // 循环遍历table对应该位置的实体，查找对应的threadLocal
            for (Entry e = tab[i];e != null;e = tab[i = nextIndex(i, len)]) {
                // 如果key threadLocal一致，则证明找到对应的threadLocal
                if (e.get() == key) {
                    // 执行清除操作
                    e.clear();
                    // 清除此位置的实体
                    expungeStaleEntry(i);
                    // 结束
                    return;
                }
            }
        }
```

ThreadLocal 的`remove()`操作实际是调用 ThreadLocalMap 的`remove(ThreadLocal<?> key)`方法，该方法进行了如下操作：

1 ) 获取对应的底层哈希表 table，计算对应 threalocal 的存储位置。

2 ) 循环遍历 table 对应该位置的实体，查找对应的 threadLocal。

3 ) 获取当前位置的threadLocal，如果 key threadLocal 一致，则证明找到对应的 threadLocal，执行删除操作，删除此位置的实体，结束。

## ThreadLocal 在现时有什么应用场景

总的来说 ThreadLocal 主要是解决2种类型的问题：

- 解决并发问题：使用 ThreadLocal 代替 synchronized 来保证线程安全。同步机制采用了“以时间换空间”的方式，而ThreadLocal 采用了“以空间换时间”的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。
- 解决数据存储问题：ThreadLocal 为变量在每个线程中都创建了一个副本，所以每个线程可以访问自己内部的副本变量，不同线程之间不会互相干扰。如一个Parameter对象的数据需要在多个模块中使用，如果采用参数传递的方式，显然会增加模块之间的耦合性。此时我们可以使用 ThreadLocal 解决。

应用场景：

Spring 使用 ThreadLocal 解决线程安全问题

- 我们知道在一般情况下，只有无状态的 Bean 才可以在多线程环境下共享，在 Spring 中，绝大部分 Bean 都可以声明为 singleton 作用域。就是因为 Spring 对一些 Bean（如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等）中非线程安全状态采用ThreadLocal进行处理，让它们也成为线程安全的状态，因为有状态的 Bean 就可以在多线程中共享了。
- 一般的 Web 应用划分为展现层、服务层和持久层三个层次，在不同的层中编写对应的逻辑，下层通过接口向上层开放功能调用。在一般情况下，从接收请求到返回响应所经过的所有程序调用都同属于一个线程ThreadLocal 是解决线程安全问题一个很好的思路，它通过为每个线程提供一个独立的变量副本解决了变量并发访问的冲突问题。在很多情况下，ThreadLocal 比直接使用 synchronized 同步机制解决线程安全问题更简单，更方便，且结果程序拥有更高的并发性。

示例代码：

```java
public abstract class RequestContextHolder  {
····

    private static final boolean jsfPresent =
            ClassUtils.isPresent("javax.faces.context.FacesContext", RequestContextHolder.class.getClassLoader());

    private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
            new NamedThreadLocal<RequestAttributes>("Request attributes");

    private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =
            new NamedInheritableThreadLocal<RequestAttributes>("Request context");

·····
}
```



## ThreadLocal 和 synchronized 的区别

ThreadLocal 和 synchronized 关键字都用于处理多线程并发访问变量的问题，只是二者处理问题的角度和思路不同。

1. ThreadLocal是一个Java类,通过对当前线程中的局部变量的操作来解决不同线程的变量访问的冲突问题。所以，ThreadLocal提供了线程安全的共享对象机制，每个线程都拥有其副本。
2. Java中的synchronized是一个保留字，它依靠JVM的锁机制来实现临界区的函数或者变量的访问中的原子性。在同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量。此时，被用作“锁机制”的变量时多个线程共享的。
3. 同步机制(synchronized关键字)采用了以“时间换空间”的方式，提供一份变量，让不同的线程排队访问。而ThreadLocal采用了“以空间换时间”的方式，为每一个线程都提供一份变量的副本，从而实现同时访问而互不影响。

## 总结：

1. ThreadLocal提供线程内部的局部变量，在本线程内随时随地可取，隔离其他线程。
2. ThreadLocal的设计是：每个Thread维护一个ThreadLocalMap哈希表，这个哈希表的key是ThreadLocal实例本身，value才是真正要存储的值Object。
3. 对ThreadLocal 的常用操作实际是对线程 Thread 中的 ThreadLocalMap 进行操作。
4. ThreadLocalMap 的底层实现是一个定制的自定义哈希表，ThreadLocalMap 的阈值threshold = 底层哈希表 table 的长度 `len * 2 / 3`，当实际存储元素个数 size  大于或等于 阈值 threshold 的 `3/4` 时`size >= threshold*3/4`，则对底层哈希表数组 table 进行扩容操作。
5. ThreadLocalMap 中的哈希表 Entry[] table 存储的核心元素是 Entry ，存储的 key 是 ThreadLocal 实例对象，value 是ThreadLocal 对应储存的值value。需要注意的是，此Entry继承了弱引用 WeakReference，所以在使用ThreadLocalMap时，发现`key == null`，则意味着此key ThreadLocal不在被引用，需要将其从ThreadLocalMap哈希表中移除。
6. ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用来引用它，那么系统 GC 的时候，这个ThreadLocal势必会被回收。所以，在ThreadLocal的get(),set(),remove()的时候都会清除线程ThreadLocalMap里所有key为null的value。如果我们不主动调用上述操作，则会导致内存泄露。
7. 为了安全地使用ThreadLocal，必须要像每次使用完锁就解锁一样，在每次使用完ThreadLocal后都要调用remove() 来清理无用的Entry。这在操作在使用线程池时尤为重要。
8. ThreadLocal 和synchronized的区别：同步机制(synchronized关键字)采用了以“时间换空间”的方式，提供一份变量，让不同的线程排队访问。而ThreadLocal采用了“以空间换时间”的方式，为每一个线程都提供一份变量的副本，从而实现同时访问而互不影响。
9. ThreadLocal主要是解决2种类型的问题：A. 解决并发问题：使用ThreadLocal代替同步机制解决并发问题。B. 解决数据存储问题：如一个Parameter对象的数据需要在多个模块中使用，如果采用参数传递的方式，显然会增加模块之间的耦合性。此时我们可以使用ThreadLocal解决。
10. 每个线程内部都有一个名字为threadLocals的成员变量，该变量类型为HashMap，其中key为我们定义的ThreadLocal变量的this引用，value则为我们set时候的值，每个线程的本地变量是存到到线程自己的内存变量threadLocals里面的，如果当前线程一直不消失那么这些本地变量会一直存到，所以可能会造成内存溢出，所以使用完毕后要记得调用ThreadLocal的remove方法删除对应线程的threadLocals中的本地变量。如果子线程中想要使用父线程中的threadlocal变量该如何做那？敬请期待 `Java中高并发编程必备基础之并发包源码剖析` 一书出版







原文地址：https://juejin.im/entry/5a4c753a6fb9a0451a76c538
