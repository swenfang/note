---
tags:
	- python
categories: 常用的内置（BIF）函数
title: Python 常用的内置（BIF）函数
---
# Python 常用的内置函数

如果你遇到一个需求，且你认为这个需求很普遍，先想想有没有什么内置函数可以使用（BIF）。另外要记住：Python 3 包含 70 多个 BIF ，所以有大量现成的功能等着你来发现。

<!-- more -->

## list()

这是一个工厂函数，创建一个新的空列表。

```python
import os;
aTuole = (123,'xyz','zare','abc');
aList = list(aTuole);
print("列表元素：",aList);
```

![mark](https://blogimg.nos-eastchina1.126.net/171229/8b4kC2eae9.png)



## range() 

**range() BIF 迭代固定次数。**

可以提供你需要的控制来迭代指定的次数，而且可以用来生成一个从 0 直到（但不包含）某个数的数字列表。

一下是这个 BIF 的用法：

```python
import os;
# num 是目标标识符，会琢个赋值为 "range()" 生成的各个数字
for num in range(4); 
print(num);
```

F5 运行程序

![mark](https://blogimg.nos-eastchina1.126.net/171228/mmHDibgjKc.png)



## enumerate()

创建成对数据的一个编码列表，从 0 开始

**先来做一个对比:**

法1: 使用 range() 和 len() 来实现

```python
import os;
aTuole = ('xyz','zare','abc');
	print(i,aTuole[i]);
  
```

法2:使用enumerate () 来实现

```python
import os;
aTuole = ('xyz','zare','abc');
for index,text in enumerate(aTuole):
	print(index,text);
```

enumerate会将数组或列表组成一个索引序列。使我们再获取索引和索引内容的时候更加方便。

![mark](https://blogimg.nos-eastchina1.126.net/171229/B9fKi2g1j6.png)



## int()

int()函数的作用是将一个数字或base类型的字符串转换成整数。

函数原型 int(x, base=10)，base缺省值为10，也就是说不指定base的值时，函数将x按十进制处理。

**注意：**

- x 可以是数字或字符串，但是base被赋值后 x 只能是字符串
- x 作为字符串时必须是 base 类型，也就是说 x 变成数字时必须能用 base 进制表示

【1】 x 是数字的情况：

```python
int(2.345)         # 2
int(2e2)           # 200
int(23, 2)         # 出错，base 被赋值后函数只接收字符串
```

![mark](https://blogimg.nos-eastchina1.126.net/171229/djc6e8BJEk.png)

【2】x 是字符串的情况：

```python
int('23', 16)      # 35
int('HI', 16)      # 出错，HI不是个16进制数
```

![mark](https://blogimg.nos-eastchina1.126.net/171229/L8glIeA3J4.png)

【3】 base 可取值范围是 2~36，囊括了所有的英文字母(不区分大小写)，十六进制中F表示15，那么G将在二十进制中表示16，依此类推....Z在三十六进制中表示35

```python
int('FZ', 16)      # 出错，FZ不能用十六进制表示
int('FZ', 36)      # 575
```

![mark](https://blogimg.nos-eastchina1.126.net/171229/14Fh4KLmBL.png)

【4】字符串 0x 可以出现在十六进制中，视作十六进制的符号，同理 0b 可以出现在二进制中，除此之外视作数字 0 和字母 x

```python
int('0x10', 16)  # 16，0x是十六进制的符号
int('0x10', 17)  # 出错，'0x10'中的 x 被视作英文字母 x
int('0x10', 36)  # 42804，36进制包含字母 x
```

![mark](https://blogimg.nos-eastchina1.126.net/171229/0lCmkdBd14.png)

## id()

id(object)函数是返回对象object在其生命周期内位于内存中的地址，id函数的参数类型是一个对象。

**注意：**

我们需要明确一点就是在Python中一切皆**对象**，变量中存放的是对象的引用。这个确实有点难以理解，“一切皆对象”？对，在Python中确实是这样，包括我们之前经常用到的字符串常量，整型常量都是对象。

```python
import os;
print(id(5));
print( id('python'));
x=2
print(id(x));
y='hello'
print(id(y));
```

这段代码的运行结果:

![mark](https://blogimg.nos-eastchina1.126.net/171229/kgjH35eHAd.png)



```python
import os;
x=2
print(id(2));
print(id(x)); 
y='hello'
print(id('hello')); 
print(id(y));
```

运行结果:

![mark](https://blogimg.nos-eastchina1.126.net/171229/B89BAdgEGe.png)

**结果说明:**对于这个语句id(2)没有报错，就可以知道2在这里是一个对象。id(x)和id(2)的值是一样的，id(y)和id('hello')的值也是一样的。

```python
x=2;
print(id(x));
y=2;
print(id(y));
s='hello';
print(id(s));
t=s;
print(id(t));
```

运行结果:

![mark](https://blogimg.nos-eastchina1.126.net/171229/d9i27aEKKg.png)

**结果说明:**id(x)和id(y)的结果是相同的，id(s)和id(t)的结果也是相同的。这说明x和y指向的是同一对象，而t和s也是指向的同一对象。x=2这句让变量x指向了int类型的对象2，而y=2这句执行时，并不重新为2分配空间，而是**让y直接指向了已经存在的int类型的对象2**.这个很好理解，因为本身只是想给y赋一个值2，而在内存中已经存在了这样一个int类型对象2，所以就直接让y指向了已经存在的对象。这样一来**不仅能达到目的，还能节约内存空间**。t=s这句变量互相赋值，也相当于是让t指向了已经存在的字符串类型的对象'hello'。

**看这幅图就理解了：**

　　![img](http://images.cnitblog.com/blog/288799/201303/15155442-4f2d077a181e4c37bc8691c2739a911f.jpg)



```python
x=2;
print(id(2)); 
print(id(x)); 
x=3;
print(id(3)); 
print(id(x)); 
L=[1,2,3];
M=L;
print(id(L));
print(id(M)); 
print(id(L[2])); 
L[0]=2;
print(id(L)); 
print(M); 
```

运行结果:

![mark](https://blogimg.nos-eastchina1.126.net/171229/HDCKE684L4.png)



**结果分析:**两次的id(x)的值不同，这个可能让人有点难以理解。注意，**在Python中，单一元素的对象是不允许更改的，比如整型数据、字符串、浮点数等。**x=3这句的执行过程并不是先获取x原来指向的对象的地址，再把内存中的值更改为3，而是新申请一段内存来存储对象3，再让x去指向对象3，所以两次id(x)的值不同。然而为何改变了L中的某个子元素的值后，id(L)的值没有发生改变？**在Python中，复杂元素的对象是允许更改的，**比如列表、字典、元组等。Python中变量存储的是对象的引用，对于列表，其id()值返回的是列表第一个子元素L[0]的存储地址。就像上面的例子，L=[1,2,3]，这里的L有三个子元素L[0]，L[1]，L[2]，L[0]、L[1]、L[2]分别指向对象1、2、3，id(L)值和对象3的存储地址相同.

**看下面这个图就明白了:**

　　![img](http://images.cnitblog.com/blog/288799/201303/15162702-c783bec88969421ebc16950714812a06.jpg)

因为L和M指向的是同一对象，所以在更改了L中子元素的值后，M也相应改变了，但是id(L)值并没有改变，因为这句L[0]=2只是让L[0]重新指向了对象2，而L[0]本身的存储地址并没有发生改变，所以id(L)的值没有改变（ id(L)的值实际等于L[0]本身的存储地址）。

## next()

next()函数返回迭代器的下一个元素

```python
it = iter([10, 20, 30, 40])
while True:
    try:
        x = next(it)
        print(x); # 或者 x = it.next()
    except StopIteration:
        break
```

运行结果:

![mark](https://blogimg.nos-eastchina1.126.net/171229/m4DBDAJ72h.png)
