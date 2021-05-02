---
tags:
	- python
categories: 数据迭代
title: 数据迭代的基本用法
---
# 数据迭代的基本用法：

## for 循环的使用

```python
>>>fav_movies = ["The Holy Crail","The Life of Brian"]
>>>for each_flick in fav_movies:print(each_flick)

The Holy Crail
The Life of Brian

```

<!-- more -->

## while 循环的使用

```python
>>>count=2
>>>while count>0: 【注意加冒号了再回车】
		print("aaaa") 【如果正确会自动缩进】
		count = count-1
		
aaaa 【输出结果】
aaaa
	
```

## enumerate，dict，zip  使用

通过一个练习，看看这三个函数怎么用的哈。小丽，你也跟着敲一下代码，看看运行效果

### 题目：

现有:

```
list1=['neil','mike','lucy']
list2=['123456','xuiasj==','passWD123']
list3=['www.abc.com','www.mike.org','www.lucy.gov']

```

需要形成列表：

```
fin_list = [{'name':'neil','passwd':'123456','url':'www.abc.com'},{'name':'mike','passwd':'xuiasj==','url':'www.mike.org'},{'name':'lucy','passwd':'passWD123','url':'www.lucy.gov'}]

```

python新手的小丽啊，这要如何实现呢？你先自己好好想想怎么实现，动手敲敲代码哈。

在这里我提供两种方法：

**方法1: 使用 enumerate 函数来实现**

```python
【按题目定义3个列表】
>>>list1 = ['neil','mike','lucy']
>>>list2 = ['123456','xuiasj==','passWD123']
>>>list3 = ['www.abc.com','www.mike.org','www.lucy.gov']
>>>fin_list = []
>>>for i , name in enumerate(list1):
  		d = {}
    	d['name'] = name
        d['password'] = list2[i]
        d['url'] = list3[i]
        fin_list.append(d)
        							【小丽你回车,会空格一行】
>>>fin_list 【再输入要输出的这个列表名】
【这是输出的结果】
[{'name': 'neil', 'password': '123456', 'url': 'www.abc.com'}, {'name': 'mike', 'password': 'xuiasj==', 'url': 'www.mike.org'}, {'name': 'lucy', 'password': 'passWD123', 'url': 'www.lucy.gov'}]

```

之前你问我:python 中 in 的使用 和 enumerate 函数，在这里我来回答你，你要认真看哈！



### enumerate 函数

一般情况下我们对一个列表或数组既要遍历索引又要遍历元素时，会这样写： 

```python
>>>for i in range (0,len(list)):  【其实，在 ypthon 中 for... in .. 它就是一个语法】
>>>print i ,list[i]				  【range 是取一个范围的值】	

```

但是这种方法有些累赘，使用内置enumerrate函数会有更加直接，优美的做法，先看看enumerate的定义：

```python
>>>def enumerate(collection):    【def 函数我在下面会跟你讲】
>>>  'Generates an indexed series:  (0,coll[0]), (1,coll[1])'
>>>  i = 0 
>>>  it = iter(collection) 
>>>  while 1: 
>>>  yield (i, it.next()) 
>>>  i += 1

```

 **enumerate会将数组或列表组成一个索引序列**。使我们再获取索引和索引内容的时候更加方便如下：

```python
>>>for index，text in enumerate(list)):
>>>   print index ,text

```

如果你要计算文件的行数，可以这样写：

```python
>>>count = len(open(thefilepath,‘rU’).readlines())

```

前面这种方法简单，但是可能比较慢，当文件比较大时甚至不能工作，下面这种循环读取的方法更合适些。

```python
>>>Count = -1 
>>>For count,line in enumerate(open(thefilepath,‘rU’))：
>>>  Pass
>>>Count += 1

```

小丽，计算文件的行数的这两种方法，看不懂没关系，你只要知道就可以了。

看到这里你必须掌握的是：for 的迭代 和 enumerate 行数的使用

**在来看看 def 函数**

### def 函数

对于某些需要重复调用的程序，可以使用函数进行定义，基本形式为：

def 函数名(参数1, 参数2, ……, 参数N):

执行语句函数名为调用的表示名，参数则是传入的参数。 

```python
# 例1：简单的函数使用
# coding=gb2312
 
# 定义函数
def hello():
  print 'hello python!'
   
# 调用函数    
hello()
   
>>> hello python!
```

函数可以带参数和返回值，参数将按从左到右的匹配，参数可设置默认值，当使用函数时没给相应的参数时，会按照默认值进行赋值。

```python
# 例2：累加计算值
# coding=gb2312
 
# 定义函数
def myadd(a=1,b=100):
  result = 0
  i = a
  while i <= b:  # 默认值为1+2+3+……+100
    result += i  
    i += 1
  return result
 
# 打印1+2+……+10    
print myadd(1,10)
print myadd()    # 使用默认参数1，100
print myadd(50)   # a赋值50，b使用默认值
   
>>> 55
>>> 5050
>>> 3825
```

Python 函数的参数传递时，值得注意的是参数传入时若为变量会被当作临时赋值给参数变量，如果是对象则会被引用。

```python
# 例3：
# coding=gb2312
 
def testpara(p1,p2):
  p1 = 10
  p2.append('hello')
 
l = []   # 定义一数组对像
a = 20   # 给变量a赋值
testpara(a,l) # 变量a与对象数组l作为参数传入
print a   # 打印运行参数后的值
for v in l: # 打印数组对象的成员
  print v
     
>>> 20    # 调用函数后a变量并未被复值
>>> hello  # 而对象l数组则增加成员hello
```

方法2: 使用 dict 和 zip  函数来实现**

```python
【按题目定义3个列表】
>>>list1 = ['neil','mike','lucy']
>>>list2 = ['123456','xuiasj==','passWD123']
>>>list3 = ['www.abc.com','www.mike.org','www.lucy.gov']
>>>fin_list = []
>>>style = ['name','passwd','url']
>>>fin_list.append(dict(zip(style,list1)))
>>>fin_list.append(dict(zip(style,list2)))
>>>fin_list.append(dict(zip(style,list3)))
>>>
>>>
>>>print(fin_list)
【这是输出的结果】
[{'name': 'neil', 'password': '123456', 'url': 'www.abc.com'}, {'name': 'mike', 'password': 'xuiasj==', 'url': 'www.mike.org'}, {'name': 'lucy', 'password': 'passWD123', 'url': 'www.lucy.gov'}]
```
