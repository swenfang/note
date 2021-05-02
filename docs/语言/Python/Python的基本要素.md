---
tags:
	- python
categories: Python 基础
title: Python 的基本要素
---
# Python 的基本要素

1. 基本数据类型
2. 对象引用
3. 组合数据类型
4. 逻辑操作符
5. 控制流语句
6. 算数操作符
7. 输入/输出
8. 函数的创建与调用

<!-- more -->

## 要素1：基本数据类型

- 任何数据语言都必须能够表示基本数据项目

- Python 中的基本数据类型有

  【1】Integral 类型 

  ​	（1）.整型：不可变类型

  ```python
  num = 1 #被保存在内存中，是不可变的
  ```

  ​	注意：对象和变量都是不可变的

  ​	（2）.布尔型  (Ture、False) 

  ​	注意：加引号 

  【2】浮点类型

  ​	（1）. 浮点数字

  ​	（2）. 复数

  【3】字符串

  ​	注意：在这里字符串表示的序列。

  ```python
  type(放任意的类型) # 输出变量的类型
  ```

  ​

## 要素2：对象引用（变量）

- Python 将所有数据存为内存对象

- Python 中，对象事实上是只想内存对象的引用

- 动态类型：在任何时刻，只要需要，某个对象都可以重新引用一个不同的对象（可以是不同的数据类型）

- 内建函数type()用于返回给定数据项的数据类型

- "=" 用于将变量名与内存中的某个对象进行绑定：如果对象事先存在，就直接绑定；否则用 “=” 创建引用的对象。

  - ![mark](https://blogimg.nos-eastchina1.126.net/180113/L99LElb9dB.png)

- 变量的命名规则

  - 只能包含字母下、数字和下划线，且不能以数字开头。
  - 区分数字大小写
  - 禁止使用保留字段（Python2和Python3有所不同）

- 命名惯例

  - 一单个下划线开头的变量名称 (_x) 不会被 from model import * 语句导入

  - 前后都有下滑线的变量名(_x _) 是系统定义的变量名称，对 Python 解释器有特殊的意义

  - 以两个下划线开头但结尾没有下划线的变量名称(__x)是类的本地变量

  - 交互模式下，变量名 “—” 用于保存最后表达式的结果

    - ```python
      >>> 1+1
      2
      >>> print(_)
      2
      ```

      ​

- 注意：

  - **变量名没有类型，对象才有**
  - 变量可以指定任何类型（这是 Python 和其他语言不同的）

## 要素3：组合数据类型

- 数据结构：通过某种方式（例如对元素进行编码）组织在一起的数据元素的集合

- Python 常用的组合数据类型

  - 序列类型

  - ![mark](https://blogimg.nos-eastchina1.126.net/180113/57KL9DI08I.png)

    - 列表：使用[]创建，如['Call','me','Ishmell','_']

      - ```python
        >>> l1 = ['Call','me','Ishmell','_']
        >>> l1[0]
        'Call'
        >>> l1[0][0]
        'C'
        ```

      - 注意：列表是可变的，可以在原处进行修改，且内容改变，id 不会改变

      - ```python
        >>> print(l1)
        ['Call', 'me', 'Ishmell', '_']
        >>> print(l1[1])
        me
        >>> l1[1] = 'your'
        >>> print(l1)
        ['Call', 'your', 'Ishmell', '_']
        >>> id(l1)
        1543556324360
        >>> l1[2]= 'Xshell'
        >>> print(l1)
        ['Call', 'your', 'Xshell', '_']
        >>> id(l1)
        1543556324360
        ```

        ​

    - 元组：使用 () 创建，如('one','two')

      - ```python
        >>> t1 = ('This','is')
        >>> t1[1]
        'is'
        >>> t1[0]
        'This'
        ```

      - 注意: 元组是不能做原处修改的，一旦修改就会引发异常

      ​

    - 字符串也属于序列类型

      - 优点：可以做字符串的切块操作

      - ```python
        >>> name = 'jerry'
        >>> name[0]
        'j'
        >>> name[0:1]
        'j'
        >>> name[0:2]
        'je'
        >>> name[:2]
        'je'
        >>> name[2:]
        'rry'
        >>> name[0:4]
        'jerr'
        >>> name[0:4:2]
        'jr'
        ```

      - 注意：切块本身会创建新的对象（因为字符串本身是不可用的）

  - 集合类型

    - 集合（杂乱的数据）

  - 映射类型

    - 字典

  - 列表是可变序列，元组是不可变序列

  - Python 中，组合数据类型也是对象，因此其可以嵌套

    - ['hello','worle',[1,2,3]]

  - 实质上，列表和元组并不是真正存储数据，而是存放对象引用

  - Python 对象可以具有其可以被调用的特定 “方法（函数）”

  - 元组、列表以及字符串等数据类型是“有大小的”，也即，其长度可用内置函数 len() 测量

## 要素4：逻辑操作符

- 逻辑运算是任何程序设计语言的基本共能

- Python 提供了4 组逻辑运算符

  - 身份操作符

    - is:判断左端对象引用是否等于右端对象引用；也可以与Node进行；

    - 对象引用可以不同，但是对象 所属的类型有可能是相同

    - ```python
      >>> name="swfswf"
      >>> test="swfswf"
      >>> type(name) is type(test)
      True
      ```

  - 比较操作符号

    - <,>,<=,>=,!=,==

  - 成员操作符

    - in 或 not in :测试成员关系

  - 逻辑运算符

    - and、or、not

## 要素5：控制流语句

- 控制流语句是过程式编程语言的基本控制机制

- Python 的常见控制流语句

  - if

    - ```python
      if boolean_expression1:
        suite1
      else if boolean_expression2:
        suite2
      .....
      else:
        else_suite
      ```

    - 注意：冒号是代码块起始的标志

  - while

    - ```python
      while boolean_expression:
        suite
      ```

  - for...in

    - ```python
      for variable in iterable:
        suite
      ```

  - try

## 要素6：算数操作符

- Python 提供了完整的算数操作符集
- 很多的 Python 数据类型也可以使用增强的赋值操作符，如+=、-= 等
- 同样的功能使用增强型赋值操作符的性能较好。
- Python 的 int 类型是不可变的，因此，增强型赋值的实际过程是创建了一个新的对象来存储结果后将变量名执行了重新绑定
  - ![mark](https://blogimg.nos-eastchina1.126.net/180113/lC2i9Jf2ag.png) 

## 要素7：输入/输出

- 现实中，具有实际共能的程序必须能够读取输入（如从键盘或文件中），以及产生输出，并写到终端或文件中

- Python 的输入/输出

  - 输出

    - Python3：print() 函数
    - Python2:  print 语句

  - 输入

    - input()

      - ```python
        >>> input("plz input a num:")
        plz input a num:a
        'a'
        >>> input("plz input a num:")
        plz input a num:3
        '3'
        >>> a = input("plz input a num:")
        plz input a num:Hello
        >>> print(a)
        Hello
        ```

    - row_input()

  - Python 解释器提供了 3 中标准的文件对象，分别为标准输入、标准输出和标准错误，它们 sys 模块中分别以 sys.stdin、sys.stdout 和 sys.stderr 形式提供

  - Python 的 print 语句实现打印

  - 从技术角度来讲，print 是把一个或多个对象转化为其文本表达形式，然后发送给标准输出或另一个类似文件的流

    - 在 Python 中，打印与文件和流的概念联系紧密

      - 文件写入方法是把字符串写入到任意文件
      - print 默认把对象打印到 stdout 流，并添加一些自动的格式化

    - 实质上，print 语句只是 Python 的人性化特性的具体实现，它提供了 sys.stdout.write() 的简单接口，再加上一些默认的格式设置

    - print 接受一个逗号分隔的对象列表，并为行尾自动添加一个换行符，如果不需要，则在最后个元素后面添加逗号

    - 实现格式化输出，print "String %format1%format2..."%(varialbe1,varialbe2,.......) 

    - | 字符   | 输出格式                       |
      | ---- | -------------------------- |
      | d,i  | 十进制整数或长整型                  |
      | u    | 无符号整数或长整型                  |
      | o    | 八进制整数或长整型                  |
      | x    | 十六进制整数或长整型                 |
      | X    | 十六进制整数                     |
      | f    | 浮点数                        |
      | e    | 浮点数                        |
      | E    | 浮点数                        |
      | g,G  | 指数小于-4或更高精度时使用%e或%E,否则使用%f |
      | s    | 字符串火任意对象，格式化代码使用str()生成字符串 |
      | r    | 同 repr() 生成的字符串            |
      | c    | 单个字符                       |
      | %    | 字面量%                       |

      ​

    - ```python
      >>> num = 7.9
      >>> print("The num is %f" %num)
      The num is 7.900000
      >>> print("The num is %d" %num)
      The num is 7
      >>> num2 = 9.13
      >>> print("The nums are %d and %f" %(num,num2))
      The nums are 7 and 9.130000
      >>> print("The nums are %e and %f"%(num,3.1))
      The nums are 7.900000e+00 and 3.100000
      >>> print("The nums are %d and %f"%(num,3.1))
      The nums are 7 and 3.100000
      >>> name = "Jerry"
      >>> print("The name is %s."%name)
      The name is Jerry.
      >>> print("The name is %s."%num)
      The name is 7.9.

      ```

    - 注意：Python 的输出是需要转换的。 

    - 数据转换类型：

      - 显示转换

      - ```python
        >>> num = 7.9
        >>> test3 = str(num)
        >>> type(test3)
        <class 'str'>
        >>> type(num)
        <class 'float'>
        ```

      - 隐式转换

    - 想知道内置有多少种类型

      - python3

      - ```python
        >>> dir()
        ['__annotations__', '__builtins__', '__doc__', '__loader__', '__name__', '__package__', '__spec__', 'name', 'num', 'num2', 'test3']
        >>> dir(__builtins__)
        ```

      - python2

      - ```python
        >>>dir(builtins)
        ['__annotations__', '__builtins__', '__doc__', '__loader__', '__name__', '__package__', '__spec__', 'name', 'num', 'num2', 'test3']
        >>> dir(__builtins__)
        ```

    - 想明确一个工具怎么使用

      - ```python
        >>> help(str)
        Help on class str in module builtins:

        class str(object)
         |  str(object='') -> str
         |  str(bytes_or_buffer[, encoding[, errors]]) -> str
         |  
         |  Create a new string object from the given object. If encoding or
         |  errors is specified, then the object must expose a data buffer
         |  that will be decoded using the given encoding and error handler.
         ....
        ```

  - % 后面可以使用的修饰符，（如果有只能按如下的顺序）

    - ```python
      %[(name)][flags][width][.precision]typecode
      ```

      - 位于括号中的一个属性后面的字典的键名，用于选出一个具体项

      - 下面标志中的一个或多个

        - 减号（-）:表示左对齐，默认为右对齐

          - ```python
            >>> print("The name are %+10f and %+f"%(num,-3.1))
            The name are  +7.900000 and -3.100000
            >>> print("The name are %-20f and %+f"%(num,-3.1))
            The name are 7.900000             and -3.100000
            ```

            ​

        - 加号（+）:表示包含数字符号，整数也会带 "+"

          - ```python
            >>>num=7.9
            >>> print("The name are %+e and %f"%(num,3.1))
            The name are +7.900000e+00 and 3.100000
            >>> print("The name are %+e and %+f"%(num,3.1))
            The name are +7.900000e+00 and +3.100000
            >>> print("The name are %+e and %f"%(num,-3.1))
            The name are +7.900000e+00 and -3.100000
            >>> print("The name are %+e and %+f"%(num,-3.1))
            The name are +7.900000e+00 and -3.100000
            >>> print("The name are %f and %f"%(num,-3.1))
            The name are 7.900000 and -3.100000
            ```

        - 零（0）:表示一个零填充

      - 一个指定最小宽度的数

      - 一个小数,用于按照精度分割字段的宽度(如下例子：15指小数占15位数)

        - ```python
          >>> print("The name are %-20.15f and %+f"%(num,-3.1))
          The name are 7.900000000000000    and -3.100000
          ```

      - 一个数字,指定要打印字符串中的最大字符个数，浮点数中小数点之后的位数，或者整数最小位数。

      - 字典：kv集合（键值对集合）

        - ```python
          >>> d1={'a':33,'b':66}
          >>> d1['a']
          33
          >>> d1={0:33,1:66}	
          >>> d1[0]
          33
          ```

        - 注意字典也是可变对象;键可以是字符也可以是数字

        - ```python
          >>> d={'x':32,'y':27.490325,'z':65}
          	
          >>> print("%(x)-10d %(y)0.3g"%d)
          	
          32         27.5
          ```

          ​

## 要素8：函数的创建与调用

- 函数实现模块化编程的组件

- Python 使用 def 语句定义函数

  - ```python
    def functionName(arguments):
      suite
    ```

- 函数可以参数化，通过传递不同的参数来调用。

  - ```python
    >>> def printName(name):
    	print(name)
    >>> printName('Tony')
    Tony
    ```

  - 注意：函数也是对象

- 每个 Python 函数都有一个返回值，默认为None,也可以使用 “return value” 明确定义返回值

- def 会创建一个函数对象，并同时创建一个函数的对象引用

  - 函数也是对象，可以存储在组合数据类型中，也可以作为参数传递给其他函数

  - callable() 可用于测试函数是否可调用

    - ```python
      >>> callable(name)
      False
      >>> callable(printName)
      True
      ```

- Python 有众多内置函数

  - dir()、id()、type() 、str()、help()、len()、callable() 等等

- Python 标准库拥有众多内置模块，这些模块拥有大量的函数

  - Python 模块实际上是包含 Python 代码的 .py 文件，其拥有自定义的函数与类及变量等
  - 导入代码块使用 import 语句进行，后跟模块名称（不能指定模块文件的后缀.py）
  - 导入一个模块后，可以访问其内部的任意函数、类及变量
