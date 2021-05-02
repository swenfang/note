---
tags:
	- 基础数据类型
categories: HashMap
title: HashMap 的源码分析
---

# JDK1.8 HashMap源码分析



HahsMap实现了Map接口。其继承关系如下图： 
![HashMap继承关系图](http://img.blog.csdn.net/20170312124705054?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMTk0MzEzMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 



HashMap有两个影响性能的重要参数：初始容量和加载因子。容量是Hash表中桶的个数，当HashMap初始化时，容量就是初始容量。加载因子是衡量hash表多满的一个指标，用来判断是否需要增加容量。当HashMap需要增加容量时，将会导致rehash操作。 
默认情况下，0.75的加载因子在时间和空间方面提供了很好的平衡。加载因子越大，增加了空间利用率但是也增加了查询的时间。

## 构造器

### 底层结构

#### JDK1.8之前的结构

在JDK1.7之前，HashMap采用的是数组+链表的结构，其结构图如下： 
![JDK1.7之前HashMap结构](http://img.blog.csdn.net/20160127173141337) 
左边部分代表Hash表，数组的每一个元素都是一个单链表的头节点，链表是用来解决冲突的，如果不同的key映射到了数组的同一位置处，就将其放入单链表中。

#### JDK1.8的结构

JDK1.8 之前的 HashMap 都采用上图的结构，都是基于一个数组和多个单链表，hash 值冲突的时候，就将对应节点以链表形式存储。如果在一个链表中查找一个节点时，将会花费 O(n) 的查找时间，会有很大的性能损失。到了JDK1.8，当同一个Hash值的节点数不小于8时，不再采用单链表形式存储，而是采用红黑树，如下图所示： 
![JDK1.8HashMap结构](http://img.blog.csdn.net/20160127173307041)

### Node介绍

Node是map的接口中的内部接口Map.Entry的实现类,用于存储HashMap中键值对的对象,是HashMap中非常重要的一个内部类,随便提一下,HashMap中有非常多的内部类,本文没做过多的介绍,读者可以自己翻看一下源码,因为篇幅实在太长了...在这里就简单的讲一下,大部分的内部类都是用于集合操作的,如`keySet`,`values`,`entrySet`等方法.

内部组成

```java
static class Node<K,V> implements Map.Entry<K,V> {
//key是不可变的
final K key;
//value
V value;
//指向下一个entry对象,具体作用后面介绍
Node<K,V> next;
//hash值
int hash;
}
```

### 重要的字段

HashMap中有几个重要的字段，如下：

```java
    //Hash表结构
    transient Node<K,V>[] table;

    //元素个数
    transient int size;

    //确保fail-fast机制
    transient int modCount;

    //下一次增容前的阈值
    int threshold;

    //加载因子
    final float loadFactor;

     //默认初始容量
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

   //最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;

    //加载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    //链表转红黑树的阈值
    static final int TREEIFY_THRESHOLD = 8;
    
```

### 红黑树的关键参数

```java
//一个桶的树化阈值
//当桶中元素个数超过这个值时，需要使用红黑树节点替换链表节点
//这个值必须为 8，要不然频繁转换效率也不高
static final int TREEIFY_THRESHOLD = 8;

//一个树的链表还原阈值
//当扩容时，桶中元素个数小于这个值，就会把树形的桶元素 还原（切分）为链表结构
//这个值应该比上面那个小，至少为 6，避免频繁转换
static final int UNTREEIFY_THRESHOLD = 6;

//哈希表的最小树形化容量
//当哈希表中的容量大于这个值时，表中的桶才能进行树形化
//否则桶内元素太多时会扩容，而不是树形化
//为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD
static final int MIN_TREEIFY_CAPACITY = 64;
```

### 构造方法

HashMap一共有4个构造方法，主要的工作就是完成容量和加载因子的赋值。Hash表都是采用的懒加载方式，当第一次插入数据时才会创建。

```java
//构造方法 初始化负载因子和阈值
public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        //容量判断 
    	if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
    	//负载银子判断
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

 public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

//为空时候使用默认分配的大小16,负载因子0.75f,默认的容量为12,当size>threshold默认容量时候就会去扩容
 public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
    }

 public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```

## 基本操作

### 确定哈希桶数组索引位置

不管增加、删除、查找键值对，定位到哈希桶数组的位置都是很关键的第一步。前面说过 HashMap 的数据结构是数组和链表的结合，所以我们当然希望这个 HashMap 里面的元素位置尽量分布均匀些，尽量使得每个位置上的元素数量只有一个，那么当我们用 hash 算法求得这个位置的时候，马上就可以知道对应位置的元素就是我们要的，不用遍历链表，大大优化了查询的效率。HashMap 定位数组索引位置，直接决定了 hash 方法的离散性能。先看看源码的实现(方法一+方法二):

```java
方法一：
static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取hashCode值
     // h ^ (h >>> 16)  为第二步 高位参与运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
方法二：
static int indexFor(int h, int length) {  //jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的
     return h & (length-1);  //第三步 取模运算
}

```

这里的 Hash 算法本质上就是三步：**取 key 的 hashCode 值、高位运算、取模运算**。

对于任意给定的对象，只要它的 hashCode() 返回值相同，那么程序调用方法一所计算得到的 Hash 码值总是相同的。我们首先想到的就是把 hash 值对数组长度取模运算，这样一来，元素的分布相对来说是比较均匀的。但是，模运算的消耗还是比较大的，在 HashMap 中是这样做的：调用方法二来计算该对象应该保存在  table  数组的哪个索引处。

这个方法非常巧妙，它通过 h & (table.length -1) 来得到该对象的保存位，而 HashMap 底层数组的长度总是 2 的n 次方，这是 HashMap 在速度上的优化。当 length 总是 2 的 n 次方时，h& (length-1) 运算等价于对 length 取模，也就是 h%length ，但是&比%具有更高的效率。

在JDK1.8的实现中，优化了高位运算的算法，通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

下面举例说明下，n为table的长度。

![hashMap哈希算法例图](https://tech.meituan.com/img/java-hashmap/hashMap%E5%93%88%E5%B8%8C%E7%AE%97%E6%B3%95%E4%BE%8B%E5%9B%BE.png)

### 添加一个元素put(K k,V v)

HashMap 允许K和V为 null，添加一个键值对时使用 put 方法，如果之前已经存在K的键值，那么旧值将会被新值替换。实现如下：

```java
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

/**
 * onlyIfAbsent 是否替换,为true时,如果存在值则替换
 * evict 主要用于LinkedHashMap移除最近未使用的节点
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //如果哈希表为空或长度为0，调用resize()方法创建哈希表
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //如果哈希表中K对应的桶为空，那么该K，V对将成为该桶的头节点
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //该桶处已有节点，即发生了哈希冲突
        else {
            Node<K,V> e; K k;
            //如果添加的值与头节点相同，将e指向p
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //如果与头节点不同，并且该桶目前已经是红黑树状态，调用putTreeVal()方法
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //桶中仍是链表阶段
            else {
                //遍历，要比较是否与已有节点相同
                for (int binCount = 0; ; ++binCount) {
                    //将e指向下一个节点，如果是null，说明链表中没有相同节点，添加到链表尾部即可
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //如果此时链表个数达到了8，那么需要将该桶处链表转换成红黑树，treeifyBin()方法将hash处的桶转成红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果与已有节点相同，跳出循环
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //如果有重复节点，那么需要返回旧值
            if (e != null) {
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                //子类实现(用于linkedHashMap)
                afterNodeAccess(e);
                return oldValue;
            }
        }
        //是一个全新节点，那么size需要+1
        ++modCount;
        //如果超过了阈值，那么需要resize()扩大容量
        if (++size > threshold)
            resize();
        //子类实现(用于linkedHashMap)
        afterNodeInsertion(evict);
        return null;
    }
   
```

从上面代码可以看到 putVal() 方法的流程： 

![mark](https://blogimg.nos-eastchina1.126.net/180306/LLFLH0iJb5.png)

1. 判断哈希表是否为空，如果为空，调用 resize() 方法进行创建哈希表 
2. 根据 hash 值得到哈希表中桶的头节点，如果为 null ，说明是第一个节点，直接调用 newNode() 方法添加节点即可 
3. 如果发生了哈希冲突，那么首先会得到头节点，比较是否相同，如果相同，则进行节点值的替换返回 
4. 如果头节点不相同，但是头节点已经是 TreeNode 了，说明该桶处已经是红黑树了，那么调用 putTreeVal() 方法将该结点加入到红黑树中 
5. 如果头节点不是 TreeNode，说明仍然是链表阶段，那么就需要从头开始遍历，一旦找到了相同的节点就跳出循环或者直到了链表尾部，那么将该节点插入到链表尾部 
6. 如果插入到链表尾部后，链表个数达到了阈值8，那么将会将该链表转换成红黑树，调用 treeifyBin() 方法 
7. 如果是新加一个数据，那么将 size+1，此时如果 size 超过了阈值，那么需要调用 resize() 方法进行扩容

#### resize()方法

下面我们一个一个分析上面提到的方法。首先是 resize() 方法，resize() 在哈希表为 null 时将会初始化，但是在已经初始化后就会进行容量扩展。下面是resize()的具体实现：

```JAVA
/**
 * 有几种情况
 * 1.当为空的时候,也就是没有初始化的时候
 * 2.当到达最大值时候
 * 3.普通扩容时候
 */
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;//旧表容量
        int oldThr = threshold;//旧表与之
        int newCap, newThr = 0;
        //旧表存在
        if (oldCap > 0) {
             // 超过最大值就不再扩充了，就只好随你碰撞去吧
            //旧表已经达到了最大容量，不能再大，直接返回旧表
            //大于2<<30 最大容量设置为2<<32 - 1
            if (oldCap >= MAXIMUM_CAPACITY) {
                //但是不移动.,没有空间移动
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 没超过最大值，就扩充为原来的2倍
            //否则，新容量为旧容量2倍，新阈值为旧阈值2倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        //如果就阈值>0，说明构造方法中指定了容量
        //用户自己设定了初始化大小
        else if (oldThr > 0)
            newCap = oldThr;
        //初始化时没有指定阈值和容量，使用默认的容量16和阈值16*0.75=12
        //如果没使用,使用默认值初始化
        else {              
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 计算新的resize上限
    	//用户自定义了map的初始化操作
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //更新阈值
    	//新的容量
        threshold = newThr;
        //创建表,初始化或更新表
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        //如果属于容量扩展，rehash操作
        if (oldTab != null) {
            // 把每个bucket都移动到新的buckets中
            //遍历旧表
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                //如果该桶处存在数据
                if ((e = oldTab[j]) != null) {
                    //将旧表数据置为null，帮助gc
                    oldTab[j] = null;
                    //如果只有一个节点，直接在新表中赋值
                    //如果当前位置只有一个元素,直接移动到新的位置
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    //如果该节点已经为红黑树
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    //如果没超过8个 是链表
                    //如果该桶处仍为链表
                    // 链表优化重hash的代码块
                    else {
                        
                        
//下面这段暂时没有太明白，通过e.hash & oldCap将链表分为两队，参考知乎上的一段解释 
/** 
* 把链表上的键值对按hash值分成lo和hi两串，lo串的新索引位置与原先相同[原先位 
* j]，hi串的新索引位置为[原先位置j+oldCap]； 
* 链表的键值对加入lo还是hi串取决于 判断条件if ((e.hash & oldCap) == 0)，因为* capacity是2的幂，所以oldCap为10...0的二进制形式，若判断条件为真，意味着 
* oldCap为1的那位对应的hash位为0，对新索引的计算没有影响（新索引 
* =hash&(newCap-*1)，newCap=oldCap<<2）；若判断条件为假，则 oldCap为1的那位* 对应的hash位为1， 
* 即新索引=hash&( newCap-1 )= hash&( (oldCap<<2) - 1)，相当于多了10...0， 
* 即 oldCap 

* 例子： 
* 旧容量=16，二进制10000；新容量=32，二进制100000 
* 旧索引的计算： 
* hash = xxxx xxxx xxxy xxxx 
* 旧容量-1 1111 
* &运算 xxxx 
* 新索引的计算： 
* hash = xxxx xxxx xxxy xxxx 
* 新容量-1 1 1111 
* &运算 y xxxx 
* 新索引 = 旧索引 + y0000，若判断条件为真，则y=0(lo串索引不变)，否则y=1(hi串 
* 索引=旧索引+旧容量10000) 
*/  
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
//此处的操作是这样子的 因为是扩容一倍的操作,所以与旧的容量进行与操作后只有两个值0 和 1
//如果是0就位置不变,如果是1就移动n+old的位置,
//个人认为这么做的好处是:
/**
* 1.不会像之前1.7发生循环依赖的问题
* 2.从概率的角度上来看可以均匀分配,(一般来说高位和低位比例差不多)
* 3.提高效率
*/
                        do {
                            next = e.next;
                            // 原索引
                            //如果和旧容量位运算后的值是0,记得当前节点和存放在链表的尾部
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            //同上
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 原索引放到bucket里
                        //为0的还是存放在当前位置
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 原索引+oldCap放到bucket里
                        //为1的就放在扩容的j + oldCap那边去
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

因为不像Java8之前的HashMap有初始化操作,此处选择将初始化和扩容放在了一起,并且又增加了红黑树的概念,所以导致整个方法的判断次数非常多,也是这个方法比较庞大的主要原因.

值得一体的是,在扩容后重新计算位置的时候,对链表进行优化,有兴趣可以搜索一下[HashMap导致cpu百分之百的问题](https://link.juejin.im/?target=http%3A%2F%2Fifeve.com%2Fhashmap-infinite-loop%2F) 而在Java中通过巧妙的进行&操作,然后获得高位是为0还是1.最终移动的位置就是低位的链表留在原地,高位的放在index+oldsize的地方就可以了,不用为每一个元素计算hash值,然后移动到对应的位置,再判断是否是链表,是否需要转换成树的操作.如下所示.

```java
hashcode: 1111111111111101212
oldcap:   0000000000000010000

```

很容易知道这个&操作之后就是为0,因为oldcap都是2的倍数,只有高位为1,所以通过&确认高位要比%取余高效. 此时在看一下上面的扩容操作也许就更清晰了.

resize()首先获取新容量以及新阈值，然后根据新容量创建新表。如果是扩容操作，则需要进行rehash操作，通过e.hash&oldCap将链表分为两列，更好地均匀分布在新表中。 

#### newNode()方法

下面介绍一些newNode方法.就是新建一个节点.可以思考一下为什么要把newNode抽离出来?

```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
    return new Node<>(hash, key, value, next);
}
```

#### putTreeVal() 方法

添加节点到红黑树的方法是Java8中新添加的,需要满足链表的长度到8,才会转换成红黑树,其主要目的是防止某个下标处的链表太长,导致在找到的时候速度很慢,下面看一下实现

```java
//尝试着往树节点添加值
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,int h, K k, V v) {
    Class<?> kc = null;
    boolean searched = false;
    //找到根节点
    TreeNode<K,V> root = (parent != null) ? root() : this;
    for (TreeNode<K,V> p = root;;) {
        int dir, ph; K pk;
        if ((ph = p.hash) > h)
            dir = -1;
        else if (ph < h)
            dir = 1;
        //存在的话直接返回,用于替换
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        //判断节点类型是否相同,不相同
        else if ((kc == null &&
                  (kc = comparableClassFor(k)) == null) ||
                 (dir = compareComparables(kc, k, pk)) == 0) {
            //没有搜索过,搜索子节点,搜过了说明没有就跳过.
            if (!searched) {
                TreeNode<K,V> q, ch;
                searched = true;
                //去子节点去找
                if (((ch = p.left) != null &&(q = ch.find(h, k, kc)) != null) ||((ch = p.right) != null &&
                     (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            //对比hash值,决定查找的方向
            dir = tieBreakOrder(k, pk);
        }

        TreeNode<K,V> xp = p;
        //找到子节点为空,就可以加进去,设置层级关系
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            Node<K,V> xpn = xp.next;
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            xp.next = x;
            x.parent = x.prev = xp;
            if (xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;
            moveRootToFront(tab, balanceInsertion(root, x));
            return null;
        }
    }
}
```

这里简单的梳理一下流程. 1.从根节点查找,找到了返回,如果没找到,找字节点 2.判断往哪个方向去查找 3.如果不存在,在子节点末端添加新节点

#### split() 方法

树的 split() 方法,主要是扩容操作,重新结算位置需要分裂树,之前讲过,扩容会根据和旧 map 容量进行&操作,移动高位为1的节点.并且验证新的节点列表是否需要重新转换成链表的形式.

当头节点是TreeNode时，将调用TreeNode的split方法将红黑树复制到新表中，代码实现如下：

```java
final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
            TreeNode<K,V> b = this;//就是上面的头结点e
           // 设置记录高低位的node,和链表一样都是计算高位是0还是1
            //与链表rehash时类似，将红黑树分为两部分
            TreeNode<K,V> loHead = null, loTail = null;
            TreeNode<K,V> hiHead = null, hiTail = null;
            int lc = 0, hc = 0;
            //遍历
            for (TreeNode<K,V> e = b, next; e != null; e = next) {
                next = (TreeNode<K,V>)e.next;
                e.next = null;
                //还是和旧的容量做位运算,为0的放在lo中
                //分散规则与rehash中相同
                if ((e.hash & bit) == 0) {
                    //判断是否为头部
                    if ((e.prev = loTail) == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                    ++lc;
                }
                //获取为1的放在hi中
                else {
                    if ((e.prev = hiTail) == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                    ++hc;
                }
            }

    		//lo链表的处理
            //如果存在低端
            if (loHead != null) {
                //如果小于7,那么当做链表处理
                //如果分散后的红黑树节点小于等于6，将红黑树节点转换成链表节点
                if (lc <= UNTREEIFY_THRESHOLD)
                    tab[index] = loHead.untreeify(map);
                else {
                    //转换成树
                    tab[index] = loHead;
                    //将链表转换成红黑树
                    if (hiHead != null) // (else is already treeified)
                        loHead.treeify(tab);
                }
            }
    		//同上
            //如果存在高端
            if (hiHead != null) {
                //如果分散后的红黑树节点小于等于6，将红黑树节点转换成链表节点
                if (hc <= UNTREEIFY_THRESHOLD)
                    tab[index + bit] = hiHead.untreeify(map);
                else {
                    tab[index + bit] = hiHead;
                    //将链表转换成红黑树节点
                    if (loHead != null)
                        hiHead.treeify(tab);
                }
            }
        }

//把树转换成链表
final Node<K,V> untreeify(HashMap<K,V> map) {
    Node<K,V> hd = null, tl = null;
    for (Node<K,V> q = this; q != null; q = q.next) {
      Node<K,V> p = map.replacementNode(q, null);
      if (tl == null)
        hd = p;
      else
        tl.next = p;
      tl = p;
    }
    return hd;
}

```

TreeNode的split方法首先将头节点从头开始遍历，区分出两条单链表，再根据如果节点数小于等于6，那么将单链表的每个TreeNode转换成Node节点；否则将单链表转换成红黑树结构。 
至此，resize()方法结束。需要注意的是rehash时，由于容量扩大一倍，本来一条链表有可能会分成两条链表，而如果将红黑树结构复制到新表时，有可能需要完成红黑树到单链表的转换。

#### treeifyBin()方法

treeifyBin()方法将表中某一个桶处的单链表结果转换成红黑树结构，其实现如下：

```java
 final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //如果哈希表不存在，或者哈希表尺寸小于64，进行resize()扩容
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        //如果桶处头节点不为null
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            //将单链表节点转换成TreeNode结构的单链表
            do {
                //将Node转换成TreeNode
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            //调用treeify将该TreeNode结构的单链表转换成红黑树
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```

上述操作做了这些事:

- 根据哈希表中元素个数确定是扩容还是树形化
- 如果是树形化
  - 遍历桶中的元素，创建相同个数的树形节点，复制内容，建立起联系
  - 然后让桶第一个元素指向新建的树头结点，替换桶的链表内容为树形内容

#### treeify() 方法

但是我们发现，之前的操作并没有设置红黑树的颜色值，现在得到的只能算是个二叉树。在 最后调用树形节点 hd.treeify(tab) 方法进行塑造红黑树，来看看代码：

```java
       final void treeify(Node[] tab) {
        TreeNode root = null;
        for (TreeNode x = this, next; x != null; x = next) {
            next = (TreeNode)x.next;
            x.left = x.right = null;
            if (root == null) { //头回进入循环，确定头结点，为黑色
                x.parent = null;
                x.red = false;
                root = x;
            }
            else {  //后面进入循环走的逻辑，x 指向树中的某个节点
                K k = x.key;
                int h = x.hash;
                Class kc = null;
                //又一个循环，从根节点开始，遍历所有节点跟当前节点 x 比较，调整位置，有点像冒泡排序
                for (TreeNode p = root;;) {
                    int dir, ph;        //这个 dir 
                    K pk = p.key;
                    if ((ph = p.hash) > h)  //当比较节点的哈希值比 x 大时， dir 为 -1
                        dir = -1;
                    else if (ph < h)  //哈希值比 x 小时 dir 为 1
                        dir = 1;
                    else if ((kc == null &&
                              (kc = comparableClassFor(k)) == null) ||
                             (dir = compareComparables(kc, k, pk)) == 0)
                        // 如果比较节点的哈希值、 x 
                        dir = tieBreakOrder(k, pk);

                        //把 当前节点变成 x 的父亲
                        //如果当前比较节点的哈希值比 x 大，x 就是左孩子，否则 x 是右孩子 
                    TreeNode xp = p;
                    if ((p = (dir <= 0) ? p.left : p.right) == null) {
                        x.parent = xp;
                        if (dir <= 0)
                            xp.left = x;
                        else
                            xp.right = x;
                        root = balanceInsertion(root, x);
                        break;
                    }
                }
            }
        }
        moveRootToFront(tab, root);
    }
```

### put操作总结

当调用put插入一个键值对时，在表为空时，会创建表。如果桶为空时，直接插入节点，如果桶不为空时，则需要对当前桶中包含的结构做判断，如果已是红黑树结构，那么需要使用红黑树的插入方法；如果不是红黑树结构，则需要遍历链表，如果添加到链表后端，如果该条链表达到了8，那么需要将该链表转换成红黑树，从treeifyBin方法可以看到，当容量小于64时，不会进行红黑树转换，只会扩容。当成功新加一个桶，那么需要将尺寸和阈值进行判断，是否需要进行resize()操作。

### get(K k)操作

get(K k)根据键得到值，如果值不存在，那么返回null。其实现如下：

```java
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

//根据键的hash值和键得到对应节点
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //可以从桶中得到对应hash值的第一个节点
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //检查首节点，如果首节点匹配，那么直接返回首节点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //如果首节点还有后续节点
            if ((e = first.next) != null) {
                //如果首节点是红黑树节点，调用getTreeNode()方法
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //首节点是链表结构，从前往后遍历
                do {
                    //一旦匹配，返回节点
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }

```

从上面代码可以看到getNode()方法中有多种情况： 
\1. 表为空或表的长度为0或表中不存在key对应的hash值桶，那么返回null 
\2. 如果表中有key对应hash值的桶，得到首节点，如果首节点匹配，那么直接返回； 
\3. 如果首节点不匹配，并且没有后续节点，那么返回null 
\4. 如果首节点有后续节点并且首节点是TreeNode,调用getTreeNode方法寻找节点 
\5. 如果首节点有后续节点并且是链表结构，那么从前往后遍历，一旦找到则返回节点，否则返回null

### remove()操作

remove(K k)用于根据键值删除键值对，如果哈希表中存在该键，那么返回键对应的值，否则返回null。其实现如下：

```java
public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }

//按照hash和key删除节点，如果不存在节点，则返回null
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        //如果哈希表不为空并且存在桶与hash值匹配,p为桶中的头节点
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            //case 1：如果头节点匹配
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
               //case2：如果头节点不匹配，且头节点是TreeNode，即桶中的结构为红黑树结构
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                //case 3:如果头节点不匹配，且头节点是Node，即桶中的结构为链表结构，遍历链表
                    do {
                        //一旦匹配，跳出循环
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }

            //如果存在待删除节点节点
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                //如果节点是TreeNode，使用红黑树的方法
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                //如果待删除节点是头节点，更改桶中的头节点即可
                else if (node == p)
                    tab[index] = node.next;
                //在链表遍历过程中，p代表node节点的前驱节点
                else
                    p.next = node.next;
                ++modCount;
                --size;
                //子类实现
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }    
```

从上面的代码可以看出，removeNode()方法首先是找到待删除的节点，如果存在待删除节点，接下来再执行删除操作。查询时流程与getNode()方法的流程类似，只不过多了在遍历链表时还需要保存前驱节点，因为后面删除时要用到（单链表结构）。删除节点时就比较简单了，三种情况三种处理方式,分别是： 
\1. 如果待删除节点是TreeNode，那么调用removeTreeNode()方法 
\2. 如果待删除节点是Node，并且待删除节点就是头节点，那么将头节点更改为原有节点的下一个节点就可以了 
\3. 如果待删除节点是Node且待删除节点不是头节点，那么将遍历过程中保存的前驱节点p的后继节点设为node的后继节点就可以了

### 红黑树中查找元素 getTreeNode()

HashMap 的查找方法是 get():

```java
public V get(Object key) {
    Node e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

```

它通过计算指定 key 的哈希值后，调用内部方法 getNode()；

```java
final Node getNode(int hash, Object key) {
    Node[] tab; Node first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

```

这个 getNode() 方法就是根据哈希表元素个数与哈希值求模（`使用的公式是 (n - 1) &hash`）得到 key 所在的桶的头结点，如果头节点恰好是红黑树节点，就调用红黑树节点的 getTreeNode() 方法，否则就遍历链表节点。

```java
    final TreeNode getTreeNode(int h, Object k) {
        return ((parent != null) ? root() : this).find(h, k, null);
    }

```

getTreeNode 方法使通过调用树形节点的 find() 方法进行查找：

```java
    //从根节点根据 哈希值和 key 进行查找
    final TreeNode find(int h, Object k, Class kc) {
        TreeNode p = this;
        do {
            int ph, dir; K pk;
            TreeNode pl = p.left, pr = p.right, q;
            if ((ph = p.hash) > h)
                p = pl;
            else if (ph < h)
                p = pr;
            else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                return p;
            else if (pl == null)
                p = pr;
            else if (pr == null)
                p = pl;
            else if ((kc != null ||
                      (kc = comparableClassFor(k)) != null) &&
                     (dir = compareComparables(kc, k, pk)) != 0)
                p = (dir < 0) ? pl : pr;
            else if ((q = pr.find(h, k, kc)) != null)
                return q;
            else
                p = pl;
        } while (p != null);
        return null;
    }

```

由于之前添加时已经保证这个树是有序的，因此查找时基本就是折半查找，效率很高。

这里和插入时一样，如果对比节点的哈希值和要查找的哈希值相等，就会判断 key 是否相等，相等就直接返回（也没有判断值哎）；不相等就从子树中递归查找。



## HashMap总结

至此，我们分析完了HashMap的主要方法：构造器、put、get和remove。只需要明白JDK1.8的HashMap底层结构，那么就很好理解了。需要注意的是什么时候应该将链表结构转换成红黑树结构，什么时候又应该将红黑树结构重新转换成链表结构，本文没有具体解释有关红黑树的结构，但是这并不影响理解HashMap的基本原理。 
**另外需要注意的是，本文的源码是基于JDK1.8的。**
