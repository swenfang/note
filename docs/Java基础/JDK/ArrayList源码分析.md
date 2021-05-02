---
tags:
	- 基础数据类型
categories: ArrayList
title: ArrayList 源码分析
---

# ArrayList 源码分析

![mark](https://blogimg.nos-eastchina1.126.net/171208/C88BKj3C76.png)

# 前言

以面试问答的形式学习我们的最常用的装载容器——`ArrayList`（源码分析基于JDK8）

<!-- more -->

# 问答内容

## ArrayList是什么，可以用来干嘛？

问：ArrayList有用过吗？它是一个什么东西？可以用来干嘛？

答：有用过，ArrayList就是数组列表，主要用来装载数据，当我们装载的是基本类型的数据`int,long,boolean,short,byte...`的时候我们只能存储他们对应的包装类，它的主要底层实现是数组`Object[] elementData`。与它类似的是LinkedList，和LinkedList相比，它的查找和访问元素的速度较快，但新增，删除的速度较慢。

示例代码：

```java
        // 创建一个ArrayList，如果没有指定初始大小，默认容器大小为10
        ArrayList<String> arrayList = new ArrayList<String>();
        // 往容器里面添加元素
        arrayList.add("张三");
        arrayList.add("李四");
        arrayList.add("王五");
        // 获取index下标为0的元素      张三
        String element = arrayList.get(0);
        // 删除index下标为1的元素      李四
        String removeElement = arrayList.remove(1);

```

ArrayList底层实现示意图

## ArrayList中不断添加数据会有什么问题吗？

问：您说它的底层实现是数组，但是数组的大小是定长的，如果我们不断的往里面添加数据的话，不会有问题吗？

答：ArrayList可以通过构造方法在初始化的时候指定底层数组的大小。

- 通过无参构造方法的方式`ArrayList()`初始化，则赋值底层数组`Object[] elementData`为一个默认空数组`Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}`所以数组容量为0，只有真正对数据进行添加`add`时，才分配默认`DEFAULT_CAPACITY = 10`的初始容量。
  示例代码：

```java
    // 定义ArrayList默认容量为10
    private static final int DEFAULT_CAPACITY = 10;

    // 空数组，当调用无参构造方法时默认复制这个空数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    // 真正保存数据的底层数组
    transient Object[] elementData; 

    // ArrayList的实际元素数量
    private int size;

    public ArrayList() {
        // 无参构造方法默认为空数组
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

```

- 通过指定容量初始大小的构造方法方式`ArrayList(int initialCapacity)`初始化，则赋值底层数组`Object[] elementData`为指定大小的数组`this.elementData = new Object[initialCapacity];`
  示例代码：

```java
    private static final Object[] EMPTY_ELEMENTDATA = {};

    // 通过构造方法出入指定的容量来设置默认底层数组大小 
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

```

- 当我们添加的元素数量已经达到底层数组`Object[] elementData`的上限时，我们再往ArrayList元素，则会触发ArrayList的自动扩容机制，ArrayList会通过位运算`int newCapacity = oldCapacity + (oldCapacity >> 1);`以1.5倍的方式初始化一个新的数组（如初始化数组大小为10，则扩容后的数组大小为15），然后使用`Arrays.copyOf(elementData, newCapacity);`方法将原数据的数据逐一复制到新数组上面去，以此达到ArrayList扩容的效果。虽然，`Arrays.copyOf(elementData, newCapacity);`方法最终调用的是`native void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)`是一个底层方法，效率还算可以，但如果我们在知道ArrayList想装多少个元素的情况下，却没有指定容器大小，则就会导致ArrayList频繁触发扩容机制，频繁进行底层数组之间的数据复制，大大降低使用效率。
  示例代码：

```java
    public boolean add(E e) {
        //确保底层数组容量，如果容量不足，则扩容
        ensureCapacityInternal(size + 1); 
        elementData[size++] = e;
        return true;
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // 容量不足，则调用grow方法进行扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * 扩容方法(重点)
     */
    private void grow(int minCapacity) {
        // 获得原容量大小
        int oldCapacity = elementData.length;
        // 新容量为原容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 再判断新容量是否已足够，如果扩容后仍然不足够，则复制为最小容量长度
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 判断是否超过最大长度限制
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 将原数组的数据复制至新数组， ArrayList的底层数组引用指向新数组
        // 如果数据量很大，重复扩容，则会影响效率
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

```

- 因此，在我们使用ArrayList的时候，如果知道最终的存储容量capacity，则应该在初始化的时候就指定ArrayList的容量`ArrayList(int initialCapacity)`，如果初始化时无法预知装载容量，但在使用过程中，得知最终容量，我们可以通过调用`ensureCapacity(int minCapacity)`方法来指定ArrayList的容量，并且，如果我们在使用途中，如果确定容量大小，但是由于之前每次扩容都扩充50%，所以会造成一定的存储空间浪费，我们可以调用`trimToSize()`方法将容器最小化到存储元素容量，进而消除这些存储空间浪费。例如：我们当前存储了11个元素，我们不会再添加但是当前的ArrayList的大小为15，有4个存储空间没有被使用，则调用`trimToSize()`方法后，则会重新创建一个容量为11的数组`Object[] elementData`，将原有的11个元素复制至新数组，达到节省内存空间的效果。
  示例代码：

```java
    /**
     * 将底层数组一次性指定到指定容量的大小
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table 
             ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

    /**
     * 将容器最小化到存储元素容量
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }

```

## ArrayList怎么 删除数据，为什么访问速度快，删除新增速度慢 ？

问：那它是怎么样删除元素的？您上面说到ArrayList访问元素速度较快，但是新增和删除的速度较慢，为什么呢？

答：

- 通过源码我们可以得知，ArrayList删除元素时，先获取对应的删除元素，然后把要删除元素对应索引index后的元素逐一往前移动1位，最后将最后一个存储元素清空并返回删除元素，以此达到删除元素的效果。
- 当我们通过下标的方式去访问元素时，我们假设访问一个元素所花费的时间为K，则通过下标一步到位的方式访问元素，时间则为1K，用“大O”表示法表示，则时间复杂度为O(1)。所以ArrayList的访问数据的数据是比较快的。
- 当我们去添加元素`add(E e)`时，我们是把元素添加至末尾，不需要移动元素，此时的时间复杂度为O(1)，但我们把元素添加到指定位置，最坏情况下，我们将元素添加至第一个位置`add(int index, E element)`，则整个ArrayList的n-1个元素都要往前移动位置，导致底层数组发生n-1次复制。通常情况下，我们说的时间复杂度都是按最坏情况度量的，此时的时间复杂度为O(n)。删除元素同理，删除最后一个元素不需要移动元素，时间复杂度为O(1)，但删除第一个元素，则需要移动n-1个元素，最坏情况下的时间复杂度也是O(n)。
- 所以ArrayList访问元素速度较快，但是新增和删除的速度较慢。

示例代码：

```java
    /**
     * 将元素添加至末尾
     */
    public boolean add(E e) {
        // 确保底层数组容量，如果容量不足，则扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    /**
     * 将元素添加至指定下标位置
     */
    public void add(int index, E element) {
         // 检查下标是否在合法范围内
        rangeCheckForAdd(index);
        // 确保底层数组容量，如果容量不足，则扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 将要添加的元素下标后的元素通过复制的方式逐一往后移动，腾出对应index下标的存储位置
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        // 将新增元素存储至指定下标索引index
        elementData[index] = element;
        // ArrayList的大小 + 1
        size++;
    }

    /**
     * 通过下标索引的方式删除元素
     */
    public E remove(int index) {
        // 检查下标是否在合法范围内
        rangeCheck(index);

        modCount++;
        // 直接通过下标去访问底层数组的元素
        E oldValue = elementData(index);

        // 计算数组需要移动的元素个数
        int numMoved = size - index - 1;
        if (numMoved > 0)
            // 将要删除的元素下标后的元素通过复制的方式逐一往前移动
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        //将底层数组长度减1，并清空最后一个存储元素。
        elementData[--size] = null; // clear to let GC do its work
        // 返回移除元素
        return oldValue;
    }

```

## ArrayList是线程安全的吗？

问：ArrayList是线程安全的吗？

答：ArrayList不是线程安全的，如果多个线程同时对同一个ArrayList更改数据的话，会导致数据不一致或者数据污染。如果出现线程不安全的操作时，ArrayList会尽可能的抛出`ConcurrentModificationException`防止数据异常，当我们在对一个ArrayList进行遍历时，在遍历期间，我们是不能对ArrayList进行添加，修改，删除等更改数据的操作的，否则也会抛出`ConcurrentModificationException`异常，此为fail-fast（快速失败）机制。从源码上分析，我们在`add,remove,clear`等更改ArrayList数据时，都会导致modCount的改变，当`expectedModCount != modCount`时，则抛出`ConcurrentModificationException`。如果想要线程安全，可以考虑使用Vector、CopyOnWriteArrayList。

示例代码：

```java
    /**
     * AbstractList.Itr 的迭代器实现
     */
    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        //期望的modCount
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

```

# 总结

1. 如果在初始化的时候知道ArrayList的初始容量，请一开始就指定容量`ArrayList<String> list = new ArrayList<String>(20);`,如果一开始不知道容量，中途才得知，请调用`list.ensureCapacity(20);`来扩充容量，如果数据已经添加完毕，但仍需要保存在内存中一段时间，请调用`list.trimToSize()`将容器最小化到存储元素容量，进而消除这些存储空间浪费。
2. ArrayList是以1.5倍的容量去扩容的，如初始容量是10，则容量依次递增扩充为：15，22，33，49。扩容后把原始数据从旧数组复制至新数组中。
3. ArrayList访问元素速度较快，下标方式访问元素，时间复杂度为O(1)，添加与删除速度较慢，时间复杂度均为O(n)。
4. ArrayList不是线程安全的，但是在发生并发行为时，它会尽可能的抛出`ConcurrentModificationException`，此为fail-fast机制。
