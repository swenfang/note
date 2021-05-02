---
tags:
	- 基础面试题
categories: 基础面试题
title: Java选择题【1~60】
---
# Java选择题【1~60】

原文地址：http://blog.csdn.net/qq_36075612/article/details/71126487

1.下面中哪两个可以在A的子类中使用：（ ）

```java
class A {

protected int method1 (int a, int b) {

return 0;

}

}
```

A. public int method 1 (int a, int b) { return 0; }

B. private int method1 (int a, int b) { return 0; }

C. private int method1 (int a, long b) { return 0; }

D. public short method1 (int a, int b) { return 0; }

解答：AC

**主要考查子类重写父类的方法的原则**

B，子类重写父类的方法，访问权限不能降低

C，属于重载

D，子类重写父类的方法 返回值类型要相同或是父类方法返回值类型的子类

2.Abstract method cannot be static. True or False ?

A True

B False

解答：A

抽象方法可以在子类中被重写，但是静态方法不能在子类中被重写，静态方法和静态属性与对象是无关的，只与类有关，这与abstract是矛盾的，所以abstract是不能被修饰为static，否则就失去了abstract的意义了

3.What will be the output when you compile and execute the following program.

```java
class Base

{

void test() {

System.out.println(“Base.test()”);

}

}

public class Child extends Base {

void test() {

System.out.println(“Child.test()”);

}

static public void main(String[] a) {

Child anObj = new Child();

Base baseObj = (Base)anObj;

baseObj.test();

}

}
```

Select most appropriate answer.

A . Child.test()

Base.test()

B. Base.test()

Child.test()

C. Base.test()

D. Child.test()

解答：D

测试代码相当于：Base baseObj = new Child();父类的引用指向子类的实例，子类又重写了父类

的test方法，因此调用子类的test方法。

4.What will be the output when you compile and execute the following program.

```java
class Base

{

static void test() {

System.out.println(“Base.test()”);

}

}

public class Child extends Base {

void test() {

System.out.println(“Child.test()”);

Base.test(); //Call the parent method

}

static public void main(String[] a) {

new Child().test();

}

}

```

Select most appropriate answer.

A . Child.test() 、Base.test()

B . Child.test()、Child.test()

C.  Compilation error. Cannot override a static method by an instance method

D. Runtime error. Cannot override a static method by an instance method

解答：C

静态方法不能在子类中被重写

5.What will be the output when you compile and execute the following program.

```java
public class Base{

private void test() {

System.out.println(6 + 6 + “(Result)”);

}

static public void main(String[] a) {

new Base().test();

}

}
```

Select most appropriate answer.

A.  66(Result)

B . 12(Result)

C . Runtime Error.Incompatible type for +. Can’t convert an int to a string.

D . Compilation Error.Incompatible type for +. Can’t add a string to an int.

解答：B

字符串与基本数据类型链接的问题,如果第一个是字符串那么后续就都按字符串处理，比如上边例子要是System.out.println(“(Result)”+6 + 6 );那么结果就是(Result)66，如果第一个和第二个。。。第n个都是基本数据第n+1是字符串类型，那么前n个都按加法计算出结果在与字符串连接

6..What will be the output when you compile and execute the following program. The symbol ’ ?’ means space.

```java
1:public class Base{

2:

3: private void test() {

4:

5: String aStr = “?One?”;

6: String bStr = aStr;

7: aStr.toUpperCase();

8: aStr.trim();

9: System.out.println(“[" + aStr + "," + bStr + "]“);

7: }

8:

9: static public void main(String[] a) {

10: new Base().test();

11: }

12: }

```

Select most appropriate answer.

A.  [ONE,?One?]

B . [?One?,One]

C . [ONE,One]

D . [ONE,ONE]

E . [?One?,?One?]

解答：E

通过 **String bStr = aStr;** 这句代码使 bStr 和 aStr 指向同一个地址空间，所以最后 aStr 和 bStr 的结果应该是一样，String 类是定长字符串，调用一个字符串的方法以后会形成一个新的字符串。

7.下面关于**变量**及其**范围**的陈述哪些是不正确的（ ）：

A．实例变量是类的成员变量

B．实例变量用关键字static声明

C．在方法中定义的局部变量在该方法被执行时创建

D．局部变量在使用前必须被初始化

解答：BC

由static修饰的变量称为类变量或是静态变量

方法加载的时候创建局部变量

8.下列关于**修饰符混用**的说法，错误的是（ ）：

A．abstract不能与final并列修饰同一个类

B．abstract类中可以有private的成员

C．abstract方法必须在abstract类中

D．static方法中能处理非static的属性

解答 D

**静态方法中不能引用非静态的成员**

9.执行完以下代码 int [ ] x = new int[25]; 后，以下哪项说明是正确的（ ）：

A、 x[24]为0

B、 x[24]未定义

C、 x[25]为0

D、 x[0]为空

解答：A

x 属于引用类型，该引用类型的每一个成员是 int 类型，默认值为：0

10.编译运行以下程序后，关于输出结果的说明正确的是 （ ）：

```java
public class Conditional{

public static void main(String args[ ]){

int x=4;

System.out.println(“value is “+ ((x>4) ? 99.9 :9));

}

}

```

A、 输出结果为：value is 99.99

B、 输出结果为：value is 9

C、 输出结果为：value is 9.0

D、 编译错误

解答：C

三目运算符中：第二个表达式和第三个表达式中如果都为基本数据类型，整个表达式的运算结果

由容量高的决定。99.9是double类型 而9是int类型，double容量高。

11.关于以下 application 的说明，正确的是（ ）：

```java
1． class StaticStuff

2． {

3． static int x=10；

4． static { x+=5；}

5． public static void main（String args[ ]）

6． {

7． System.out.println(“x=” + x);

8． }

9． static { x/=3;}

10. }

```

A、 4行与9行不能通过编译，因为缺少方法名和返回类型

B、 9行不能通过编译，因为只能有一个静态初始化器

C、 编译通过，执行结果为：x=5

D、编译通过，执行结果为：x=3

解答：C

自由块是类加载的时候就会被执行到的，自由块的执行顺序是按照在类中出现的先后顺序执行。

12.关于以下程序代码的说明正确的是（ ）：

```java
1．class HasStatic{

2． private static int x=100；

3． public static void main(String args[ ]){

4． HasStatic hs1=new HasStatic( );

5． hs1.x++;

6． HasStatic hs2=new HasStatic( );

7． hs2.x++;

8． hs1=new HasStatic( );

9． hs1.x++;

10． HasStatic.x–;

11． System.out.println(“x=”+x);

12． }

13．}

```

A、5行不能通过编译，因为引用了私有静态变量

B、10行不能通过编译，因为x是私有静态变量

C、程序通过编译，输出结果为：x=103

D、程序通过编译，输出结果为：x=102

解答：D

静态变量是所有对象所共享的，所以上述代码中的几个对象操作是同一静态变量x， 静态变量可以通过类名调用。

13.下列说法正确的有（）

A． class 中的constructor不可省略

B． constructor 必须与 class同名，但方法不能与class同名

C． constructor 在一个对象被 new 时执行

D．一个 class 只能定义一个constructor

解答：C

构造方法的作用是在实例化对象的时候给数据成员进行初始化

A．类中如果没有显示的给出构造方法，系统会提供一个无参构造方法

B．构造方法与类同名，类中可以有和类名相同的方法

D．构造方法可以重载

14.下列哪种说法是正确的（）

A．实例方法可直接调用超类的实例方法

B．实例方法可直接调用超类的类方法

C．实例方法可直接调用其他类的实例方法

D．实例方法可直接调用本类的类方法

解答：D

A. 实例方法不可直接调用超类的私有实例方法

B. 实例方法不可直接调用超类的私有的类方法

C．要看访问权限

15.下列哪一种叙述是正确的（ ）

A． abstract修饰符可修饰字段、方法和类

B． 抽象方法的body部分必须用一对大括号{ }包住

C． 声明抽象方法，大括号可有可无

D． 声明抽象方法不可写出大括号

解答：D

abstract可以修饰方法和类，不能修饰属性。抽象方法没有方法体，即没有大括号{}

16.下面代码的执行结果是？

```java
import java.util.*;

public class ShortSet{

public static void main(String args[])

{

Set<Short> s=new HashSet<Short>();

for(Short i=0;i<100;i++)

{

s.add(i);

s.remove(i-1);

}

System.out.println(s.size());

}

}
```

A. 1

B. 100

C. Throws Exception

D. None of the Above

解答：B

i 是 Short 类型 i-1 是int类型,其包装类为 Integer ，所以 s.remove(i-1); 不能移除Set集合中Short类型对象。

17.链表具有的特点是：(选择3项)

A、不必事先估计存储空间

B、可随机访问任一元素

C、插入删除不需要移动元素

D、所需空间与线性表长度成正比

解答：ACD

A.采用动态存储分配，不会造成内存浪费和溢出。

B. 不能随机访问，查找时要从头指针开始遍历

C. 插入、删除时，只要找到对应前驱结点，修改指针即可，无需移动元素

D. 需要用额外空间存储线性表的关系，存储密度小

18.Java语言中，String类的IndexOf()方法返回的类型是？

A、Int16 B、Int32 C、int D、long

解答：C

indexOf方法的声明为：public int indexOf(int ch)

在此对象表示的字符序列中第一次出现该字符的索引；如果未出现该字符，则返回 -1。

19.以下关于面向对象概念的描述中，不正确的一项是（）。(选择1项)

A.在现实生活中，对象是指客观世界的实体

B.程序中的对象就是现实生活中的对象

C.在程序中，对象是通过一种抽象数据类型来描述的，这种抽象数据类型称为类（class）

D.在程序中，对象是一组变量和相关方法的集合

解答：B

20..执行下列代码后,哪个结论是正确的 String[] s=new String[10];

A． s[9] 为 null;

B． s[10] 为 “”;

C． s[0] 为 未定义

D． s.length 为10

解答：AD

s是引用类型，s中的每一个成员都是引用类型，即String类型，String类型默认的值为null

s数组的长度为10。

21.属性的可见性有。(选择3项)

A.公有的

B.私有的

C.私有保护的

D.保护的

解答：ABD

属性的可见性有四种：公有的（public） 保护的（protected） 默认的 私有的（private）

22..在字符串前面加上_____符号，则字符串中的转义字符将不被处理。(选择1项)

A @

B \

C #

D %

解答：B

23.下列代码哪行会出错: (选择1项)

```java
1) public void modify() { 
2) int I, j, k; 
3) I = 100; 
4) while ( I > 0 ) { 
5) j = I * 2; 
6) System.out.println (” The value of j is ” + j ); 
7) k = k + 1; 
8) I–; 
9) } 
10) }
```

A. 4

B. 6

C. 7

D. 8

解答：C

k没有初始化就使用了

24.对记录序列{314，298，508，123，486，145}按从小到大的顺序进行插入排序，经过两趟排序后的结果为：(选择1项)

A {314，298，508，123，145，486}

B {298，314，508，123，486，145}

C {298，123，314，508，486，145}

D {123、298，314，508，486，145}

解答：B

插入排序算法：

```java
public static void injectionSort(int[] number) {

// 第一个元素作为一部分，对后面的部分进行循环

for (int j = 1; j < number.length; j++) {

int tmp = number[j];

int i = j – 1;

while (tmp < number[i]) {

number[i + 1] = number[i];

i–;

if (i == -1)

break;

}

number[i + 1] = tmp;

}

}
```

25.栈是一种。(选择1项)

A 存取受限的线性结构

B 存取不受限的线性结构

C 存取受限的非线性结构

D 存取不受限的非线性结构

解答：A

栈（stack）在计算机科学中是限定仅在表尾进行插入或删除操作的线性表。

26.下列哪些语句关于内存回收的说明是正确的。(选择1项)

A. 程序员必须创建一个线程来释放内存

B. 内存回收程序负责释放无用内存

C. 内存回收程序允许程序员直接释放内存

D. 内存回收程序可以在指定的时间释放内存对象

解答：B

垃圾收集器在一个Java程序中的执行是自动的，不能强制执行，即使程序员能明确地判断出有一块内存已经无用了，是应该回收的，程序员也不能强制垃圾收集器回收该内存块。程序员唯一能做的就是通过调用System. gc 方法来”建议”执行垃圾收集器，但其是否可以执行，什么时候执行却都是不可知的。

27.Which method must be defined by a class implementing the java.lang.Runnable interface?

A. void run()

B. public void run()

C. public void start()

D. void run(int priority)

E. public void run(int priority)

F. public void start(int priority)

解答：B

实现Runnable接口，接口中有一个抽象方法run，实现类中实现该方法。

28 Given:

```java
public static void main(String[] args) {

Object obj = new Object() {

public int hashCode() {

return 42;

}

};

System.out.println(obj.hashCode());

}
```

What is the result?

A. 42

B. An exception is thrown at runtime.

C. Compilation fails because of an error on line 12.

D. Compilation fails because of an error on line 16.

E. Compilation fails because of an error on line 17.

解答：A

匿名内部类覆盖hashCode方法。

29.哪两个是Java编程语言中的保留字 ? (Choose two)

A. run

B. import

C. default

D. implements

解答：BD

import导入包的保留字，implements实现接口的保留字。

30. Which two statements are true regarding the return values of property written hashCodeand equals methods from two instances of the same class? (Choose two)

A. If the hashCode values are different, the objects might be equal.

B. If the hashCode values are the same, the object must be equal.

C. If the hashCode values are the same, the objects might be equal.

D. If the hashCode values are different, the objects must be unequal.

解答：CD

先通过 hashcode 来判断某个对象是否存放某个桶里，但这个桶里可能有很多对象，那么我们就需要再通过 equals 来在这个桶里找到我们要的对象。

31. 字符的数字范围是什么?

A. 0 … 32767

B. 0 … 65535

C. –256 … 255

D. –32768 … 32767

E. Range is platform dependent.

解答：B

在Java中，char是一个无符号16位类型，取值范围为0到65535。

32. Given:

```java
public class Test {

private static float[] f = new float[2];

public static void main(String args[]) {

System.out.println(“f[0] = “ + f[0]);

}

}

```

What is the result?

A. f[0] = 0

B. f[0] = 0.0

C. Compilation fails.

D. An exception is thrown at runtime.

解答：B

33. Given:

```java
public class Test {

public static void main(String[] args) {

String str = NULL;

System.out.println(str);

}

}
```

What is the result?

A. NULL

B. 编译失败

C. 运行代码没有输出

D. 运行时抛出异常

解答：B

null应该小写

34、Exhibit:

```java
1.public class X implements Runnable {

2. private int x;

3. private int y;

4. public static void main(String [] args) {

5. X that = new X();

6. (new Thread(that)).start();

7. (new Thread(that)).start();

8. }

9. public synchronized void run( ){

10. for (;;) {

11. x++;

12. y++;

13. System.out.println(“x = “ + x + “, y = “ + y);

14. }

15. }

16.}

```

What is the result?

A. 第11行的错误导致编译失败

B. 第7行和第8行的错误导致编译失败。

C. 该程序打印 x 和 y 的值对，这些值在同一行上可能不总是相同的 (例如,  “x=2, y=1”)

D. 该程序在同一行上打印x和y的值总是一对相同的值(例如, “x=1, y=1”. 此外,每个值出现两次(例如, “x=1, y=1” 其次是“x=1, y=1”)

E. 该程序在同一行上打印x和y的值总是一对相同的值 (例如,  “x=1, y=1”.此外,每个值出现两次 (例如, “x=1, y=1” 其次 “x=2, y=2”)

解答：E

多线程共享相同的数据，使用 synchronized 实现数据同步。

35、哪两个不能直接导致线程停止执行? (Choose Two)

A. 使用 synchronized block.

B. 在对象上调用 wait 方法 

C. 调用对象的 notify 方法

D. 在InputStream对象上调用read方法。

E. 调用Thread对象上的SetPriority方法。

解答：AD

stop方法.这个方法将终止所有未结束的方法，包括run方法。当一个线程停止时候，他会立即释

放 所有他锁住对象上的锁。这会导致对象处于不一致的状态。 当线程想终止另一个线程的时

候，它无法知道何时调用stop是安全的，何时会导致对象被破坏。所以这个方法被弃用了。你应

该中断一个线程而不是停止他。被中断的线程会在安全的时候停止。

36、 关于创建默认构造函数描述正确的是? (Choose Two)

A.  默认构造函数初始化方法变量

B.  默认构造函数调用超类的无参数构造函数

C.  默认构造函数初始化在类中声明的实例变量

D.  如果一个类缺少一个无参数构造函数，但有其他构造函数，编译器会创建一个默认构造函数。

E.  编译器只有在没有其他构造函数是才会创建构造函数

解答：CE

构造方法的作用是实例化对象的时候给数据成员初始化，如果类中没有显示的提供构造方法，系统会提供默认的无参构造方法，如果有了其它构造方法，默认的构造方法不再提供。

37、 Given:

```java
public class OuterClass {

private double d1 = 1.0;

//insert code here

}
```

您需要在第2行插入内部类声明。 哪两个内部类声明是有效的？

A. static class InnerOne { public double methoda() {return d1;} }

B. static class InnerOne { static double methoda() {return d1;} }

C. private class InnerOne { public double methoda() {return d1;} }

D. protected class InnerOne { static double methoda() {return d1;} }

E. public abstract class InnerOne { public abstract double methoda(); }

解答：CE

AB.内部类可以声明为static的，但此时就不能再使用外层封装类的非static的成员变量；

D.非static的内部类中的成员不能声明为static的，只有在顶层类或static的内部类中

才可声明static成员

38、哪两个声明可以防止重写方法？ (Choose Two)

A. final void methoda() {}

B. void final methoda() {}

C. static void methoda() {}

D. static final void methoda() {}

E. final abstract void methoda() {}

解答：AD

final修饰方法，在子类中不能被重写。

39、Given:

```java
public class Test {

public static void main (String args[]) {

class Foo {

public int i = 3;

}

Object o = (Object) new Foo();

Foo foo = (Foo)o;

System.out.println(foo.i);

}

}

```

What is the result?

A. 编译将失败。

B. 编译将成功，程序将打印“3”

C. 编译会成功，但程序会在第6行抛出ClassCastException。

D. 编译会成功，但程序会在第7行抛出ClassCastException。

解答：B

局部内部类的使用

40、 Given:

```java
public class Test {

public static void main (String [] args) {

String foo = “blue”;

String bar = foo;

foo = “green”;

System.out.println(bar);

}

}
```

What is the result?

A. 抛出异常。

B. 代码不会编译。

C. 该程序打印“null”

D. 该程序打印“blue”

E. 该程序打印“green”

解答：D

采用String foo = “blue”定义方式定义的字符串放在字符串池中，通过String bar = foo;

他们指向了同一地址空间，就是同一个池子，当执行foo = “green”; foo指向新的地址空间。

41、Which code determines the int value foo closest to a double value bar?

A. int foo = (int) Math.max(bar);

B. int foo = (int) Math.min(bar);

C. int foo = (int) Math.abs(bar);

D. int foo = (int) Math.ceil(bar);

E. int foo = (int) Math.floor(bar);

F. int foo = (int) Math.round(bar);

解答：DEF

A B两个选项方法是用错误，都是两个参数。

abs方法是取bar的绝对值，

ceil方法返回最小的（最接近负无穷大）double 值，该值大于等于参数，并等于某个整数。

floor方法返回最大的（最接近正无穷大）double 值，该值小于等于参数，并等于某个整数。

round方法 返回最接近参数的 long。

42、 Exhibit:

```java
1.package foo;

2.import java.util.Vector;

3.private class MyVector extends Vector {

4.int i = 1;

5.public MyVector() {

6.i = 2;

7. }

8.}

9.public class MyNewVector extends MyVector {

10.public MyNewVector () {

11. i = 4;

12.}

13.public static void main (String args []) {

14.MyVector v = new MyNewVector();

15. }

16.}

```

The file MyNewVector.java is shown in the exhibit. What is the result?

A. 汇编将会成功。

B. 编译到第3行的时候失败。

C. 编译到第6行的时候失败。

D. 编译到第9行的时候失败。

E. 编译到第14行的时候失败。

解答：B

类MyVector不能是私有的

43、Given:

```java
public class Test {

public static void main (String[]args) {

String foo = args[1];

String bar = args[2];

String baz = args[3];

System.out.println(“baz = ” + baz);

}

}

And the output:

Baz = 2

```

Which command line invocation will produce the output?

A. java Test 2222

B. java Test 1 2 3 4

C. java Test 4 2 4 2

D. java Test 4 3 2 1

解答：C

数组下标从0开始

44、 Given:

```java
1. public interface Foo{

2.int k = 4;

3. }
```

哪三个相当于第2行? (Choose Three)

A. final int k = 4;

B. Public int k = 4;

C. static int k = 4;

D. Private int k = 4;

E. Abstract int k = 4;

F. Volatile int k = 4;

G. Transient int k = 4;

H. protected int k = 4;

解答：BDE

static：修饰的静态变量

final 修饰的是常量

abstract不能修饰变量

Volatile修饰的成员变量在每次被线程访问时，都强迫从共享内存中重读该成员变量的值。

而且，当成员变量发生变化时，强迫线程将变化值回写到共享内存。这样在任何时刻，

两个不同的线程总是看到某个成员变量的同一个值。

Transient：对不需序列化的类的域使用transient关键字,以减少序列化的数据量。

int k=4相当于public static final int k=4; 在接口中可以不写static final

45、 Given:

```java
public class foo {

static String s;

public static void main (String[]args) {

System.out.println (“s=” + s);

}

}

```

What is the result?

A. 代码编译并打印“s =”。

B. 代码编译并打印“s = null”。

C. 该代码不会编译，因为字符串s未初始化。

D.代码不编译，因为字符串不能被引用。

E. 代码编译，但调用toString时会引发NullPointerException。

解答：B

String为禁用数据类型，引用类型数据成员的默认值为null

46、哪两个创建一个数组的实例? (Choose Two)

A. int[] ia = new int [15];

B. float fa = new float [20];

C. char[] ca = “Some String”;

D. Object oa = new float[20];

E. Int ia [][] = (4, 5, 6) (1, 2, 3)

解答：AD

任何类的父类都是 Object，数组也有数据引用类型，Object oa = new float[20];这种写法相当于父类的用指向之类的实例。

47、Given:

```java
public class ExceptionTest {

class TestException extends Exception {}

public void runTest () throws TestException {}

public void test () /* Point X*/ {

runTest ();

}

}
```

在第4行的X点上，可以添加哪些代码来编译代码？

A. throws Exception

B. Catch (Exception e).

C. Throws RuntimeException.

D. Catch (TestException e).

E. No code is necessary.

解答：A

方法上使用throws抛出异常，Exception是异常类的超类。

48、Exhibit:

```java
public class SwitchTest {

public static void main (String []args) {

System.out.println (“value =” +switchIt(4));

}

public static int switchIt(int x) {

int j = 1;

switch (x) {

case 1: j++;

case 2: j++;

case 3: j++;

case 4: j++;

case 5: j++;

default:j++;

}

return j + x;

}

}
```

第3行的输出是什么?

A. Value =3

B. Value =4

C. Value =5

D. Value =6

E. Value =7

F. Value =8

解答：F

由于case块没有break语句，那么从case 4：向下的代码都会执行。

49、使用 throw 语句可以抛出哪四种类型的对象? (Choose Four)

A. Error

B. Event

C. Object

D. Exception

E. Throwable

F. RuntimeException

解答：ADEF

能够抛出的对象类型要是 Throwable  或是 Throwable 的子类

50．在下面程序的第6行补充上下列哪个方法,会导致在编译过程中发生错误?

```java
1) class Super{

2) public float getNum(){

3) return 3.0f;

4) }

}

5) pubhc class Sub extends Super{

6)

7) }
```

A. public float getNum(){retun 4.0f;}

B. public void getNum(){}

C. public void getNum(double d){}

D. public double getNum(float d){ retun 4.0f ;} 

解答：B

方法重写的问题。子类中有和父类的方法名相同，但是参数不同，不会出编译错误，认为是子类

的特有的方法，但是如果子类中方法和父类的方法名，参数，访问权限，异常都相同，只有返回值

类型不同会编译不通过。

51.下面关于import, class和package的声明顺序哪个正确？( )

A. package, import, class

B. class, import, package

C. import, package, class

D. package, class, import

解答：A

52.下面哪个是正确的？( )

A. String temp [] = new String {“a” “b” “c”};

B. String temp [] = {“a” “b” “c”}

C. String temp = {“a”, “b”, “c”}

D. String temp [] = {“a”, “b”, “c”}

解答：D

53.关于java.lang.String类，以下描述正确的一项是（ ）

A. String类是final类故不可以继承；

B. String类是final类故可以继承；

C. String类不是final类故不可以继承；

D. String类不是final类故可以继承；

 解答：A

String类是final的，在java中final修饰类的不能被继承

54.关于实例方法和类方法，以下描述正确的是：( )

A. 实例方法只能访问实例变量

B. 类方法既可以访问类变量，也可以访问实例变量

C. 类方法只能通过类名来调用

D. 实例方法只能通过对象来调用

解答：D

A 实例方法可以访问类变量

B类方法只能访问类变量

C类方法可以通过对象调用

55.接口是Java面向对象的实现机制之一，以下说法正确的是：( )

A. Java支持多重继承，一个类可以实现多个接口；

B. Java只支持单重继承，一个类可以实现多个接口；

C. Java只支持单重继承，一个类只可以实现一个接口；

D. Java支持多重继承，但一个类只可以实现一个接口。

解答：B

Java支持单重继承，一个类只能继承自另外的一个类，但是一个类可以实现多个接口。

56.下列关于 interface 的说法正确的是：( )

A. interface中可以有private方法

B. interface中可以有final方法

C. interface中可以有function实现

D. interface可以继承其他interface

解答：D

A.   接口中不可以有private的方法

B．接口中不可以有final的方法 接口中的方法默认是 public abstract的

C．接口中的方法不可以有实现

57.已知A类被打包在packageA , B类被打包在packageB ，且B类被声明为public ，且有一个成员变量x被声明为, protected控制方式 。C类也位于packageA包，且继承了B类 。则以下说话正确的是（ ）

A. A类的实例不能访问到B类的实例

B. A类的实例能够访问到B类一个实例的x成员

C. C类的实例可以访问到B类一个实例的x成员

D. C类的实例不能访问到B类的实例

解答：C

不同包子类的关系， 可以访问到父类B的protected成员

58.以下程序正确的输出是（ ）

```java
package test;

public class FatherClass {

public FatherClass() {

System.out.println(“FatherClass Create”);

}

}

package test;

import test.FatherClass;

public class ChildClass extends FatherClass {

public ChildClass() {

System.out.println(“ChildClass Create”);

}

public static void main(String[] args) {

FatherClass fc = new FatherClass();

ChildClass cc = new ChildClass();

}

}
```

A.

FatherClass Create

FatherClass Create

ChildClass Create

B.

FatherClass Create

ChildClass Create

FatherClass Create

C.

ChildClass Create

ChildClass Create

FatherClass Create

D.

ChildClass Create

FatherClass Create

FatherClass Create

解答：A

在子类构造方法的开始默认情况下有一句super();来调用父类的构造方法

59.给定如下代码，下面哪个可以作为该类的构造函数 ( )

```
public class Test {

}
```

A. public void Test() {?}

B. public Test() {?}

C. public static Test() {?}

D. public static void Test() {?}

解答：B

构造方法：与类同名没有放回类型

60.题目:

```java
1. public class test (

2. public static void main (String args[]) {

3. int i = 0xFFFFFFF1;

4. int j = -i;

5.

6. }

7. )

```

程序运行到第5行时,j的值为 多少?( )

A. –15

B. 0

C. 1

D. 14

E. 在第三行的错误导致编译失败

解答：D

int i = 0xFFFFFFF1;相当于 int i=-15 然后对i进行取反即取绝对值再减一