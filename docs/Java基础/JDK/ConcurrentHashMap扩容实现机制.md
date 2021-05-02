# ConcurrentHashMap扩容实现机制

jdk8中，采用多线程扩容。整个扩容过程，通过CAS设置sizeCtl，transferIndex等变量协调多个线程进行**并发扩容**。

# 扩容相关的属性

## nextTable

扩容期间，将table数组中的元素 迁移到 nextTable。

```
    /**
     * The next table to use; non-null only while resizing.
       扩容时，将table中的元素迁移至nextTable . 扩容时非空
     */
    private transient volatile Node<K,V>[] nextTable;



```

## sizeCtl属性

```
    private transient volatile int sizeCtl;
   

```

**多线程之间，以volatile的方式读取sizeCtl属性，来判断ConcurrentHashMap当前所处的状态。通过cas设置sizeCtl属性，告知其他线程ConcurrentHashMap的状态变更**。

不同状态，sizeCtl所代表的含义也有所不同。

- 未初始化：
  - sizeCtl=0：表示没有指定初始容量。
  - sizeCtl>0：表示初始容量。


- 初始化中：
  - sizeCtl=-1,标记作用，告知其他线程，正在初始化
- 正常状态：
  - sizeCtl=0.75n ,扩容阈值
- 扩容中:
  - sizeCtl < 0 : 表示有其他线程正在执行扩容
  - sizeCtl = (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2 :表示此时只有一个线程在执行扩容

ConcurrentHashMap的状态图如下：

![mark](https://blogimg.nos-eastchina1.126.net/171214/2ifidi9IcG.png)

## transferIndex属性

```
    private transient volatile int transferIndex;
    
    
     /**
      扩容线程每次最少要迁移16个hash桶
     */
    private static final int MIN_TRANSFER_STRIDE = 16;

```

**扩容索引，表示已经分配给扩容线程的table数组索引位置。主要用来协调多个线程，并发安全地获取迁移任务（hash桶）。**

1 在扩容之前，transferIndex 在数组的最右边 。此时有一个线程发现已经到达扩容阈值，准备开始扩容。

![mark](https://blogimg.nos-eastchina1.126.net/171214/8bdb3048Bk.png)

2 扩容线程，在迁移数据之前，首先要将transferIndex右移（以cas的方式修改 **transferIndex=transferIndex-stride(要迁移hash桶的个数)**），获取迁移任务。每个扩容线程都会通过for循环+CAS的方式设置transferIndex，因此可以确保多线程扩容的并发安全。

![mark](https://blogimg.nos-eastchina1.126.net/171214/kjgeEBH1Cj.png)

换个角度，我们可以将待迁移的table数组，看成一个任务队列，transferIndex看成任务队列的头指针。而扩容线程，就是这个队列的消费者。扩容线程通过CAS设置transferIndex索引的过程，就是消费者从任务队列中获取任务的过程。为了性能考虑，我们当然不会每次只获取一个任务（hash桶），因此ConcurrentHashMap规定，每次至少要获取16个迁移任务（迁移16个hash桶，MIN_TRANSFER_STRIDE = 16）

cas设置transferIndex的源码如下：

```
  private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        //计算每次迁移的node个数
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // 确保每次迁移的node个数不少于16个
        ...
        for (int i = 0, bound = 0;;) {
            ...
            //cas无锁算法设置 transferIndex = transferIndex - stride
            if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                  ...
                  ...
            }
            ...//省略迁移逻辑
        }
    }


```

## ForwardingNode节点

1. 标记作用，表示其他线程正在扩容，并且此节点已经扩容完毕
2. 关联了nextTable,扩容期间可以通过find方法，访问已经迁移到了nextTable中的数据

```
     static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ForwardingNode(Node<K,V>[] tab) {
            //hash值为MOVED（-1）的节点就是ForwardingNode
            super(MOVED, null, null, null);
            this.nextTable = tab;
        }
        //通过此方法，访问被迁移到nextTable中的数据
        Node<K,V> find(int h, Object k) {
           ...
        }
    }

```

# 何时扩容

## 1 当前容量超过阈值

```
  final V putVal(K key, V value, boolean onlyIfAbsent) {
        ...
        addCount(1L, binCount);
        ...
  }

```

```
  private final void addCount(long x, int check) {
        ...
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            //s>=sizeCtl 即容量达到扩容阈值，需要扩容
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
               //调用transfer()扩容
               ...
            }
        }
    }

```

## 2 当链表中元素个数超过默认设定（8个），当数组的大小还未超过64的时候，此时进行数组的扩容，如果超过则将链表转化成红黑树

```
 final V putVal(K key, V value, boolean onlyIfAbsent) {
        ...
        if (binCount != 0) {
                    //链表中元素个数超过默认设定（8个）
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
        }
        ...
 }
      

```

```
    private final void treeifyBin(Node<K,V>[] tab, int index) {
        Node<K,V> b; int n, sc;
        if (tab != null) {
            //数组的大小还未超过64
            if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
                //扩容
                tryPresize(n << 1);
            else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                //转换成红黑树
                ...
            }
        }
    }

```

## 3 当发现其他线程扩容时，帮其扩容

```
   final V putVal(K key, V value, boolean onlyIfAbsent) {
      ...
       //f.hash == MOVED 表示为：ForwardingNode，说明其他线程正在扩容
       else if ((fh = f.hash) == MOVED)
           tab = helpTransfer(tab, f);
      ...
   }
  

```

# 扩容过程分析

1. 线程执行put操作，发现容量已经达到扩容阈值，需要进行扩容操作，此时transferindex=tab.length=32

![mark](https://blogimg.nos-eastchina1.126.net/171214/1j5jEjjfGC.png)

2. 扩容线程A 以cas的方式修改transferindex=32-16=16 ,然后按照降序迁移table[32]--table[16]这个区间的hash桶

![mark](https://blogimg.nos-eastchina1.126.net/171214/bECFGcgkKg.png)

3. 迁移hash桶时，会将桶内的链表或者红黑树，按照一定算法，拆分成2份，将其插入nextTable[i]和nextTable[i+n]（n是table数组的长度）。 迁移完毕的hash桶,会被设置成ForwardingNode节点，以此告知访问此桶的其他线程，此节点已经迁移完毕。

![mark](https://blogimg.nos-eastchina1.126.net/171214/8I0gCE52kl.png)

相关代码如下：

```
  private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
              ...//省略无关代码
              synchronized (f) {
                      //将node链表，分成2个新的node链表
                      for (Node<K,V> p = f; p != lastRun; p = p.next) {
                          int ph = p.hash; K pk = p.key; V pv = p.val;
                          if ((ph & n) == 0)
                              ln = new Node<K,V>(ph, pk, pv, ln);
                          else
                              hn = new Node<K,V>(ph, pk, pv, hn);
                      }
                      //将新node链表赋给nextTab
                      setTabAt(nextTab, i, ln);
                      setTabAt(nextTab, i + n, hn);
                      setTabAt(tab, i, fwd);
              }
              ...//省略无关代码
  }

```

4. 此时线程2访问到了ForwardingNode节点，如果线程2执行的put或remove等写操作，那么就会先帮其扩容。如果线程2执行的是get等读方法，则会调用ForwardingNode的find方法，去nextTable里面查找相关元素。

![mark](https://blogimg.nos-eastchina1.126.net/171214/k7aDHb1JCc.png)

5. 线程2加入扩容操作

![mark](https://blogimg.nos-eastchina1.126.net/171214/29j61FCbIA.png)

6. 如果准备加入扩容的线程，发现以下情况，放弃扩容，直接返回。

- 发现transferIndex=0,即**所有node均已分配**
- 发现扩容线程已经达到**最大扩容线程数**

![mark](https://blogimg.nos-eastchina1.126.net/171214/B6ch1929bL.png)

# 部分源码分析

## tryPresize方法

协调多个线程如何调用transfer方法进行hash桶的迁移（addCount，helpTransfer 方法中也有类似的逻辑）

```java
    private final void tryPresize(int size) {
        //计算扩容的目标size
        int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
            tableSizeFor(size + (size >>> 1) + 1);
        int sc;
        while ((sc = sizeCtl) >= 0) {
            Node<K,V>[] tab = table; int n;
            //tab没有初始化
            if (tab == null || (n = tab.length) == 0) {
                n = (sc > c) ? sc : c;
                //初始化之前，CAS设置sizeCtl=-1 
                if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                    try {
                        if (table == tab) {
                            @SuppressWarnings("unchecked")
                            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                            table = nt;
                            //sc=0.75n,相当于扩容阈值
                            sc = n - (n >>> 2);
                        }
                    } finally {
                        // 此时并没有通过CAS赋值，因为其他想要执行初始化的线程，
                        // 发现sizeCtl=-1，就直接返回，从而确保任何情况，
                        // 只会有一个线程执行初始化操作。
                        sizeCtl = sc;
                    }
                }
            }
            //目标扩容size小于扩容阈值，或者容量超过最大限制时，不需要扩容
            else if (c <= sc || n >= MAXIMUM_CAPACITY)
                break;
            //扩容
            else if (tab == table) {
                int rs = resizeStamp(n);
                //sc<0表示，已经有其他线程正在扩容
                if (sc < 0) {
                    Node<K,V>[] nt;
                //1 (sc >>> RESIZE_STAMP_SHIFT) != rs ：扩容线程数 > MAX_RESIZERS-1
                //2 sc == rs + 1 和 sc == rs + MAX_RESIZERS ：表示什么？？？       
                //3 (nt = nextTable) == null ：表示nextTable正在初始化
                //4 transferIndex <= 0 ：表示所有hash桶均分配出去
                     
                    //如果不需要帮其扩容，直接返回
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
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

## transfer方法

负责迁移node节点

```java
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        //计算需要迁移多少个hash桶（MIN_TRANSFER_STRIDE该值作为下限，以避免扩容线程过多）
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
       
        if (nextTab == null) {            // initiating
            try {
                //扩容一倍
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        int nextn = nextTab.length;
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; 
      
        // 1.逆序迁移已经获取到的hash桶集合，如果迁移完毕，
        // 则更新transferIndex，获取下一批待迁移的hash桶
        // 2.如果transferIndex=0，表示所以hash桶均被分配，
      	// 将i置为-1，准备退出transfer方法
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            
            //更新待迁移的hash桶索引
            while (advance) {
                int nextIndex, nextBound;
                //更新迁移索引i。
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    // transferIndex<=0表示已经没有需要迁移的hash桶，
                  	// 将i置为-1，线程准备退出
                    i = -1;
                    advance = false;
                }
                // 当迁移完bound这个桶后，尝试更新transferIndex，
                // 获取下一批待迁移的hash桶
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            //退出transfer
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    //最后一个迁移的线程，recheck后，做收尾工作，然后退出
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
			   // 第一个扩容的线程，执行transfer方法之前，会设置 sizeCtl = 
                // (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2) 	
                // 后续帮其扩容的线程，执行transfer方法之前，会设置 sizeCtl = sizeCtl+1
                // 每一个退出transfer的方法的线程，退出之前，会设置 sizeCtl = sizeCtl-1
                // 那么最后一个线程退出时：
                // 必然有sc == (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2)，
                // 即 (sc - 2) == resizeStamp(n) << RESIZE_STAMP_SHIFT
                    
                    //不相等，说明不到最后一个线程，直接退出transfer方法
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    //最后退出的线程要重新check下是否全部迁移完毕
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            //迁移node节点
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        //链表迁移
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
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
                            //将node链表，分成2个新的node链表
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            //将新node链表赋给nextTab
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        //红黑树迁移
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
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }

```

# 总结

多线程无锁扩容的关键就是通过CAS设置sizeCtl与transferIndex变量，协调多个线程对table数组中的node进行迁移。

**勘误：tab.length为32，扩容阈值是32*0.75=24**
