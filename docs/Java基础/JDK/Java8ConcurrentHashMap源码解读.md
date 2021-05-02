---
tags:
	- 基础数据类型
categories: ConcurrentHashMap
title: ConcurrentHashMap 源码解读
---
# Java 8 ConcurrentHashMap 源码解读

 **ConcurrentHashMap** 当之无愧是支持并发最好的键值对（Map）集合。在日常编码中，出场率也相当之高。在jdk8中，集合类 ConcurrentHashMap 经 *Doug Lea* 大师之手，借助volatile语义以及CAS操作进行优化，使得该集合类更好地发挥出了并发的优势。与jdk7中相比，在原有段锁（Segment）的基础上，引入了数组＋链表＋红黑树的存储模型，在查询效率上花费了不少心思。

![mark](https://blogimg.nos-eastchina1.126.net/171214/93aB6f3iC1.png)

## 基础数据结构

ConcurrentHashMap内存存储结构图大致如下：

![mark](https://blogimg.nos-eastchina1.126.net/171214/g3HD7ABdjH.png)

### 概述

1、设计首要目的：维护并发可读性（get、迭代相关）；次要目的：使空间消耗比HashMap相同或更好，且支持多线程高效率的初始插入（empty table）。

2、HashTable线程安全，但采用synchronized，多线程下效率低下。线程1put时，线程2无法put或get。

### 阅前了解

在真正阅读 ConcurrentHashMap 源码之前，我们简单复习下关于volatile和CAS的概念，这样才能更好地帮助我们理解源码中的关键方法。

#### volatile语义

java提供的关键字volatile是最轻量级的同步机制。当定义一个变量为volatile时，它就具备了三层语义： - 可见性（Visibility）：在多线程环境下，一个变量的写操作总是对其后的读取线程可见 - 原子性（Atomicity）：volatile的读/写操作具有原子性 - 有序性（Ordering）：禁止指令的重排序优化，JVM会通过插入内存屏障（Memory Barrier）指令来保证

就同步性能而言，大多数场景下volatile的总开销是要比锁低的。在ConcurrentHashMap的源码中，我们能看到频繁的volatile变量读取与写入。

#### CAS操作

CAS一般被理解为**原子操作**。在java中，正是利用了处理器的CMPXCHG（intel）指令实现CAS操作。CAS需要接受原有期望值expected以及想要修改的新值x，只有在原有期望值与当前值相等时才会更新为x，否则为失败。在ConcurrentHashMap的方法中，大量使用CAS获取/修改互斥量，以达到多线程并发环境下的正确性。


## ConcurrentHashMap 的常量

```java
// maximum_capacity table的最大容量，必须为2次幂形式
private static final int MAXIMUM_CAPACITY = 1 << 30 
```

```java
// default_capacity table的默认初始容量，必须为2次幂形式
private static final int DEFAULT_CAPACITY = 16 
```

```java
// max_array_size MAX_VALUE=2^31-1=2147483647
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8
```

```java
// default_concurrency_leve 未被用到，用来兼容之前版本
private static finalint DEFAULT_CONCURRENCY_LEVEL = 16
```

```java
// load_factor table的负载因子，当前节点数量超过 n * LOAD_FACTOR，执行扩容
// 位操作表达式为 n - (n >>> 2)
private static final float LOAD_FACTOR = 0.75f	
```

```java
// treeify_threshold 针对每个桶（bin），链表转换为红黑树的节点数阈值
static final int TREEIFY_THRESHOLD = 8
```

```java
// 针对每个桶（bin），红黑树退化为链表的节点数阈值
static final int UNTREEIFY_THRESHOLD = 6
```

```java
// min_treeify_capacity 最小的树的容量
static final int MIN_TREEIFY_CAPACITY = 64
```

```java
// 扩容线程每次最少要迁移16个hash桶
// min_transfer_stride 在扩容中，参与的单个线程允许处理的最少table桶首节点个数
// 虽然适当添加线程，会使得整个扩容过程变快，但需要考虑多线程内存同时分配的问题
private static final int MIN_TRANSFER_STRIDE = 16
```

```java
// resize stamp bits sizeCtl 中记录 size 的 bit 数
private static int RESIZE_STAMP_BITS = 16
```

```java
// max_resizers 2^15-1 参与扩容的最大线程数
private static final int MAX_RESIZERS = (1<<(32-RESIZE_STAMP_BITS))-1
```

```java
// 32 - 16 = 16, sizeCtl 中记录 size 大小的偏移量
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS = 16
```

```java
// 转为 nodes的hash值、标示位
static final int MOVED = -1
```

```java
// 树的根节点的 hash 值
static final int TREEBIN = -2 
```

```java
// ReservationNode 的 hash 值
static final int RESERVED = -3
```

```java
// 一些特定的哈希值代表不同含义
static final int MOVED     = -1; // hash for forwarding nodes
static final int TREEBIN   = -2; // hash for roots of trees
static final int RESERVED  = -3; // hash for transient reservations
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
```

```java
// CPU数
static final int NCPU = Runtime.getRuntime().availableProcessors()
```

```java
/**
 * 真正存储Node数据（桶首）节点的数组table
 * 所有Node节点根据hash分桶存储
 * table数组中存储的是所有桶（bin）的首节点
 * hash值相同的节点以链表形式分装在桶中
 * 当一个桶中节点数达到8个时，转换为红黑树，提高查询效率
 * 装载Node的数组，作为ConcurrentHashMap的数据容器，采用懒加载的方式
 * 直到第一次插入数据的时候才会进行初始化操作，数组的大小总是为2的幂次方。
 */
transient volatile Node<K,V>[] table;
// 扩容时候使用,平时为null，只有在扩容的时候才为非null
private transient volatile Node<K,V>[] nextTable;
// 没有竞争条件时，使用
private transient volatile long baseCount;

// 扩容时，将table中的元素迁移至nextTable . 扩容时非空
private transient volatile Node<K,V>[] nextTable;

/**
 *  重要控制变量
 *  根据变量的数值不同，类实例处于不同阶段
 *  1. = -1 : 正在初始化
 *  2. < -1 : 正在扩容，数值为 -(1 + 参与扩容的线程数)
 *  3. = 0  : 创建时初始为0
 *  4. > 0  : 下一次扩容的大小
 */
private transient volatile int sizeCtl;
```



## ConcurrentHashMap 重要属性

### Node

Key-value entry, 继承自Map.Entry<K,V>对象。

Node<K,V>节点是ConcurrentHashMap存储数据的最**基本结构**。一个数据mapping节点中，存储4个变量：当前节点hash值、节点的key值、节点的value值、指向下一个节点的指针next。其中在子类中的hash可以为负数，具有特殊的并发处理意义，后文会解释。除了具有特殊意义的子类，Node中的key和val不允许为null。

```java
static class Node<K,V> implements Map.Entry<K,V> {  
  		// Node节点的hash值和key的hash值相同
  		// TreeNode节点的hash值
        final int hash;  
        final K key;  
        volatile V val; //带有同步锁的value(保证可见性)  
        volatile Node<K,V> next;//带有同步锁的next指针
   
        Node(inthash, K key, V val, Node<K,V> next) {  
            this.hash = hash;  
            this.key = key;  
            this.val = val;  
            this.next = next;  
        }  
   
        public final K getKey()       { return key; }  
        public final V getValue()     { return val; }  
        // HashMap调用Objects.hashCode()，最终也是调用Object.hashCode()；效果一样  
        public final int hashCode()   { returnkey.hashCode() ^ val.hashCode(); }  
        public final String toString(){ returnkey + "=" + val; }  
  		//不允许直接改变value的值
        public final V setValue(V value) { // 不允许修改value值，HashMap允许  
            throw new UnsupportedOperationException();  
        }  
        // HashMap使用if (o == this)，且嵌套if；concurrent使用&&  
        public final boolean equals(Object o) {  
            Object k, v, u; Map.Entry<?,?> e;  
            return ((oinstanceof Map.Entry) &&  
                    (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&  
                    (v = e.getValue()) != null &&  
                    (k == key || k.equals(key)) &&  
                    (v == (u = val) || v.equals(u)));  
        }  
   
        Node<K,V> find(inth, Object k) { // 增加find方法辅助get方法  
            Node<K,V> e = this;  
            if (k != null) {  
                do {  
                    K ek;  
                    if (e.hash == h &&  
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))  
                        return e;  
                  /**
                  *  以链表形式查找桶中下一个Node信息
                  *  当转换为subclass红黑树节点TreeNode
                  *  则使用TreeNode中的find进行查询操作
                  */
                } while ((e = e.next) != null);  
            }  
            returnnull;  
        }  
    }  
```

另外可以看出很多属性都是用volatile进行修饰的，也就是为了保证内存可见性。

1.   这个Node内部类与HashMap中定义的Node类很相似，但是有一些差别  
2.   它对value和next属性设置了volatile同步锁  
3.   它不允许调用setValue方法直接改变Node的value域  
4.   它增加了find方法辅助map.get()方法  

### TreeNode

Node的子类，红黑树节点，当Node链表过长时，会转换成红黑树。

位于 ConcurrentHashMap 类的 2653行 或 搜索 /* ---------------- TreeNodes -------------- */

```java
// Nodes for use in TreeBins，链表>8，才可能转为TreeNode.  
// HashMap的TreeNode继承至LinkedHashMap.Entry；而这里继承至自己实现的Node，将带有next指针，便于treebin访问。  
    static final class TreeNode<K,V> extends Node<K,V> {   
        TreeNode<K,V> parent;  // red-black tree links  
        TreeNode<K,V> left;  
        TreeNode<K,V> right;  
        TreeNode<K,V> prev;    // needed to unlink next upon deletion  
        boolean red;  
   
        TreeNode(inthash, K key, V val, Node<K,V> next,  
                 TreeNode<K,V> parent) {  
            super(hash, key, val, next);  
            this.parent = parent;  
        }  
   
        Node<K,V> find(inth, Object k) {  
            return findTreeNode(h, k, null);  
        }  
   
        /** 
         * Returns the TreeNode (or null if not found) for the given key 
         * starting at given root. 
         */ // 查找hash为h，key为k的节点  
        final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {  
            if (k != null) { // 比HMap增加判空  
                TreeNode<K,V> p = this;  
                do  {  
                    intph, dir; K pk; TreeNode<K,V> q;  
                    TreeNode<K,V> pl = p.left, pr = p.right;  
                    if ((ph = p.hash) > h)  
                        p = pl;  
                    elseif (ph < h)  
                        p = pr;  
                    elseif ((pk = p.key) == k || (pk != null && k.equals(pk)))  
                        returnp;  
                    elseif (pl == null)  
                        p = pr;  
                    elseif (pr == null)  
                        p = pl;  
                    elseif ((kc != null ||  
                              (kc = comparableClassFor(k)) != null) &&  
                             (dir = compareComparables(kc, k, pk)) != 0)  
                        p = (dir < 0) ? pl : pr;  
                    elseif ((q = pr.findTreeNode(h, k, kc)) != null)  
                        returnq;  
                    else  
                        p = pl;  
                } while (p != null);  
            }  
            return null;  
        }  
    }  
// 和HashMap相比，这里的TreeNode相当简洁；ConcurrentHashMap链表转树时，并不会直接转，
// 正如注释（Nodes for use in TreeBins）所说，只是把这些节点包装成TreeNode放到TreeBin中，
// 再由TreeBin来转化红黑树。  
```

树节点，继承于承载数据的Node类。而红黑树的操作是针对TreeBin类的，从该类的注释也可以看出，也就是TreeBin会将TreeNode进行再一次封装

### TreeBin

位于 ConcurrentHashMap 类的 2709 行 或 搜索 /* ---------------- TreeBins -------------- */	

```java
// TreeBin用于封装维护TreeNode，包含putTreeVal、lookRoot、UNlookRoot、remove、
// balanceInsetion、balanceDeletion等方法，这里只分析其构造函数。当链表转树时，
// 用于封装TreeNode，也就是说，ConcurrentHashMap的红黑树存放的是TreeBin，而不是treeNode。  
TreeBin(TreeNode<K,V> b) {  
    super(TREEBIN, null, null, null);//hash值为常量TREEBIN=-2,表示roots of trees  
    this.first = b;  
    TreeNode<K,V> r = null;  
    for (TreeNode<K,V> x = b, next; x != null; x = next) {  
        next = (TreeNode<K,V>)x.next;  
        x.left = x.right = null;  
        if (r == null) {  
            x.parent = null;  
            x.red = false;  
            r = x;  
        }  
        else {  
            K k = x.key;  
            inth = x.hash;  
            Class<?> kc = null;  
            for (TreeNode<K,V> p = r;;) {  
                intdir, ph;  
                K pk = p.key;  
                if ((ph = p.hash) > h)  
                    dir = -1;  
                elseif (ph < h)  
                    dir = 1;  
                elseif ((kc == null &&  
                          (kc = comparableClassFor(k)) == null) ||  
                         (dir = compareComparables(kc, k, pk)) == 0)  
                    dir = tieBreakOrder(k, pk);  
                    TreeNode<K,V> xp = p;  
                if ((p = (dir <= 0) ? p.left : p.right) == null) {  
                    x.parent = xp;  
                    if (dir <= 0)  
                        xp.left = x;  
                    else  
                        xp.right = x;  
                    r = balanceInsertion(r, x);  
                    break;  
                }  
            }  
        }  
    }  
    this.root = r;  
    assert checkInvariants(root);  
} 
```

这个类并不负责包装用户的key、value信息，而是包装的很多TreeNode节点。实际的ConcurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象。

### threeifyBin 

位于 ConcurrentHashMap 类的 2611 行 或 搜索 "private final void treeifyBin"	

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {  
        Node<K,V> b; intn, sc;  
    if (tab != null) { 
      	// 数组的大小还未超过64
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)  
            tryPresize(n << 1); // 容量<64，则table两倍扩容，不转树了  
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {  
            synchronized (b) { // 读写锁  
                if (tabAt(tab, index) == b) {  
                    TreeNode<K,V> hd = null, tl = null;  
                    for (Node<K,V> e = b; e != null; e = e.next) {  
                        TreeNode<K,V> p =  
                            new TreeNode<K,V>(e.hash, e.key, e.val,  
                                              null, null);  
                        if ((p.prev = tl) == null)  
                            hd = p;  
                        else  
                            tl.next = p;  
                        tl = p;  
                    }  
                    setTabAt(tab, index, new TreeBin<K,V>(hd));  
                }  
            }  
        }  
    }  
}  

```

### ForwardingNode

位于 ConcurrentHashMap 类的 2163 行 或 搜索 "static final class ForwardingNode"	

```java
// A node inserted at head of bins during transfer operations.连接两个table  
// 并不是我们传统的包含key-value的节点，只是一个标志节点，并且指向nextTable，提供find方法而已。
// 生命周期：仅存活于扩容操作且bin不为null时，一定会出现在每个bin的首位。  
static final class ForwardingNode<K,V> extends Node<K,V> {  
    final Node<K,V>[] nextTable;  
    ForwardingNode(Node<K,V>[] tab) {  
        super(MOVED, null, null, null); // 此节点hash=-1，key、value、next均为null  
        this.nextTable = tab;  
    }  
   
    Node<K,V> find(int h, Object k) {  
        // 查nextTable节点，outer避免深度递归  
        outer: for (Node<K,V>[] tab = nextTable;;) {  
            Node<K,V> e; intn;  
            if (k == null || tab == null || (n = tab.length) == 0 ||  
                (e = tabAt(tab, (n - 1) & h)) == null)  
                returnnull;  
            for (;;) { // CAS算法多和死循环搭配！直到查到或null  
                int eh; K ek;  
                if ((eh = e.hash) == h &&  
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))  
                    returne;  
                if (eh < 0) {  
                    if (e instanceof ForwardingNode) {  
                        tab = ((ForwardingNode<K,V>)e).nextTable;  
                        continue outer;  
                    }  
                    else  
                        return e.find(h, k);  
                }  
                if ((e = e.next) == null)  
                    return null;  
            }  
        }  
    }  
}  
```

在扩容时才会出现的特殊节点，其key,value,hash全部为null。并拥有nextTable指针引用新的table数组。

### Traverser

```java
    static class Traverser<K,V> {
        Node<K,V>[] tab; 
      	//下一个要访问的entry
        Node<K,V> next;
      	//发现forwardingNode时，保存当前tab相关信息
        TableStack<K,V> stack, spare; 
        //下一个要访问的hash桶索引
        int index;             
      	//当前正在访问的初始tab的hash桶索引
        int baseIndex;          
        //初始tab的hash桶索引边界
        int baseLimit; 
        //初始tab的长度
        final int baseSize; 

        Traverser(Node<K,V>[] tab, int size, int index, int limit) {
            this.tab = tab;
            this.baseSize = size;
            this.baseIndex = this.index = index;
            this.baseLimit = limit;
            this.next = null;
        }

        // 如果有可能，返回下一个有效节点，否则返回null。
        final Node<K,V> advance() {
            Node<K,V> e;
          	//获取Node链表的下一个元素e
            if ((e = next) != null)
                e = e.next;
            for (;;) {
                Node<K,V>[] t; int i, n;  
              	// e不为空，返回e
                if (e != null)
                    return next = e;
              	//e为空，说明此链表已经遍历完成，准备遍历下一个hash桶
                if (baseIndex >= baseLimit || (t = tab) == null ||
                    (n = t.length) <= (i = index) || i < 0)
                  	//到达边界，返回null
                    return next = null;
              	//获取下一个hash桶对应的node链表的头节点
                if ((e = tabAt(t, i)) != null && e.hash < 0) {
                  	//转发节点,说明此hash桶中的节点已经迁移到了nextTable
                    if (e instanceof ForwardingNode) {
                        tab = ((ForwardingNode<K,V>)e).nextTable;
                        e = null;
                      	//保存当前tab的遍历状态
                        pushState(t, i, n);
                        continue;
                    }
                    //红黑树
                    else if (e instanceof TreeBin)
                        e = ((TreeBin<K,V>)e).first;
                    else
                        e = null;
                }
                if (stack != null)
                  	// 此时遍历的是迁移目标nextTable,尝试回退到源table，
                    // 继续遍历源table中的节点
                    recoverState(n);
                else if ((index = i + baseSize) >= n)
                  	//初始tab的hash桶索引+1 ，即遍历下一个hash桶
                    index = ++baseIndex; 
            }
        }

        // 在遇到转发节点时保存遍历状态。
        private void pushState(Node<K,V>[] t, int i, int n) {
            TableStack<K,V> s = spare;  // reuse if possible
            if (s != null)
                spare = s.next;
            else
                s = new TableStack<K,V>();
            s.tab = t;
            s.length = n;
            s.index = i;
            s.next = stack;
            stack = s;
        }

	   // 可能会弹出遍历状态
        private void recoverState(int n) {
            TableStack<K,V> s; int len;
          // (s = stack) != null :stack不空，说明此时遍历的是nextTable
          //  (index += (len = s.length)) >= n: 确保了按照index,
          //index+tab.length的顺序遍历nextTable,条件成立表示nextTable已经遍历完毕
            
            //nextTable中的桶遍历完毕
            while ((s = stack) != null && (index += (len = s.length)) >= n) {
              	//弹出tab，获取tab的遍历状态，开始遍历tab中的桶
                n = len;
                index = s.index;
                tab = s.tab;
                s.tab = null;
                TableStack<K,V> next = s.next;
                s.next = spare; // save for reuse
                stack = next;
                spare = s;
            }
            if (s == null && (index += baseSize) >= n)
                index = ++baseIndex;
        }
    }
```



## tryPresize(扩容)

协调多个线程如何调用transfer方法进行hash桶的迁移（addCount，helpTransfer 方法中也有类似的逻辑）

tryPresize在putAll以及treeifyBin中调用

```java
private final void tryPresize(int size) {  
  	    //计算扩容的目标size
        // 给定的容量若>=MAXIMUM_CAPACITY的一半，直接扩容到允许的最大值，否则调用函数扩容  
        int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :  
            tableSizeFor(size + (size >>> 1) + 1);  
        int sc;  
        while ((sc = sizeCtl) >= 0) { //没有正在初始化或扩容，或者说表还没有被初始化  
            Node<K,V>[] tab = table; int n;  
           //tab没有初始化	
           if(tab == null || (n = tab.length) == 0) {  
                n = (sc > c) ? sc : c; // 扩容阀值取较大者  
         // 期间没有其他线程对表操作，则CAS将SIZECTL状态置为-1，表示正在进行初始化  
             	//初始化之前，CAS设置sizeCtl=-1
                if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {  
                    try {  
                        if (table == tab) {  
                            @SuppressWarnings("unchecked")  
                            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];  
                            table = nt;  
                          	//sc=0.75n,相当于扩容阈值
                            sc = n - (n >>> 2); //无符号右移2位，此即0.75*n  
                        }  
                    } finally {  
                      	// 此时并没有通过CAS赋值，因为其他想要执行初始化的线程，
                        // 发现sizeCtl=-1，就直接返回，从而确保任何情况，
                        // 只会有一个线程执行初始化操作。
                        sizeCtl = sc;
                    }  
                }  
            }// 若欲扩容值不大于原阀值，或现有容量>=最值，什么都不用做了 
          	//目标扩容size小于扩容阈值，或者容量超过最大限制时，不需要扩容
            else if (c <= sc || n >= MAXIMUM_CAPACITY)  
                break;  
          	//扩容
            else if (tab == table) { 
                int rs = resizeStamp(n);  
              	// sc<0表示，已经有其他线程正在扩容
                if (sc < 0) {  
                    Node<K,V>[] nt;//RESIZE_STAMP_SHIFT=16,MAX_RESIZERS=2^15-1 
               // 1. (sc >>> RESIZE_STAMP_SHIFT) != rs ：扩容线程数 > MAX_RESIZERS-1
               // 2. sc == rs + 1 和 sc == rs + MAX_RESIZERS ：表示什么？？？
               // 3. (nt = nextTable) == null ：表示nextTable正在初始化
               // transferIndex <= 0 ：表示所有hash桶均分配出去
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||  
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||  
                        transferIndex <= 0)  
                      	//如果不需要帮其扩容，直接返回
                        break;  
                  	//CAS设置sizeCtl=sizeCtl+1
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) 
                      	//帮其扩容
                        transfer(tab, nt);  
                }  
              	// 第一个执行扩容操作的线程，将sizeCtl设置为：
                // (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2)
                else if (U.compareAndSwapInt(this, SIZECTL, sc,  
                                             (rs << RESIZE_STAMP_SHIFT) + 2))  
                    transfer(tab, null);  
            }  
        }  
    }  
```

```java
private static final int tableSizeFor(int c){//和HashMap一样,返回>=n的最小2的自然数幂  
  int n = c - 1;  
  n |= n >>> 1;  
  n |= n >>> 2;  
  n |= n >>> 4;  
  n |= n >>> 8;  
  n |= n >>> 16;  
  return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;  
}  
```



## spread 重新哈希

spread()重哈希，以减小Hash冲突。我们知道对于一个hash表来说，hash值分散的不够均匀的话会大大增加哈希冲突的概率，从而影响到hash表的性能。因此通过spread方法进行了一次重hash从而大大减小哈希冲突的可能性。spread方法为：

```
static final int spread(int h) {
  return (h ^ (h >>> 16)) & HASH_BITS;
}
```

该方法主要是**将key的hashCode的低16位于高16位进行异或运算**，这样不仅能够使得hash值能够分散能够均匀减小hash冲突的概率，另外另外只用到了异或运算，在性能开销上也能兼顾，做到平衡的trade-off。

## get(查找)

```java
public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
  		 // 1. 重hash
        int h = spread(key.hashCode());
  
  		// 2. table[i]桶节点的key与查找的key相同，则直接返回
        if ((tab = table) != null && (n = tab.length) > 0 &&
            // 唯一一处volatile读操作
            (e = tabAt(tab, (n - 1) & h)) != null) {  
            // 注意：因为容器大小为2的次方，所以 h mod n = h & (n -1)
          
            if ((eh = e.hash) == h) {// 如果hash值相等
              	// 检查第一个Node
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
          	// hash为负表示是扩容中的ForwardingNode节点
            // 直接调用ForwardingNode的find方法(可以是代理到扩容中的nextTable)
          	// 3. 当前节点hash小于0说明为树节点，在红黑树中查找即可
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            // 遍历链表，对比key值
          	// 通过next指针，逐一查找
            while ((e = e.next) != null) {
              	//4. 从链表中查找，查找到则返回该节点的value，否则就返回null即可
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

代码的逻辑请看注释，首先先看当前的hash桶数组节点即table[i]是否为查找的节点，若是则直接返回；若不是，则继续再看当前是不是树节点？通过看节点的hash值是否为小于0，如果小于0则为树节点。如果是树节点在红黑树中查找节点；如果不是树节点，那就只剩下为链表的形式的一种可能性了，就向后遍历查找节点，若查找到则返回节点的value即可，若没有找到就返回null。

这个 get 请求，我们需要 cas 来保证变量的原子性。如果 tab[i] 正被锁住，那么 CAS 就会失败，失败之后就会不断的重试。这也保证了在高并发情况下不会出错。

我们来分析一下哪些情况会导致 get 在并发的情况下可能取不到值。

1. 一个线程在 get 的时候，另一个线程在对同一个 key 的 node 进行 remove 操作
2. 一个线程在 get 的时候，另一个线程正在重排 table 。可能导致旧 table 取不到值

那么本质是，我在get的时候，有其他线程在对同一桶的链表或树进行修改。那么get是怎么保证同步性的呢？我们看到e = tabAt(tab, (n - 1) & h)) != null，在看下tablAt到底是干嘛的：

```
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
```

它是对tab[i]进行原子性的读取，因为我们知道putVal等对table的桶操作是有加锁的，那么一般情况下我们对桶的读也是要加锁的，但是我们这边为什么不需要加锁呢？因为我们用了Unsafe的getObjectVolatile，因为table是volatile类型，所以对tab[i]的原子请求也是可见的。因为如果同步正确的情况下，根据happens-before原则，**对volatile域的写入操作happens-before于每一个后续对同一域的读操作**。所以不管其他线程对table链表或树的修改，都对get读取可见。用一张图说明，协调读-写线程可见示意图：

jdk7是没有用到CAS操作和Unsafe类的，下面是jdk7的get方法：

```
V get(Object key, int hash) { 
            if(count != 0) {       // 首先读 count 变量
                HashEntry<K,V> e = getFirst(hash); 
                while(e != null) { 
                    if(e.hash == hash && key.equals(e.key)) { 
                        V v = e.value; 
                        if(v != null)            
                            return v; 
                        // 如果读到 value 域为 null，说明发生了重排序，加锁后重新读取
                        return readValueUnderLock(e); 
                    } 
                    e = e.next; 
                } 
            } 
            return null; 
        }
```

为什么我们在get的时候需要判断count不等于0呢？如果是在HashMap的源码中是没有这个判断的，不用判断不是也是可以的吗？这个就是用到线程安全发布情况下happens-before原则之volatile变量法则：**对volatile域的写入操作happens-before于每一个后续对同一域的读操作**，看下面的示意图：

## tabAt 

以 volatile 读的方式读取 table 数组中的元素

```java
// 这边为什么i要等于((long)i << ASHIFT) + ABASE呢,计算偏移量
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
  // Key对应的数组元素的可见性，由Unsafe的getObjectVolatile方法保证。
  return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
```

tabAt 方法用来获取table数组中索引为i的Node元素。

## put/putVal 

putVal是将一个新key-value mapping插入到当前ConcurrentHashMap的关键方法。

此方法的具体流程如下图：

![mark](https://blogimg.nos-eastchina1.126.net/171214/CEj7GLmk1G.png)

```java
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        // 不允许 key 和 value 为空
        if (key == null || value == null) throw new NullPointerException();
      	// 1.计算 key 的 hash 值(计算新节点的hash值)
        int hash = spread(key.hashCode()); // 返回 (h^(h>>>16))&HASH_BITS
        int binCount = 0;
      	// 获取当前table，进入死循环,直到插入成功！
        for (Node<K,V>[] tab = table;;) { 
            Node<K,V> f; int n, i, fh;
          	// 2. 如果当前 table 还没初始化先调用 initTable 方法将 tab 进行初始化
            if (tab == null || (n = tab.length) == 0)
                tab = initTable(); // 如果table为空，执行初始化，也即是延迟初始化
          	// 3. tab中索引为i的位置的元素为null,则直接使用 CAS 将值插入即可
          	// 如果bin为空，则采用cas算法赋值，无需加锁
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,new Node<K,V>(hash, key, value, null)))
                  	// 直接设置为桶首节点成功，退出死循环（出口之一）
                    break;              
            }
          	// 4. 当前正在扩容
          	// 当前桶首节点正在特殊的扩容状态下，当前线程尝试参与扩容
            // 然后重新进入死循环
            //f.hash == MOVED 表示为：ForwardingNode，说明其他线程正在扩容
            else if ((fh = f.hash) == MOVED) // MOVED = -1 
                tab = helpTransfer(tab, f); // 当发现其他线程扩容时，帮其扩容
           // 通过桶首节点，将新节点加入table
            else {
                V oldVal = null;
              	// 获取桶首节点实例对象锁，进入临界区进行添加操作
                synchronized (f) {
                  	// 再判断以此f是否仍是第一个Node，如果不是，退出临界区，重复添加操作
                    if (tabAt(tab, i) == f) {
                        //5. 当前为链表，在链表中插入新的键值对
                        if (fh >= 0) { // 桶首节点hash值>0，表示为链表
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                              	// 找到hash值相同的key,覆盖旧值即可
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                  	// 仅 putIfAbsent() 方法中的 onlyIfAbsend 为 true;
                                    if (!onlyIfAbsent)
                                      	// putIfAbsend() 包含 key 则返回 get ,否则 put 并返回
                                        e.val = value; 
                                    break;
                                }
                                Node<K,V> pred = e;
                                //如果到链表末尾仍未找到，则直接将新值插入到链表末尾即可
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                      	// 桶首节点为Node子类型TreeBin，表示为红黑树
                        // 6.当前为红黑树，将新的键值对插入到红黑树中
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                          	// 调用putTreeVal方法，插入新值
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                              	// key已经存在，则替换
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                 // 7.插入完键值对后再根据实际大小看是否需要转换成红黑树
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                      	// 插入新节点后，达到链表转换红黑树阈值，则执行转换操作
                      	// 此函数内部会判断是树化，还是扩容：tryPresize
                        treeifyBin(tab, i);
                  	// 退出死循环（出口之二）
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
      	// 更新计算count时的base和counterCells数组
      	//8.对当前容量大小进行检查，如果超过了临界值（实际大小*加载因子）就需要扩容 
        addCount(1L, binCount);
        return null;
    }
```

当table[i]为链表的头结点，在链表中插入新值在table[i]不为null并且不为forwardingNode时，并且当前Node f的hash值大于0（fh >= 0）的话说明当前节点f为当前桶的所有的节点组成的链表的头结点。那么接下来，要想向ConcurrentHashMap插入新值的话就是向这个链表插入新值。通过synchronized (f)的方式进行加锁以实现线程安全性。往链表中插入节点的部分代码为：

```java
if (fh >= 0) {
    binCount = 1;
    for (Node<K,V> e = f;; ++binCount) {
        K ek;
        // 找到hash值相同的key,覆盖旧值即可
        if (e.hash == hash &&
            ((ek = e.key) == key ||
             (ek != null && key.equals(ek)))) {
            oldVal = e.val;
            if (!onlyIfAbsent)
                e.val = value;
            break;
        }
        Node<K,V> pred = e;
        if ((e = e.next) == null) {
            //如果到链表末尾仍未找到，则直接将新值插入到链表末尾即可
            pred.next = new Node<K,V>(hash, key,
                                      value, null);
            break;
        }
    }
}

```

这部分代码很好理解，就是两种情况：1. 在链表中如果找到了与待插入的键值对的key相同的节点，就直接覆盖即可；2. 如果直到找到了链表的末尾都没有找到的话，就直接将待插入的键值对追加到链表的末尾即可。



当table[i]为红黑树的根节点，在红黑树中插入新值按照之前的数组+链表的设计方案，这里存在一个问题，即使负载因子和Hash算法设计的再合理，也免不了会出现拉链过长的情况，一旦出现拉链过长，甚至在极端情况下，查找一个节点会出现时间复杂度为O(n)的情况，则会严重影响ConcurrentHashMap的性能，于是，在JDK1.8版本中，对数据结构做了进一步的优化，引入了红黑树。而当链表长度太长（默认超过8）时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高ConcurrentHashMap的性能，其中会用到红黑树的插入、删除、查找等算法。当table[i]为红黑树的树节点时的操作为：

```java
if (f instanceof TreeBin) {
    Node<K,V> p;
    binCount = 2;
    if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                   value)) != null) {
        oldVal = p.val;
        if (!onlyIfAbsent)
            p.val = value;
    }
}

```

首先在if中通过`f instanceof TreeBin`判断当前table[i]是否是树节点，这下也正好验证了我们在最上面介绍时说的TreeBin会对TreeNode做进一步封装，对红黑树进行操作的时候针对的是TreeBin而不是TreeNode。这段代码很简单，调用putTreeVal方法完成向红黑树插入新节点，同样的逻辑，**如果在红黑树中存在于待插入键值对的Key相同（hash值相等并且equals方法判断为true）的节点的话，就覆盖旧值，否则就向红黑树追加新节点**。



当table[i]为红黑树的根节点，在红黑树中插入新值。按照之前的数组+链表的设计方案，这里存在一个问题，即使负载因子和Hash算法设计的再合理，也免不了会出现拉链过长的情况，一旦出现拉链过长，甚至在极端情况下，查找一个节点会出现时间复杂度为O(n)的情况，则会严重影响ConcurrentHashMap的性能，于是，在JDK1.8版本中，对数据结构做了进一步的优化，引入了红黑树。而当链表长度太长（默认超过8）时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高ConcurrentHashMap的性能，其中会用到红黑树的插入、删除、查找等算法。当table[i]为红黑树的树节点时的操作为：

```java
if (f instanceof TreeBin) {
 Node<K,V> p;
 binCount = 2;
 if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,value)) != null) {
 			oldVal = p.val;
            if (!onlyIfAbsent)
            p.val = value;
    }
}
```

首先在if中通过`f instanceof TreeBin`判断当前table[i]是否是树节点，这下也正好验证了我们在最上面介绍时说的TreeBin会对TreeNode做进一步封装，对红黑树进行操作的时候针对的是TreeBin而不是TreeNode。这段代码很简单，调用putTreeVal方法完成向红黑树插入新节点，同样的逻辑，**如果在红黑树中存在于待插入键值对的Key相同（hash值相等并且equals方法判断为true）的节点的话，就覆盖旧值，否则就向红黑树追加新节点**。



根据当前节点个数进行调整当完成数据新节点插入之后，会进一步对当前链表大小进行调整，这部分代码为：

```
if (binCount != 0) {
    if (binCount >= TREEIFY_THRESHOLD)
        treeifyBin(tab, i);
    if (oldVal != null)
        return oldVal;
    break;
}

```

很容易理解，如果当前链表节点个数大于等于8（TREEIFY_THRESHOLD）的时候，就会调用treeifyBin方法将tabel[i]（第i个散列桶）拉链转换成红黑树。

**关于Put方法的逻辑就基本说的差不多了，现在来做一些总结：**

**整体流程：**

1. 首先对于每一个放入的值，首先利用spread方法对key的hashcode进行一次hash计算，由此来确定这个值在 table中的位置；
2. 如果当前table数组还未初始化，先将table数组进行初始化操作；
3. 如果这个位置是null的，那么使用CAS操作直接放入；
4. 如果这个位置存在结点，说明发生了hash碰撞，首先判断这个节点的类型。如果该节点fh==MOVED(代表forwardingNode,数组正在进行扩容)的话，说明正在进行扩容；
5. 如果是链表节点（fh>0）,则得到的结点就是hash值相同的节点组成的链表的头节点。需要依次向后遍历确定这个新加入的值所在位置。如果遇到hash值与key值都与新加入节点是一致的情况，则只需要更新value值即可。否则依次向后遍历，直到链表尾插入这个结点；
6. 如果这个节点的类型是TreeBin的话，直接调用红黑树的插入方法进行插入新的节点；
7. 插入完节点之后再次检查链表长度，如果长度大于8，就把这个链表转换成红黑树；
8. 对当前容量大小进行检查，如果超过了临界值（实际大小*加载因子）就需要扩容。

**该流程中，可以细细品味的环节有： - 初始化方法 initTable - 扩容方法 transfer (在多线程扩容方法 helpTransfer 中被调用)**

## initTable

initTable方法允许多线程同时进入，但只有一个线程可以完成table的初始化，其他线程都会通过yield方法让出cpu。

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
      	// 前文提及sizeCtl是重要的控制变量
        // sizeCtl = -1 表示正在初始化
        if ((sc = sizeCtl) < 0)
            // 已经有其他线程在执行初始化，则主动让出cpu
            // 1. 保证只有一个线程正在进行初始化操作
            Thread.yield();
      
      	// 利用CAS操作设置sizeCtl为-1
        // 设置成功表示当前线程为执行初始化的唯一线程
        // 此处进入临界区
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
              	// 由于让出cpu的线程也会后续进入该临界区
                // 需要进行再次确认table是否为null
                if ((tab = table) == null || tab.length == 0) {
                    // 2. 得出数组的大小
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    // 3. 这里才真正的初始化数组，即分配Node数组
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                  	// 默认负载为0.75
                    // 4. 计算数组中可用的大小：实际大小n*0.75（加载因子）
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
          	// 退出死循环的唯一出口
            break;
        }
    }
    return tab;
}
```

代码的逻辑请见注释，有可能存在一个情况是多个线程同时走到这个方法中，为了保证能够正确初始化，在第1步中会先通过if进行判断，**若当前已经有一个线程正在初始化即sizeCtl值变为-1**，这个时候其他线程在If判断为true从而调用Thread.yield()让出CPU时间片。正在进行初始化的线程会调用U.compareAndSwapInt方法将sizeCtl改为-1即正在初始化的状态。另外还需要注意的事情是，在第四步中会进一步计算数组中可用的大小即为数组实际大小n乘以加载因子0.75.可以看看这里乘以0.75是怎么算的，0.75为四分之三，这里`n - (n >>> 2)`是不是刚好是n-(1/4)n=(3/4)n，挺有意思的吧:)。如果选择是无参的构造器的话，这里在new Node数组的时候会使用默认大小为`DEFAULT_CAPACITY`（16），然后乘以加载因子0.75为12，也就是说数组的可用大小为12。

## casTabAt(原子操作方法)

以 CAS 的方式，将元素插入到 table 数组

```java
  /*
    *这边为什么i要等于((long)i << ASHIFT) + ABASE呢,计算偏移量
    *ASHIFT是指tab[i]中第i个元素在相对于数组第一个元素的偏移量，而ABASE就算第一数组的内存素的偏移地址
    *所以呢，((long)i << ASHIFT) + ABASE就算i最后的地址
    * 那么compareAndSwapObject的作用就算tab[i]和c比较，如果相等就tab[i]=v否则tab[i]=c;
    */
    // 利用CAS算法设置i位置上的Node节点（将c和table[i]比较，相同则插入v）。  
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,  
                                        Node<K,V> c, Node<K,V> v) {  
      	//原子的执行如下逻辑：如果tab[i]==c,则设置tab[i]=v，并返回ture.否则返回false
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);  
    }
```

利用CAS操作设置table数组中索引为i的元素

## setTabAt	

以 valatile 写的方式，将元素插入 table 数组

```
 static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
     U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
 }
 
```

该方法用来设置table数组中索引为i的元素

## 实例构造器方法

在使用ConcurrentHashMap第一件事自然而然就是new 出来一个ConcurrentHashMap对象，一共提供了如下几个构造器方法：

```java
// 1. 构造一个空的map，即table数组还未初始化，初始化放在第一次插入数据时，默认大小为16
ConcurrentHashMap()
// 2. 给定map的大小
ConcurrentHashMap(int initialCapacity) 
// 3. 给定一个map
ConcurrentHashMap(Map<? extends K, ? extends V> m)
// 4. 给定map的大小以及加载因子
ConcurrentHashMap(int initialCapacity, float loadFactor)
// 5. 给定map大小，加载因子以及并发度（预计同时操作数据的线程）
ConcurrentHashMap(int initialCapacity,float loadFactor, int concurrencyLevel)

```

ConcurrentHashMap一共给我们提供了5中构造器方法，具体使用请看注释，我们来看看第2种构造器，传入指定大小时的情况，该构造器源码为：

```java
public ConcurrentHashMap(int initialCapacity) {
    //1. 小于0直接抛异常
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    //2. 判断是否超过了允许的最大值，超过了话则取最大值，否则再对该值进一步处理
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    //3. 赋值给sizeCtl
    this.sizeCtl = cap;
}

```

这段代码的逻辑请看注释，很容易理解，如果小于0就直接抛出异常，如果指定值大于了所允许的最大值的话就取最大值，否则，在对指定值做进一步处理。最后将cap赋值给sizeCtl,关于sizeCtl的说明请看上面的说明，**当调用构造器方法之后，sizeCtl的大小应该就代表了ConcurrentHashMap的大小，即table数组长度**。tableSizeFor做了哪些事情了？源码为：

## tableSizeFor

```

private static final int tableSizeFor(int c) {
    int n = c - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

```

通过注释就很清楚了，该方法会将调用构造器方法时指定的大小转换成一个2的幂次方数，也就是说ConcurrentHashMap的大小一定是2的幂次方，比如，当指定大小为18时，为了满足2的幂次方特性，实际上concurrentHashMapd的大小为2的5次方（32）。另外，需要注意的是，**调用构造器方法的时候并未构造出table数组（可以理解为ConcurrentHashMap的数据容器），只是算出table数组的长度，当第一次向ConcurrentHashMap插入数据的时候才真正的完成初始化创建table数组的工作**。

## helpTransfer(协助扩容)

```java
// 协助扩容方法。多线程下，当前线程检测到其他线程正进行扩容操作，则协助其一起扩容；（只有这种情况会被调用）从某种程度上说，其“优先级”很高，只要检测到扩容，就会放下其他工作，先扩容。  
// 调用之前，nextTable一定已存在。  
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {  
    Node<K,V>[] nextTab; intsc;  
    if (tab != null && (finstanceof ForwardingNode) &&  
        (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {  
        intrs = resizeStamp(tab.length); //标志位  
        while (nextTab == nextTable && table == tab &&  
               (sc = sizeCtl) < 0) {  
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||  
                sc == rs + MAX_RESIZERS || transferIndex <= 0)  
                break;  
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {  
                transfer(tab, nextTab);//调用扩容方法，直接进入复制阶段  
                break;  
            }  
        }  
        return nextTab;  
    }  
    return table;  
}  
```

## addCount

在put方法结尾处调用了addCount方法，把当前ConcurrentHashMap的元素个数+1这个方法一共做了两件事,更新baseCount的值，检测是否进行扩容。

```java
   private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;
     
     	//利用CAS方法更新baseCount的值
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {// 1
            CounterCell a; long v; int m;
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
              	// 多线程 CAS 发生失败的时候执行
                fullAddCount(x, uncontended); // 2
                return;
            }
            if (check <= 1)
                return;
            s = sumCount();
        }
     	//如果check值大于等于0 则需要检验是否需要进行扩容操作
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
          	// 当条件满足的时候开始扩容
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                int rs = resizeStamp(n);
              	// 如果小于0 说明已经有线程在进行扩容了
                if (sc < 0) {
        // 一下的情况说明已经有在扩容或者多线程进行了扩容，其他线程直接 break 不要进入扩容
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                  	// 如果已经有其他线程在执行扩容操作
                  	// 如果相等说明已经完成，可以继续扩容
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
              	// 当前线程是唯一的或是第一个发起扩容的线程  此时nextTable=null
       			// 这个时候 sizeCtl 已经等于(rs<<RESIZE_STAMP_SHIFT)+2 等于一个大的负数，这边
              	// 加上2很巧，因为 transfer 后面对 sizeCtl-- 操作的时候，最多只能减两个就结束
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }
```

看上面的注释1,每次都会对 baseCount 加1,如果并发竞争太大，那么可能导致 U.compareAndSwapLong(this,BASECOUNT,b=baseCount,s = b + x) 失败,那么为了提高高并发的时候 baseCount 可见性的失败的问题,又避免一直重试，这样性能会有很大的影响,那么在 jdk 8的时候是有引入一个类 Striped64 ,其中 LongAdder 和 DoubleAdder 就是对这个类的实现。这两个方法都是为了解决高并发场景而生的，是 AtomicLong 的加强版,AtomicLong 在高并发场景性能会比 LongAdder 差。但是 LongAdder 的空间复杂度会高点。

## fullAddCount

```java
   private final void fullAddCount(long x, boolean wasUncontended) {
        int h;
        // 获取当前线程的 probe 值作为 hash 值,如果0则强制初始化当前线程的 Probe 值，
     	// 初始化 probe 值不为 0
        if ((h = ThreadLocalRandom.getProbe()) == 0) {
            ThreadLocalRandom.localInit();      // force initialization
            h = ThreadLocalRandom.getProbe();
          	// 设置未竞争标记为true
            wasUncontended = true;
        }
        boolean collide = false;                // True if last slot nonempty
        for (;;) {
            CounterCell[] as; CounterCell a; int n; long v;
            if ((as = counterCells) != null && (n = as.length) > 0) {
                if ((a = as[(n - 1) & h]) == null) {
                  	// Try to attach new Cell 如果当前没有 CounterCell 就创建一个
                    if (cellsBusy == 0) {            
                        CounterCell r = new CounterCell(x); // Optimistic create
                        if (cellsBusy == 0 &&
                            // 这边加上 cellsBusy 锁  
                            U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                            boolean created = false;
                            try {               // Recheck under lock
                                CounterCell[] rs; int m, j;
                                if ((rs = counterCells) != null &&
                                    (m = rs.length) > 0 &&
                                    rs[j = (m - 1) & h] == null) {
                                    rs[j] = r;
                                    created = true;
                                }
                            } finally {
                                // 释放 cellsBusy 锁定，让其他线程可以进来
                                cellsBusy = 0; 
                            }
                            if (created)
                                break;
                            continue;           // Slot is now non-empty
                        }
                    }
                    collide = false;
                }
              	// wasUncontended 为 false 说明已经发生了竞争，重置为true重新执行上面代码
                else if (!wasUncontended)       // CAS already known to fail
                    wasUncontended = true;      // Continue after rehash
              	// 对 cell 的值进行累计x(1)
                else if (U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))
                    break;
              	// 表明 as 已经过时，说明 cells 已经初始化完成，看下面，
              	// 重置 collide 为 false 表明已经存在竞争
                else if (counterCells != as || n >= NCPU)
                    collide = false;            // At max size or stale
                else if (!collide)
                    collide = true;
                else if (cellsBusy == 0 &&
                         U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                    try {
                      	// 下面的方法主要是给 counterCells 扩容，尽可能避免冲突
                        if (counterCells == as) {// Expand table unless stale
                            CounterCell[] rs = new CounterCell[n << 1];
                            for (int i = 0; i < n; ++i)
                                rs[i] = as[i];
                            counterCells = rs;
                        }
                    } finally {
                        cellsBusy = 0;
                    }
                    collide = false;
                    continue;                   // Retry with expanded table
                }
                h = ThreadLocalRandom.advanceProbe(h);
            }
          	// 表明 counterCells 还没初始化，则初始化，这边用 cellsBusy 加锁
            else if (cellsBusy == 0 && counterCells == as &&
                     U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                boolean init = false;
                try {                           // Initialize table
                    if (counterCells == as) {
                        CounterCell[] rs = new CounterCell[2];
                        rs[h & 1] = new CounterCell(x);
                        counterCells = rs;
                        init = true;
                    }
                } finally {
                    cellsBusy = 0;
                }
                if (init)
                    break;
            }
          	// 最终如果上面的都失败就把 x 累计到 baseCount
            else if (U.compareAndSwapLong(this, BASECOUNT, v = baseCount, v + x))
                break;                          // Fall back on using base
        }
    }

```

回到 addCount 来,我们每次竞争都对 baseCount 进行加 1 当达到一定的容量时，就需要对 table 进行扩容。 使用 transfer 方法。

## transfer 

负责迁移node节点

扩容transfer方法是一个设计极为精巧的方法。通过互斥读写ForwardingNode，多线程可以协同完成扩容任务。

![mark](https://blogimg.nos-eastchina1.126.net/171214/k7LKAjJm0m.png)

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
  		//计算每次迁移的node个数（MIN_TRANSFER_STRIDE该值作为下限，以避免扩容线程过多）
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
          	// 确保每次迁移的node个数不少于16个
            stride = MIN_TRANSFER_STRIDE; 
  		// nextTab为扩容中的临时table
        if (nextTab == null) {
            try {
              	//扩容一倍	
                @SuppressWarnings("unchecked")
              	// 1. 新建一个 node 数组，容量为之前的两倍
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
          	// transferIndex为扩容复制过程中的桶首节点遍历索引
            // 所以从n开始，表示从后向前遍历
            transferIndex = n;
        }
        int nextn = nextTab.length;
  		// ForwardingNode是Node节点的直接子类，是扩容过程中的特殊桶首节点
      	// 该类中没有key,value,next
      	// hash值为特定的-1
        // 附加Node<K,V>[] nextTable变量指向扩容中的nextTab
        // 在find方法中，将扩容中的查询操作导入到nextTab上
  		//2. 新建forwardingNode引用，在之后会用到
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
  		// 循环的关键变量，判断是否已经扩容完成，完成就 return , 退出循环
        boolean finishing = false; 
  		 //【1】逆序迁移已经获取到的hash桶集合，如果迁移完毕，则更新transferIndex，
         // 获取下一批待迁移的hash桶
         //【2】如果transferIndex=0，表示所以hash桶均被分配，将i置为-1，
  		// 准备退出transfer方法
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
          	// 3. 确定遍历中的索引i（更新待迁移的hash桶索引）
          	// 循环的关键 i , i-- 操作保证了倒叙遍历数组
            while (advance) {
                int nextIndex, nextBound;
              	// 更新迁移索引i
                if (--i >= bound || finishing)
                    advance = false;
              	// transferIndex = 0表示table中所有数组元素都已经有其他线程负责扩容
              	// nextIndex=transferIndex=n=tab.length(默认16)
                else if ((nextIndex = transferIndex) <= 0) {
                  	// transferIndex<=0表示已经没有需要迁移的hash桶，
                  	// 将i置为-1，线程准备退出
                    i = -1;
                    advance = false;
                }
             //cas无锁算法设置 transferIndex = transferIndex - stride		
             // 尝试更新transferIndex，获取当前线程执行扩容复制的索引区间
             // 更新成功，则当前线程负责完成索引为(nextBound，nextIndex)之间的桶首节点扩容
             //当迁移完bound这个桶后，尝试更新transferIndex，获取下一批待迁移的hash桶
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            } //退出transfer
          	//4.将原数组中的元素复制到新数组中去
            //4.5 for循环退出，扩容结束修改sizeCtl属性
// i<0 说明已经遍历完旧的数组tab;i>=n什么时候有可能呢？在下面看到i=n,所以目前i最大应该是n吧
// i+n>=nextn,nextn=nextTab.length,所以如果满足i+n>=nextn说明已经扩容完成
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {   // a
                  	//最后一个迁移的线程，recheck后，做收尾工作，然后退出
                    nextTable = null;
                    table = nextTab;
                  	// 扩容成功，设置新sizeCtl，仍然为总大小的0.75
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
			
                // 第一个扩容的线程，执行transfer方法之前，会设置 sizeCtl = 
                // (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2) 	
                // 后续帮其扩容的线程，执行transfer方法之前，会设置 sizeCtl = sizeCtl+1
                // 每一个退出transfer的方法的线程，退出之前，会设置 sizeCtl = sizeCtl-1
                // 那么最后一个线程退出时：
                // 必然有sc == (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2)，
                // 即 (sc - 2) == resizeStamp(n) << RESIZE_STAMP_SHIFT
              
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {                  
                  	// 如果有多个线程进行扩容，那么这个值在第二个线程以后就不会相等，因为 
                  	// sizeCtl 已经被减1了，所以后面的线程只能直接返回，
                  	// 始终保证只有一个线程执行了a(上面的注释a)
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                  	// finishing 和 advance 保证线程已经扩容完成了可以退出循环
                    finishing = advance = true;
                  	//最后退出的线程要重新check下是否全部迁移完毕
                    i = n;
                }
            }
          	// 当前table节点为空，不需要复制，直接放入ForwardingNode
          	//4.1 当前数组中第i个元素为null，用CAS设置成特殊节点forwardingNode(可以理解成占位符)
          	// 如果 tab[i] 为 null,那么就把 fwd 插入到 tab[i],表明这个节点已经处理过了
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
          	// 当前table节点已经是ForwardingNode
            // 表示已经被其他线程处理了，则直接往前遍历
            // 通过CAS读写ForwardingNode节点状态，达到多线程互斥处理
          	// 4.2 如果遍历到ForwardingNode节点说明这个点已经被处理过了直接跳过
            // 这里是控制并发扩容的核心
          	// 如果 f.hash=-1 的话说明该节点为 ForwardingNode,说明该节点已经处理过了
            else if ((fh = f.hash) == MOVED)
                advance = true; 
          	//迁移node节点
            else {
              	// 锁住当前桶首节点
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                      	// 链表节点复制(链表迁移)
                        if (fh >= 0) {
                        // 4.3 处理当前节点为链表的头结点的情况，构造两个链表，一个是原链表  
                        // 另一个是原链表的反序排列
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                //将node链表，分成2个新的node链表
                // 这边还对链表进行遍历，这边的算法和hashMap的算法又不一样了，对半拆分
                // 把链表拆分为，hash&n 等于0和不等于0的，然后分别放在新表的i和i+n位置           	
                // 此方法同 HashMap 的 resize
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                          	//将新node链表赋给nextTab
                          	//在nextTable的i位置上插入一个链表
                            setTabAt(nextTab, i, ln);
                            //在nextTable的i+n的位置上插入另一个链表
                            setTabAt(nextTab, i + n, hn);
                          	// 扩容成功后，设置ForwardingNode节点
                          	//在table的i位置上插入forwardNode节点表示已经处理过该节点
                          	// 把已经替换的节点的旧tab的i的位置用fwd替换，fwd包含nextTab
                            setTabAt(tab, i, fwd);
                            //设置advance为true 返回到上面的while循环中 就可以执行i--操作
                            advance = true;
                        }
                      	// 红黑树节点复制(红黑树迁移)
                      	//4.4 处理当前节点是TreeBin时的情况，操作和上面的类似
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                          	// 判断扩容后是否还需要红黑树
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                          	// 扩容成功后，设置ForwardingNode节点
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```

代码逻辑请看注释,整个扩容操作分为**两个部分**：

**第一部分**是构建一个nextTable,它的容量是原来的两倍，这个操作是单线程完成的。新建table数组的代码为:`Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1]`,在原容量大小的基础上右移一位。

**第二个部分**就是将原来table中的元素复制到nextTable中，主要是遍历复制的过程。
根据运算得到当前遍历的数组的位置i，然后利用tabAt方法获得i位置的元素再进行判断：

1. 如果这个位置为空，就在原table中的i位置放入forwardNode节点，这个也是触发并发扩容的关键点；
2. 如果这个位置是Node节点（fh>=0），如果它是一个链表的头节点，就构造一个反序链表，把他们分别放在nextTable的i和i+n的位置上
3. 如果这个位置是TreeBin节点（fh<0），也做一个反序处理，并且判断是否需要untreefi，把处理的结果分别放在nextTable的i和i+n的位置上
4. 遍历过所有的节点以后就完成了复制工作，这时让nextTable作为新的table，并且更新sizeCtl为新容量的0.75倍 ，完成扩容。设置为新容量的0.75倍代码为 `sizeCtl = (n << 1) - (n >>> 1)`，仔细体会下是不是很巧妙，n<<1相当于n右移一位表示n的两倍即2n,n>>>1左右一位相当于n除以2即0.5n,然后两者相减为2n-0.5n=1.5n,是不是刚好等于新容量的0.75倍即2n*0.75=1.5n。最后用一个示意图来进行总结（图片摘自网络）：

## mappingCount 与 size

**mappingCount**与**size**方法的类似 从给出的注释来看，应该使用mappingCount代替size方法 两个方法都没有直接返回basecount 而是统计一次这个值，而这个值其实也是一个大概的数值，因此可能在统计的时候有其他线程正在执行插入或删除操作。

```java
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
            (int)n);
}
 /**
 * Returns the number of mappings. This method should be used
 * instead of {@link #size} because a ConcurrentHashMap may
 * contain more mappings than can be represented as an int. The
 * value returned is an estimate; the actual count may differ if
 * there are concurrent insertions or removals.
 *
 * @return the number of mappings
 * @since 1.8
 */
public long mappingCount() {
    long n = sumCount();
    return (n < 0L) ? 0L : n; // ignore transient negative values
}

 final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;//所有counter的值求和
        }
    }
    return sum;
}

```

## remove

**和put方法一样，多个remove线程请求不同的hash桶时，可以并发执行**

![img](http://upload-images.jianshu.io/upload_images/6283837-95df888f4f738601.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

如图所示：删除的node节点的next依然指着下一个元素。此时若有一个遍历线程正在遍历这个已经删除的节点，这个遍历线程依然可以通过next属性访问下一个元素。从遍历线程的角度看，他并没有感知到此节点已经删除了，这说明了ConcurrentHashMap提供了弱一致性的迭代器。

```java
 	public V remove(Object key) {
        return replaceNode(key, null, null);
    }
    
      // 当参数 value == null 时，删除节点。否则更新节点的值为value
      // cv 是个期望值，当 map[key].value 等于期望值 cv 或 cv == null 时，
      // 删除节点，或者更新节点的值
      final V replaceNode(Object key, V value, Object cv) {
        int hash = spread(key.hashCode());
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
          	// table 还没初始化或key对应的 hash 桶为空
            if (tab == null || (n = tab.length) == 0 ||
                (f = tabAt(tab, i = (n - 1) & hash)) == null)
                break;
          	// 正在扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                boolean validated = false;
                synchronized (f) {
                  	// CAS 获取 tab[i] ,如果此时 tab[i] != f,说明其他线程修改了 tab[i]
                    // 回到 for 循环开始处，重新执行
                    if (tabAt(tab, i) == f) {
                      	// node 链表
                        if (fh >= 0) {
                            validated = true;
                            for (Node<K,V> e = f, pred = null;;) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    V ev = e.val;
                                  	// ev 代表参数期望值
                                  	// cv == null:直接更新value/删除节点
                                  	// cv 不为空，则只有在 key 的 oldVal 等于
                                  	// 期望值的时候，才更新 value/删除节点
                                    if (cv == null || cv == ev ||
                                        (ev != null && cv.equals(ev))) {
                                        oldVal = ev;
                                      	//更新value
                                        if (value != null)
                                            e.val = value;
                                      	//删除非头节点
                                        else if (pred != null)
                                            pred.next = e.next;
                                      	//删除头节点
                                        else
                                          	// 因为已经获取了头结点锁，所以此时
                                          	// 不需要使用casTabAt
                                            setTabAt(tab, i, e.next);
                                    }
                                    break;
                                }
                              	//当前节点不是目标节点，继续遍历下一个节点
                                pred = e;
                                if ((e = e.next) == null)
                                  	//到达链表尾部，依旧没有找到，跳出循环
                                    break;
                            }
                        }
                      	//红黑树
                        else if (f instanceof TreeBin) {
                            validated = true;
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> r, p;
                            if ((r = t.root) != null &&
                                (p = r.findTreeNode(hash, key, null)) != null) {
                                V pv = p.val;
                                if (cv == null || cv == pv ||
                                    (pv != null && cv.equals(pv))) {
                                    oldVal = pv;
                                    if (value != null)
                                        p.val = value;
                                    else if (t.removeTreeNode(p))
                                        setTabAt(tab, i, untreeify(t.first));
                                }
                            }
                        }
                    }
                }
                if (validated) {
                    if (oldVal != null) {
                      	//如果删除了节点，更新size
                        if (value == null)
                            addCount(-1L, -1);
                        return oldVal;
                    }
                    break;
                }
            }
        }
        return null;
    }

```



## ForwardingNode

```java
    static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ForwardingNode(Node<K,V>[] tab) {
          	//hash值为MOVED（-1）的节点就是ForwardingNode
            super(MOVED, null, null, null);
            this.nextTable = tab;
        }

      	//通过此方法，访问被迁移到nextTable中的数据
        Node<K,V> find(int h, Object k) {
            // loop to avoid arbitrarily deep recursion on forwarding nodes
            outer: for (Node<K,V>[] tab = nextTable;;) {
                Node<K,V> e; int n;
                if (k == null || tab == null || (n = tab.length) == 0 ||
                    (e = tabAt(tab, (n - 1) & h)) == null)
                    return null;
                for (;;) {
                    int eh; K ek;
                    if ((eh = e.hash) == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                    if (eh < 0) {
                        if (e instanceof ForwardingNode) {
                            tab = ((ForwardingNode<K,V>)e).nextTable;
                            continue outer;
                        }
                        else
                            return e.find(h, k);
                    }
                    if ((e = e.next) == null)
                        return null;
                }
            }
        }
    }
```



# 总结

JDK6,7中的ConcurrentHashmap主要使用Segment来实现减小锁粒度，分割成若干个Segment，在put的时候需要锁住Segment，get时候不加锁，使用volatile来保证可见性，当要统计全局时（比如size），首先会尝试多次计算modcount来确定，这几次尝试中，是否有其他线程进行了修改操作，如果没有，则直接返回size。如果有，则需要依次锁住所有的Segment来计算。

而在1.8的时候摒弃了segment臃肿的设计，这种设计在定位到具体的桶时，要先定位到具体的segment，然后再
在segment中定位到具体的桶。而到了1.8的时候是针对的是Node[] tale数组中的每一个桶，进一步减小了锁粒度。并且防止拉链过长导致性能下降，当链表长度大于8的时候采用红黑树的设计。

主要设计上的变化有以下几点:

1. 不采用segment而采用node，锁住node来实现减小锁粒度。
2. 设计了MOVED状态 当resize的中过程中 线程2还在put数据，线程2会帮助resize。
3. 使用3个CAS操作来确保node的一些操作的原子性，这种方式代替了锁。
4. sizeCtl的不同值来代表不同含义，起到了控制的作用。
5. 采用synchronized而不是ReentrantLock
6. volatile语义提供更细颗粒度的轻量级锁，使得多线程可以(几乎)同时读写实例中的关键量，正确理解当前类所处的状态，进入对应if语句中执行相关逻辑。
7. 采用更加细粒度的hash桶级别锁，扩容期间，依然可以保证写操作的并发度。
8. 多线程无锁扩容的关键就是通过CAS设置sizeCtl与transferIndex变量，协调多个线程对table数组中的node进行迁移。

参考文章：http://www.cnblogs.com/huaizuo/p/5413069.html

参考文章：http://www.bijishequ.com/detail/560964?p=

参考文章：https://bentang.me/tech/2016/12/01/jdk8-concurrenthashmap-1/

参考文章:   http://www.jianshu.com/p/5bc70d9e5410

扩容原理:   http://www.jianshu.com/p/487d00afe6ca

遍历操作：http://www.jianshu.com/p/3e85ac8f8662

改进说明带例子:http://www.voidcn.com/article/p-gdbewnlb-qh.html

http://nannan408.iteye.com/blog/2217042
