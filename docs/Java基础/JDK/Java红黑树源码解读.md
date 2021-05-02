---
tags:
	- 数据结构
categories: 红黑树
title: Java 红黑树源码解读
---

#  Java 红黑树源码解读

## 目录

### **红黑树的介绍**

红黑树(Red-Black Tree，简称R-B Tree)，它**一种特殊的二叉查找树**。
红黑树是特殊的二叉查找树，意味着它**满足二叉查找树的特征:**

**任意一个节点所包含的键值，大于等于左孩子的键值，小于等于右孩子的键值。**
除了具备该特性之外，红黑树还包括许多额外的信息。

红黑树的每个节点上都有存储位表示节点的颜色，颜色是红(Red)或黑(Black)。
红黑树的特性:
【1】 **每个节点或者是黑色，或者是红色。**
【2】 **根节点是黑色。**
【3】 **每个叶子节点是黑色。 [注意：这里叶子节点，是指为空的叶子节点！]**
【4】 **如果一个节点是红色的，则它的子节点必须是黑色的。**
【5】 **从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。**

关于它的特性，需要注意的是：
第一，特性(3)中的叶子节点，是只为空(NIL或null)的节点。
第二，特性(5)，**确保没有一条路径会比其他路径长出俩倍**。因而，红黑树是相对是接近平衡的二叉树。

红黑树示意图如下：

[![img](http://images.cnitblog.com/i/497634/201403/251730074203156.jpg)](http://images.cnitblog.com/i/497634/201403/251730074203156.jpg)



### 红黑树的原理

在研究红黑树原理，我们可以通过以下的实例进行 debug 调试，看它的整个执行过。接着再阅读以下内容，会比较容易理解红黑的原理，才能进一步深入研究清楚。

**实例地址：https://github.com/KnIfER/RBTree-java**



### **红黑树的实现(代码说明)**

红黑树的基本操作是**添加**、**删除**和**旋转**。在对红黑树进行添加或删除后，会用到旋转方法。为什么呢？道理很简单，添加或删除红黑树中的节点之后，红黑树就发生了变化，可能不满足红黑树的5条性质，也就不再是一颗红黑树了，而是一颗普通的树。而通过旋转，可以使这颗树重新成为红黑树。简单点说，**旋转的目的是让树保持红黑树的特性。**
旋转包括两种：**左旋** 和 **右旋**。下面分别对红黑树的基本操作进行介绍。

 

#### **1. 基本定义**

```java
public class RBTree<T extends Comparable<T>> {

    private RBTNode<T> mRoot;    // 根结点

    private static final boolean RED   = false;
    private static final boolean BLACK = true;

    public class RBTNode<T extends Comparable<T>> {
        boolean color;        // 颜色
        T key;                // 关键字(键值)
        RBTNode<T> left;    // 左孩子
        RBTNode<T> right;    // 右孩子
        RBTNode<T> parent;    // 父结点

        public RBTNode(T key, boolean color, RBTNode<T> parent, RBTNode<T> left, RBTNode<T> right) {
            this.key = key;
            this.color = color;
            this.parent = parent;
            this.left = left;
            this.right = right;
        }

    }

    ...
}

```

RBTree是红黑树对应的类，RBTNode是红黑树的节点类。在RBTree中包含了根节点mRoot和红黑树的相关API。
注意：在实现红黑树API的过程中，我重载了许多函数。重载的原因，一是因为有的API是内部接口，有的是外部接口；二是为了让结构更加清晰。

 

#### **2. 左旋**

[![img](http://images.cnitblog.com/i/497634/201403/251733282013849.jpg)](http://images.cnitblog.com/i/497634/201403/251733282013849.jpg)

对x进行左旋，意味着"将x变成一个左节点"。

 

左旋的实现代码(Java语言)

```java
/* 
 * 对红黑树的节点(x)进行左旋转
 *
 * 左旋示意图(对节点x进行左旋)：
 *      px                              px
 *     /                               /
 *    x                               y                
 *   /  \      --(左旋)-.           / \                #
 *  lx   y                          x  ry     
 *     /   \                       /  \
 *    ly   ry                     lx  ly  
 *
 *
 */
private void leftRotate(RBTNode<T> x) {
    // 设置x的右孩子为y
    RBTNode<T> y = x.right;

    // 将 “y的左孩子” 设为 “x的右孩子”；
    // 如果y的左孩子非空，将 “x” 设为 “y的左孩子的父亲”
    x.right = y.left;
    if (y.left != null)
        y.left.parent = x;

    // 将 “x的父亲” 设为 “y的父亲”
    y.parent = x.parent;

    if (x.parent == null) {
        this.mRoot = y;            // 如果 “x的父亲” 是空节点，则将y设为根节点
    } else {
        if (x.parent.left == x)
            x.parent.left = y;    // 如果 x是它父节点的左孩子，则将y设为“x的父节点的左孩子”
        else
            x.parent.right = y;    // 如果 x是它父节点的左孩子，则将y设为“x的父节点的左孩子”
    }
    
    // 将 “x” 设为 “y的左孩子”
    y.left = x;
    // 将 “x的父节点” 设为 “y”
    x.parent = y;
}

```

#### **3. 右旋**

[![img](http://images.cnitblog.com/i/497634/201403/251735527958942.jpg)](http://images.cnitblog.com/i/497634/201403/251735527958942.jpg)

对y进行左旋，意味着"将y变成一个右节点"。

 

右旋的实现代码(Java语言)

```java
/* 
 * 对红黑树的节点(y)进行右旋转
 *
 * 右旋示意图(对节点y进行左旋)：
 *            py                               py
 *           /                                /
 *          y                                x                  
 *         /  \      --(右旋)-.            /  \                     #
 *        x   ry                           lx   y  
 *       / \                                   / \                   #
 *      lx  rx                                rx  ry
 * 
 */
private void rightRotate(RBTNode<T> y) {
    // 设置x是当前节点的左孩子。
    RBTNode<T> x = y.left;

    // 将 “x的右孩子” 设为 “y的左孩子”；
    // 如果"x的右孩子"不为空的话，将 “y” 设为 “x的右孩子的父亲”
    y.left = x.right;
    if (x.right != null)
        x.right.parent = y;

    // 将 “y的父亲” 设为 “x的父亲”
    x.parent = y.parent;

    if (y.parent == null) {
        this.mRoot = x;            // 如果 “y的父亲” 是空节点，则将x设为根节点
    } else {
        if (y == y.parent.right)
            y.parent.right = x;    // 如果 y是它父节点的右孩子，则将x设为“y的父节点的右孩子”
        else
            y.parent.left = x;    // (y是它父节点的左孩子) 将x设为“x的父节点的左孩子”
    }

    // 将 “y” 设为 “x的右孩子”
    x.right = y;

    // 将 “y的父节点” 设为 “x”
    y.parent = x;
}
```

 

#### **4. 添加**

将一个节点插入到红黑树中，需要执行哪些步骤呢？首先，将红黑树当作一颗二叉查找树，将节点插入；然后，将节点着色为红色；最后，通过"旋转和重新着色"等一系列操作来修正该树，使之重新成为一颗红黑树。详细描述如下：
**第一步: 将红黑树当作一颗二叉查找树，将节点插入。**
​       红黑树本身就是一颗二叉查找树，将节点插入后，该树仍然是一颗二叉查找树。也就意味着，树的键值仍然是有序的。此外，无论是左旋还是右旋，若旋转之前这棵树是二叉查找树，旋转之后它一定还是二叉查找树。这也就意味着，任何的旋转和重新着色操作，都不会改变它仍然是一颗二叉查找树的事实。
好吧？那接下来，我们就来想方设法的旋转以及重新着色，使这颗树重新成为红黑树！

**第二步：将插入的节点着色为"红色"。**
​       为什么着色成红色，而不是黑色呢？为什么呢？在回答之前，我们需要重新温习一下红黑树的特性：
(1) 每个节点或者是黑色，或者是红色。
(2) 根节点是黑色。
(3) 每个叶子节点是黑色。 [注意：这里叶子节点，是指为空的叶子节点！]
(4) 如果一个节点是红色的，则它的子节点必须是黑色的。
(5) 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。
​      将插入的节点着色为红色，不会违背"特性(5)"！少违背一条特性，就意味着我们需要处理的情况越少。接下来，就要努力的让这棵树满足其它性质即可；满足了的话，它就又是一颗红黑树了。o(∩∩)o...哈哈

**第三步: 通过一系列的旋转或着色等操作，使之重新成为一颗红黑树。**
​       第二步中，将插入节点着色为"红色"之后，不会违背"特性(5)"。那它到底会违背哪些特性呢？
​       对于"特性(1)"，显然不会违背了。因为我们已经将它涂成红色了。
​       对于"特性(2)"，显然也不会违背。在第一步中，我们是将红黑树当作二叉查找树，然后执行的插入操作。而根据二叉查找数的特点，插入操作不会改变根节点。所以，根节点仍然是黑色。
​       对于"特性(3)"，显然不会违背了。这里的叶子节点是指的空叶子节点，插入非空节点并不会对它们造成影响。
​       对于"特性(4)"，是有可能违背的！
​       那接下来，想办法使之"满足特性(4)"，就可以将树重新构造成红黑树了。

添加操作的实现代码

```java
/* 
 * 将结点插入到红黑树中
 *
 * 参数说明：
 *     node 插入的结点        // 对应《算法导论》中的node
 */
private void insert(RBTNode<T> node) {
    int cmp;
    RBTNode<T> y = null;
    RBTNode<T> x = this.mRoot;

    // 1. 将红黑树当作一颗二叉查找树，将节点添加到二叉查找树中。
    while (x != null) {
        y = x;
        cmp = node.key.compareTo(x.key);
        if (cmp < 0)
            x = x.left;
        else
            x = x.right;
    }

    node.parent = y;
    if (y!=null) {
        cmp = node.key.compareTo(y.key);
        if (cmp < 0)
            y.left = node;
        else
            y.right = node;
    } else {
        this.mRoot = node;
    }

    // 2. 设置节点的颜色为红色
    node.color = RED;

    // 3. 将它重新修正为一颗二叉查找树
    insertFixUp(node);
}

/* 
 * 新建结点(key)，并将其插入到红黑树中
 *
 * 参数说明：
 *     key 插入结点的键值
 */
public void insert(T key) {
    RBTNode<T> node=new RBTNode<T>(key,BLACK,null,null,null);

    // 如果新建结点失败，则返回。
    if (node != null)
        insert(node);
}
```



**内部接口** -- insert(node)的作用是将"node"节点插入到红黑树中。
**外部接口** -- insert(key)的作用是将"key"添加到红黑树中。

添加修正操作的实现代码



```java
/*
 * 红黑树插入修正函数
 *
 * 在向红黑树中插入节点之后(失去平衡)，再调用该函数；
 * 目的是将它重新塑造成一颗红黑树。
 *
 * 参数说明：
 *     node 插入的结点        // 对应《算法导论》中的z
 */
private void insertFixUp(RBTNode<T> node) {
    RBTNode<T> parent, gparent;

    // 若“父节点存在，并且父节点的颜色是红色”
    while (((parent = parentOf(node))!=null) && isRed(parent)) {
        // 获得祖父节点
        gparent = parentOf(parent);

        //若“父节点”是“祖父节点的左孩子”
        if (parent == gparent.left) {
            // Case 1条件：叔叔节点是红色
            RBTNode<T> uncle = gparent.right;
            if ((uncle!=null) && isRed(uncle)) {
                setBlack(uncle);
                setBlack(parent);
                setRed(gparent);
                node = gparent;
                continue;
            }

            // Case 2条件：叔叔是黑色，且当前节点是右孩子
            if (parent.right == node) {
                RBTNode<T> tmp;
                leftRotate(parent);
                tmp = parent;
                parent = node;
                node = tmp;
            }

            // Case 3条件：叔叔是黑色，且当前节点是左孩子。
            setBlack(parent);
            setRed(gparent);
            rightRotate(gparent);
        } else {    //若“z的父节点”是“z的祖父节点的右孩子”
            // Case 1条件：叔叔节点是红色
            RBTNode<T> uncle = gparent.left;
            if ((uncle!=null) && isRed(uncle)) {
                setBlack(uncle);
                setBlack(parent);
                setRed(gparent);
                node = gparent;
                continue;
            }

            // Case 2条件：叔叔是黑色，且当前节点是左孩子
            if (parent.left == node) {
                RBTNode<T> tmp;
                rightRotate(parent);
                tmp = parent;
                parent = node;
                node = tmp;
            }

            // Case 3条件：叔叔是黑色，且当前节点是右孩子。
            setBlack(parent);
            setRed(gparent);
            leftRotate(gparent);
        }
    }

    // 将根节点设为黑色
    setBlack(this.mRoot);
}

```

insertFixUp(node)的作用是对应"上面所讲的第三步"。它是一个内部接口。

 

#### **5. 删除操作**

将红黑树内的某一个节点删除。需要执行的操作依次是：首先，将红黑树当作一颗二叉查找树，将该节点从二叉查找树中删除；然后，通过"旋转和重新着色"等一系列来修正该树，使之重新成为一棵红黑树。详细描述如下：
**第一步：将红黑树当作一颗二叉查找树，将节点删除。**
​       这和"删除常规二叉查找树中删除节点的方法是一样的"。分3种情况：
① 被删除节点没有儿子，即为叶节点。那么，直接将该节点删除就OK了。
② 被删除节点只有一个儿子。那么，直接删除该节点，并用该节点的唯一子节点顶替它的位置。
③ 被删除节点有两个儿子。那么，先找出它的后继节点；然后把“它的后继节点的内容”复制给“该节点的内容”；之后，删除“它的后继节点”。在这里，后继节点相当于替身，在将后继节点的内容复制给"被删除节点"之后，再将后继节点删除。这样就巧妙的将问题转换为"删除后继节点"的情况了，下面就考虑后继节点。 在"被删除节点"有两个非空子节点的情况下，它的后继节点不可能是双子非空。既然"的后继节点"不可能双子都非空，就意味着"该节点的后继节点"要么没有儿子，要么只有一个儿子。若没有儿子，则按"情况① "进行处理；若只有一个儿子，则按"情况② "进行处理。

**第二步：通过"旋转和重新着色"等一系列来修正该树，使之重新成为一棵红黑树。**
​        因为"第一步"中删除节点之后，可能会违背红黑树的特性。所以需要通过"旋转和重新着色"来修正该树，使之重新成为一棵红黑树。

 

删除操作的实现代码

```java
/* 
 * 删除结点(node)，并返回被删除的结点
 *
 * 参数说明：
 *     node 删除的结点
 */
private void remove(RBTNode<T> node) {
    RBTNode<T> child, parent;
    boolean color;

    // 被删除节点的"左右孩子都不为空"的情况。
    if ( (node.left!=null) && (node.right!=null) ) {
        // 被删节点的后继节点。(称为"取代节点")
        // 用它来取代"被删节点"的位置，然后再将"被删节点"去掉。
        RBTNode<T> replace = node;

        // 获取后继节点
        replace = replace.right;
        while (replace.left != null)
            replace = replace.left;

        // "node节点"不是根节点(只有根节点不存在父节点)
        if (parentOf(node)!=null) {
            if (parentOf(node).left == node)
                parentOf(node).left = replace;
            else
                parentOf(node).right = replace;
        } else {
            // "node节点"是根节点，更新根节点。
            this.mRoot = replace;
        }

        // child是"取代节点"的右孩子，也是需要"调整的节点"。
        // "取代节点"肯定不存在左孩子！因为它是一个后继节点。
        child = replace.right;
        parent = parentOf(replace);
        // 保存"取代节点"的颜色
        color = colorOf(replace);

        // "被删除节点"是"它的后继节点的父节点"
        if (parent == node) {
            parent = replace;
        } else {
            // child不为空
            if (child!=null)
                setParent(child, parent);
            parent.left = child;

            replace.right = node.right;
            setParent(node.right, replace);
        }

        replace.parent = node.parent;
        replace.color = node.color;
        replace.left = node.left;
        node.left.parent = replace;

        if (color == BLACK)
            removeFixUp(child, parent);

        node = null;
        return ;
    }

    if (node.left !=null) {
        child = node.left;
    } else {
        child = node.right;
    }

    parent = node.parent;
    // 保存"取代节点"的颜色
    color = node.color;

    if (child!=null)
        child.parent = parent;

    // "node节点"不是根节点
    if (parent!=null) {
        if (parent.left == node)
            parent.left = child;
        else
            parent.right = child;
    } else {
        this.mRoot = child;
    }

    if (color == BLACK)
        removeFixUp(child, parent);
    node = null;
}

/* 
 * 删除结点(z)，并返回被删除的结点
 *
 * 参数说明：
 *     tree 红黑树的根结点
 *     z 删除的结点
 */
public void remove(T key) {
    RBTNode<T> node; 

    if ((node = search(mRoot, key)) != null)
        remove(node);
}
```



**内部接口** -- remove(node)的作用是将"node"节点插入到红黑树中。
**外部接口** -- remove(key)删除红黑树中键值为key的节点。

删除修正操作的实现代码(Java语言)

```java
/*
 * 红黑树删除修正函数
 *
 * 在从红黑树中删除插入节点之后(红黑树失去平衡)，再调用该函数；
 * 目的是将它重新塑造成一颗红黑树。
 *
 * 参数说明：
 *     node 待修正的节点
 */
private void removeFixUp(RBTNode<T> node, RBTNode<T> parent) {
    RBTNode<T> other;

    while ((node==null || isBlack(node)) && (node != this.mRoot)) {
        if (parent.left == node) {
            other = parent.right;
            if (isRed(other)) {
                // Case 1: x的兄弟w是红色的  
                setBlack(other);
                setRed(parent);
                leftRotate(parent);
                other = parent.right;
            }

            if ((other.left==null || isBlack(other.left)) &&
                (other.right==null || isBlack(other.right))) {
                // Case 2: x的兄弟w是黑色，且w的俩个孩子也都是黑色的  
                setRed(other);
                node = parent;
                parent = parentOf(node);
            } else {

                if (other.right==null || isBlack(other.right)) {
                    // Case 3: x的兄弟w是黑色的，并且w的左孩子是红色，右孩子为黑色。  
                    setBlack(other.left);
                    setRed(other);
                    rightRotate(other);
                    other = parent.right;
                }
                // Case 4: x的兄弟w是黑色的；并且w的右孩子是红色的，左孩子任意颜色。
                setColor(other, colorOf(parent));
                setBlack(parent);
                setBlack(other.right);
                leftRotate(parent);
                node = this.mRoot;
                break;
            }
        } else {

            other = parent.left;
            if (isRed(other)) {
                // Case 1: x的兄弟w是红色的  
                setBlack(other);
                setRed(parent);
                rightRotate(parent);
                other = parent.left;
            }

            if ((other.left==null || isBlack(other.left)) &&
                (other.right==null || isBlack(other.right))) {
                // Case 2: x的兄弟w是黑色，且w的俩个孩子也都是黑色的  
                setRed(other);
                node = parent;
                parent = parentOf(node);
            } else {

                if (other.left==null || isBlack(other.left)) {
                    // Case 3: x的兄弟w是黑色的，并且w的左孩子是红色，右孩子为黑色。  
                    setBlack(other.right);
                    setRed(other);
                    leftRotate(other);
                    other = parent.left;
                }

                // Case 4: x的兄弟w是黑色的；并且w的右孩子是红色的，左孩子任意颜色。
                setColor(other, colorOf(parent));
                setBlack(parent);
                setBlack(other.left);
                rightRotate(parent);
                node = this.mRoot;
                break;
            }
        }
    }

    if (node!=null)
        setBlack(node);
}
```

removeFixup(node, parent)是对应"上面所讲的第三步"。它是一个内部接口。

 

### **红黑树的Java实现(完整源码)**

下面是红黑树实现的完整代码和相应的测试程序。
(1) 除了上面所说的"左旋"、"右旋"、"添加"、"删除"等基本操作之后，还实现了"遍历"、"查找"、"打印"、"最小值"、"最大值"、"创建"、"销毁"等接口。
(2) 函数接口大多分为内部接口和外部接口。内部接口是private函数，外部接口则是public函数。
(3) 测试代码中提供了"插入"和"删除"动作的检测开关。默认是关闭的，打开方法可以参考"代码中的说明"。建议在打开开关后，在草稿上自己动手绘制一下红黑树。

红黑树的实现文件(RBTree.java)

```java
  1 /**
  2  * Java 语言: 红黑树
  3  *
  4  * @author skywang
  5  * @date 2013/11/07
  6  */
  7 
  8 public class RBTree<T extends Comparable<T>> {
  9 
 10     private RBTNode<T> mRoot;    // 根结点
 11 
 12     private static final boolean RED   = false;
 13     private static final boolean BLACK = true;
 14 
 15     public class RBTNode<T extends Comparable<T>> {
 16         boolean color;        // 颜色
 17         T key;                // 关键字(键值)
 18         RBTNode<T> left;    // 左孩子
 19         RBTNode<T> right;    // 右孩子
 20         RBTNode<T> parent;    // 父结点
 21 
 22         public RBTNode(T key, boolean color, RBTNode<T> parent, RBTNode<T> left, RBTNode<T> right) {
 23             this.key = key;
 24             this.color = color;
 25             this.parent = parent;
 26             this.left = left;
 27             this.right = right;
 28         }
 29 
 30         public T getKey() {
 31             return key;
 32         }
 33 
 34         public String toString() {
 35             return ""+key+(this.color==RED?"(R)":"B");
 36         }
 37     }
 38 
 39     public RBTree() {
 40         mRoot=null;
 41     }
 42 
 43     private RBTNode<T> parentOf(RBTNode<T> node) {
 44         return node!=null ? node.parent : null;
 45     }
 46     private boolean colorOf(RBTNode<T> node) {
 47         return node!=null ? node.color : BLACK;
 48     }
 49     private boolean isRed(RBTNode<T> node) {
 50         return ((node!=null)&&(node.color==RED)) ? true : false;
 51     }
 52     private boolean isBlack(RBTNode<T> node) {
 53         return !isRed(node);
 54     }
 55     private void setBlack(RBTNode<T> node) {
 56         if (node!=null)
 57             node.color = BLACK;
 58     }
 59     private void setRed(RBTNode<T> node) {
 60         if (node!=null)
 61             node.color = RED;
 62     }
 63     private void setParent(RBTNode<T> node, RBTNode<T> parent) {
 64         if (node!=null)
 65             node.parent = parent;
 66     }
 67     private void setColor(RBTNode<T> node, boolean color) {
 68         if (node!=null)
 69             node.color = color;
 70     }
 71 
 72     /*
 73      * 前序遍历"红黑树"
 74      */
 75     private void preOrder(RBTNode<T> tree) {
 76         if(tree != null) {
 77             System.out.print(tree.key+" ");
 78             preOrder(tree.left);
 79             preOrder(tree.right);
 80         }
 81     }
 82 
 83     public void preOrder() {
 84         preOrder(mRoot);
 85     }
 86 
 87     /*
 88      * 中序遍历"红黑树"
 89      */
 90     private void inOrder(RBTNode<T> tree) {
 91         if(tree != null) {
 92             inOrder(tree.left);
 93             System.out.print(tree.key+" ");
 94             inOrder(tree.right);
 95         }
 96     }
 97 
 98     public void inOrder() {
 99         inOrder(mRoot);
100     }
101 
102 
103     /*
104      * 后序遍历"红黑树"
105      */
106     private void postOrder(RBTNode<T> tree) {
107         if(tree != null)
108         {
109             postOrder(tree.left);
110             postOrder(tree.right);
111             System.out.print(tree.key+" ");
112         }
113     }
114 
115     public void postOrder() {
116         postOrder(mRoot);
117     }
118 
119 
120     /*
121      * (递归实现)查找"红黑树x"中键值为key的节点
122      */
123     private RBTNode<T> search(RBTNode<T> x, T key) {
124         if (x==null)
125             return x;
126 
127         int cmp = key.compareTo(x.key);
128         if (cmp < 0)
129             return search(x.left, key);
130         else if (cmp > 0)
131             return search(x.right, key);
132         else
133             return x;
134     }
135 
136     public RBTNode<T> search(T key) {
137         return search(mRoot, key);
138     }
139 
140     /*
141      * (非递归实现)查找"红黑树x"中键值为key的节点
142      */
143     private RBTNode<T> iterativeSearch(RBTNode<T> x, T key) {
144         while (x!=null) {
145             int cmp = key.compareTo(x.key);
146 
147             if (cmp < 0) 
148                 x = x.left;
149             else if (cmp > 0) 
150                 x = x.right;
151             else
152                 return x;
153         }
154 
155         return x;
156     }
157 
158     public RBTNode<T> iterativeSearch(T key) {
159         return iterativeSearch(mRoot, key);
160     }
161 
162     /* 
163      * 查找最小结点：返回tree为根结点的红黑树的最小结点。
164      */
165     private RBTNode<T> minimum(RBTNode<T> tree) {
166         if (tree == null)
167             return null;
168 
169         while(tree.left != null)
170             tree = tree.left;
171         return tree;
172     }
173 
174     public T minimum() {
175         RBTNode<T> p = minimum(mRoot);
176         if (p != null)
177             return p.key;
178 
179         return null;
180     }
181      
182     /* 
183      * 查找最大结点：返回tree为根结点的红黑树的最大结点。
184      */
185     private RBTNode<T> maximum(RBTNode<T> tree) {
186         if (tree == null)
187             return null;
188 
189         while(tree.right != null)
190             tree = tree.right;
191         return tree;
192     }
193 
194     public T maximum() {
195         RBTNode<T> p = maximum(mRoot);
196         if (p != null)
197             return p.key;
198 
199         return null;
200     }
201 
202     /* 
203      * 找结点(x)的后继结点。即，查找"红黑树中数据值大于该结点"的"最小结点"。
204      */
205     public RBTNode<T> successor(RBTNode<T> x) {
206         // 如果x存在右孩子，则"x的后继结点"为 "以其右孩子为根的子树的最小结点"。
207         if (x.right != null)
208             return minimum(x.right);
209 
210         // 如果x没有右孩子。则x有以下两种可能：
211         // (01) x是"一个左孩子"，则"x的后继结点"为 "它的父结点"。
212         // (02) x是"一个右孩子"，则查找"x的最低的父结点，并且该父结点要具有左孩子"，找到的这个"最低的父结点"就是"x的后继结点"。
213         RBTNode<T> y = x.parent;
214         while ((y!=null) && (x==y.right)) {
215             x = y;
216             y = y.parent;
217         }
218 
219         return y;
220     }
221      
222     /* 
223      * 找结点(x)的前驱结点。即，查找"红黑树中数据值小于该结点"的"最大结点"。
224      */
225     public RBTNode<T> predecessor(RBTNode<T> x) {
226         // 如果x存在左孩子，则"x的前驱结点"为 "以其左孩子为根的子树的最大结点"。
227         if (x.left != null)
228             return maximum(x.left);
229 
230         // 如果x没有左孩子。则x有以下两种可能：
231         // (01) x是"一个右孩子"，则"x的前驱结点"为 "它的父结点"。
232         // (01) x是"一个左孩子"，则查找"x的最低的父结点，并且该父结点要具有右孩子"，找到的这个"最低的父结点"就是"x的前驱结点"。
233         RBTNode<T> y = x.parent;
234         while ((y!=null) && (x==y.left)) {
235             x = y;
236             y = y.parent;
237         }
238 
239         return y;
240     }
241 
242     /* 
243      * 对红黑树的节点(x)进行左旋转
244      *
245      * 左旋示意图(对节点x进行左旋)：
246      *      px                              px
247      *     /                               /
248      *    x                               y                
249      *   /  \      --(左旋)-.           / \                #
250      *  lx   y                          x  ry     
251      *     /   \                       /  \
252      *    ly   ry                     lx  ly  
253      *
254      *
255      */
256     private void leftRotate(RBTNode<T> x) {
257         // 设置x的右孩子为y
258         RBTNode<T> y = x.right;
259 
260         // 将 “y的左孩子” 设为 “x的右孩子”；
261         // 如果y的左孩子非空，将 “x” 设为 “y的左孩子的父亲”
262         x.right = y.left;
263         if (y.left != null)
264             y.left.parent = x;
265 
266         // 将 “x的父亲” 设为 “y的父亲”
267         y.parent = x.parent;
268 
269         if (x.parent == null) {
270             this.mRoot = y;            // 如果 “x的父亲” 是空节点，则将y设为根节点
271         } else {
272             if (x.parent.left == x)
273                 x.parent.left = y;    // 如果 x是它父节点的左孩子，则将y设为“x的父节点的左孩子”
274             else
275                 x.parent.right = y;    // 如果 x是它父节点的左孩子，则将y设为“x的父节点的左孩子”
276         }
277         
278         // 将 “x” 设为 “y的左孩子”
279         y.left = x;
280         // 将 “x的父节点” 设为 “y”
281         x.parent = y;
282     }
283 
284     /* 
285      * 对红黑树的节点(y)进行右旋转
286      *
287      * 右旋示意图(对节点y进行左旋)：
288      *            py                               py
289      *           /                                /
290      *          y                                x                  
291      *         /  \      --(右旋)-.            /  \                     #
292      *        x   ry                           lx   y  
293      *       / \                                   / \                   #
294      *      lx  rx                                rx  ry
295      * 
296      */
297     private void rightRotate(RBTNode<T> y) {
298         // 设置x是当前节点的左孩子。
299         RBTNode<T> x = y.left;
300 
301         // 将 “x的右孩子” 设为 “y的左孩子”；
302         // 如果"x的右孩子"不为空的话，将 “y” 设为 “x的右孩子的父亲”
303         y.left = x.right;
304         if (x.right != null)
305             x.right.parent = y;
306 
307         // 将 “y的父亲” 设为 “x的父亲”
308         x.parent = y.parent;
309 
310         if (y.parent == null) {
311             this.mRoot = x;            // 如果 “y的父亲” 是空节点，则将x设为根节点
312         } else {
313             if (y == y.parent.right)
314                 y.parent.right = x;    // 如果 y是它父节点的右孩子，则将x设为“y的父节点的右孩子”
315             else
316                 y.parent.left = x;    // (y是它父节点的左孩子) 将x设为“x的父节点的左孩子”
317         }
318 
319         // 将 “y” 设为 “x的右孩子”
320         x.right = y;
321 
322         // 将 “y的父节点” 设为 “x”
323         y.parent = x;
324     }
325 
326     /*
327      * 红黑树插入修正函数
328      *
329      * 在向红黑树中插入节点之后(失去平衡)，再调用该函数；
330      * 目的是将它重新塑造成一颗红黑树。
331      *
332      * 参数说明：
333      *     node 插入的结点        // 对应《算法导论》中的z
334      */
335     private void insertFixUp(RBTNode<T> node) {
336         RBTNode<T> parent, gparent;
337 
338         // 若“父节点存在，并且父节点的颜色是红色”
339         while (((parent = parentOf(node))!=null) && isRed(parent)) {
340             gparent = parentOf(parent);
341 
342             //若“父节点”是“祖父节点的左孩子”
343             if (parent == gparent.left) {
344                 // Case 1条件：叔叔节点是红色
345                 RBTNode<T> uncle = gparent.right;
346                 if ((uncle!=null) && isRed(uncle)) {
347                     setBlack(uncle);
348                     setBlack(parent);
349                     setRed(gparent);
350                     node = gparent;
351                     continue;
352                 }
353 
354                 // Case 2条件：叔叔是黑色，且当前节点是右孩子
355                 if (parent.right == node) {
356                     RBTNode<T> tmp;
357                     leftRotate(parent);
358                     tmp = parent;
359                     parent = node;
360                     node = tmp;
361                 }
362 
363                 // Case 3条件：叔叔是黑色，且当前节点是左孩子。
364                 setBlack(parent);
365                 setRed(gparent);
366                 rightRotate(gparent);
367             } else {    //若“z的父节点”是“z的祖父节点的右孩子”
368                 // Case 1条件：叔叔节点是红色
369                 RBTNode<T> uncle = gparent.left;
370                 if ((uncle!=null) && isRed(uncle)) {
371                     setBlack(uncle);
372                     setBlack(parent);
373                     setRed(gparent);
374                     node = gparent;
375                     continue;
376                 }
377 
378                 // Case 2条件：叔叔是黑色，且当前节点是左孩子
379                 if (parent.left == node) {
380                     RBTNode<T> tmp;
381                     rightRotate(parent);
382                     tmp = parent;
383                     parent = node;
384                     node = tmp;
385                 }
386 
387                 // Case 3条件：叔叔是黑色，且当前节点是右孩子。
388                 setBlack(parent);
389                 setRed(gparent);
390                 leftRotate(gparent);
391             }
392         }
393 
394         // 将根节点设为黑色
395         setBlack(this.mRoot);
396     }
397 
398     /* 
399      * 将结点插入到红黑树中
400      *
401      * 参数说明：
402      *     node 插入的结点        // 对应《算法导论》中的node
403      */
404     private void insert(RBTNode<T> node) {
405         int cmp;
406         RBTNode<T> y = null;
407         RBTNode<T> x = this.mRoot;
408 
409         // 1. 将红黑树当作一颗二叉查找树，将节点添加到二叉查找树中。
410         while (x != null) {
411             y = x;
412             cmp = node.key.compareTo(x.key);
413             if (cmp < 0)
414                 x = x.left;
415             else
416                 x = x.right;
417         }
418 
419         node.parent = y;
420         if (y!=null) {
421             cmp = node.key.compareTo(y.key);
422             if (cmp < 0)
423                 y.left = node;
424             else
425                 y.right = node;
426         } else {
427             this.mRoot = node;
428         }
429 
430         // 2. 设置节点的颜色为红色
431         node.color = RED;
432 
433         // 3. 将它重新修正为一颗二叉查找树
434         insertFixUp(node);
435     }
436 
437     /* 
438      * 新建结点(key)，并将其插入到红黑树中
439      *
440      * 参数说明：
441      *     key 插入结点的键值
442      */
443     public void insert(T key) {
444         RBTNode<T> node=new RBTNode<T>(key,BLACK,null,null,null);
445 
446         // 如果新建结点失败，则返回。
447         if (node != null)
448             insert(node);
449     }
450 
451 
452     /*
453      * 红黑树删除修正函数
454      *
455      * 在从红黑树中删除插入节点之后(红黑树失去平衡)，再调用该函数；
456      * 目的是将它重新塑造成一颗红黑树。
457      *
458      * 参数说明：
459      *     node 待修正的节点
460      */
461     private void removeFixUp(RBTNode<T> node, RBTNode<T> parent) {
462         RBTNode<T> other;
463 
464         while ((node==null || isBlack(node)) && (node != this.mRoot)) {
465             if (parent.left == node) {
466                 other = parent.right;
467                 if (isRed(other)) {
468                     // Case 1: x的兄弟w是红色的  
469                     setBlack(other);
470                     setRed(parent);
471                     leftRotate(parent);
472                     other = parent.right;
473                 }
474 
475                 if ((other.left==null || isBlack(other.left)) &&
476                     (other.right==null || isBlack(other.right))) {
477                     // Case 2: x的兄弟w是黑色，且w的俩个孩子也都是黑色的  
478                     setRed(other);
479                     node = parent;
480                     parent = parentOf(node);
481                 } else {
482 
483                     if (other.right==null || isBlack(other.right)) {
484                         // Case 3: x的兄弟w是黑色的，并且w的左孩子是红色，右孩子为黑色。  
485                         setBlack(other.left);
486                         setRed(other);
487                         rightRotate(other);
488                         other = parent.right;
489                     }
490                     // Case 4: x的兄弟w是黑色的；并且w的右孩子是红色的，左孩子任意颜色。
491                     setColor(other, colorOf(parent));
492                     setBlack(parent);
493                     setBlack(other.right);
494                     leftRotate(parent);
495                     node = this.mRoot;
496                     break;
497                 }
498             } else {
499 
500                 other = parent.left;
501                 if (isRed(other)) {
502                     // Case 1: x的兄弟w是红色的  
503                     setBlack(other);
504                     setRed(parent);
505                     rightRotate(parent);
506                     other = parent.left;
507                 }
508 
509                 if ((other.left==null || isBlack(other.left)) &&
510                     (other.right==null || isBlack(other.right))) {
511                     // Case 2: x的兄弟w是黑色，且w的俩个孩子也都是黑色的  
512                     setRed(other);
513                     node = parent;
514                     parent = parentOf(node);
515                 } else {
516 
517                     if (other.left==null || isBlack(other.left)) {
518                         // Case 3: x的兄弟w是黑色的，并且w的左孩子是红色，右孩子为黑色。  
519                         setBlack(other.right);
520                         setRed(other);
521                         leftRotate(other);
522                         other = parent.left;
523                     }
524 
525                     // Case 4: x的兄弟w是黑色的；并且w的右孩子是红色的，左孩子任意颜色。
526                     setColor(other, colorOf(parent));
527                     setBlack(parent);
528                     setBlack(other.left);
529                     rightRotate(parent);
530                     node = this.mRoot;
531                     break;
532                 }
533             }
534         }
535 
536         if (node!=null)
537             setBlack(node);
538     }
539 
540     /* 
541      * 删除结点(node)，并返回被删除的结点
542      *
543      * 参数说明：
544      *     node 删除的结点
545      */
546     private void remove(RBTNode<T> node) {
547         RBTNode<T> child, parent;
548         boolean color;
549 
550         // 被删除节点的"左右孩子都不为空"的情况。
551         if ( (node.left!=null) && (node.right!=null) ) {
552             // 被删节点的后继节点。(称为"取代节点")
553             // 用它来取代"被删节点"的位置，然后再将"被删节点"去掉。
554             RBTNode<T> replace = node;
555 
556             // 获取后继节点
557             replace = replace.right;
558             while (replace.left != null)
559                 replace = replace.left;
560 
561             // "node节点"不是根节点(只有根节点不存在父节点)
562             if (parentOf(node)!=null) {
563                 if (parentOf(node).left == node)
564                     parentOf(node).left = replace;
565                 else
566                     parentOf(node).right = replace;
567             } else {
568                 // "node节点"是根节点，更新根节点。
569                 this.mRoot = replace;
570             }
571 
572             // child是"取代节点"的右孩子，也是需要"调整的节点"。
573             // "取代节点"肯定不存在左孩子！因为它是一个后继节点。
574             child = replace.right;
575             parent = parentOf(replace);
576             // 保存"取代节点"的颜色
577             color = colorOf(replace);
578 
579             // "被删除节点"是"它的后继节点的父节点"
580             if (parent == node) {
581                 parent = replace;
582             } else {
583                 // child不为空
584                 if (child!=null)
585                     setParent(child, parent);
586                 parent.left = child;
587 
588                 replace.right = node.right;
589                 setParent(node.right, replace);
590             }
591 
592             replace.parent = node.parent;
593             replace.color = node.color;
594             replace.left = node.left;
595             node.left.parent = replace;
596 
597             if (color == BLACK)
598                 removeFixUp(child, parent);
599 
600             node = null;
601             return ;
602         }
603 
604         if (node.left !=null) {
605             child = node.left;
606         } else {
607             child = node.right;
608         }
609 
610         parent = node.parent;
611         // 保存"取代节点"的颜色
612         color = node.color;
613 
614         if (child!=null)
615             child.parent = parent;
616 
617         // "node节点"不是根节点
618         if (parent!=null) {
619             if (parent.left == node)
620                 parent.left = child;
621             else
622                 parent.right = child;
623         } else {
624             this.mRoot = child;
625         }
626 
627         if (color == BLACK)
628             removeFixUp(child, parent);
629         node = null;
630     }
631 
632     /* 
633      * 删除结点(z)，并返回被删除的结点
634      *
635      * 参数说明：
636      *     tree 红黑树的根结点
637      *     z 删除的结点
638      */
639     public void remove(T key) {
640         RBTNode<T> node; 
641 
642         if ((node = search(mRoot, key)) != null)
643             remove(node);
644     }
645 
646     /*
647      * 销毁红黑树
648      */
649     private void destroy(RBTNode<T> tree) {
650         if (tree==null)
651             return ;
652 
653         if (tree.left != null)
654             destroy(tree.left);
655         if (tree.right != null)
656             destroy(tree.right);
657 
658         tree=null;
659     }
660 
661     public void clear() {
662         destroy(mRoot);
663         mRoot = null;
664     }
665 
666     /*
667      * 打印"红黑树"
668      *
669      * key        -- 节点的键值 
670      * direction  --  0，表示该节点是根节点;
671      *               -1，表示该节点是它的父结点的左孩子;
672      *                1，表示该节点是它的父结点的右孩子。
673      */
674     private void print(RBTNode<T> tree, T key, int direction) {
675 
676         if(tree != null) {
677 
678             if(direction==0)    // tree是根节点
679                 System.out.printf("%2d(B) is root\n", tree.key);
680             else                // tree是分支节点
681                 System.out.printf("%2d(%s) is %2d's %6s child\n", tree.key, isRed(tree)?"R":"B", key, direction==1?"right" : "left");
682 
683             print(tree.left, tree.key, -1);
684             print(tree.right,tree.key,  1);
685         }
686     }
687 
688     public void print() {
689         if (mRoot != null)
690             print(mRoot, mRoot.key, 0);
691     }
692 }
```

红黑树的测试文件(RBTreeTest.java)

```java
 1 /**
 2  * Java 语言: 二叉查找树
 3  *
 4  * @author skywang
 5  * @date 2013/11/07
 6  */
 7 public class RBTreeTest {
 8 
 9     private static final int a[] = {10, 40, 30, 60, 90, 70, 20, 50, 80};
10     private static final boolean mDebugInsert = false;    // "插入"动作的检测开关(false，关闭；true，打开)
11     private static final boolean mDebugDelete = false;    // "删除"动作的检测开关(false，关闭；true，打开)
12 
13     public static void main(String[] args) {
14         int i, ilen = a.length;
15         RBTree<Integer> tree=new RBTree<Integer>();
16 
17         System.out.printf("== 原始数据: ");
18         for(i=0; i<ilen; i++)
19             System.out.printf("%d ", a[i]);
20         System.out.printf("\n");
21 
22         for(i=0; i<ilen; i++) {
23             tree.insert(a[i]);
24             // 设置mDebugInsert=true,测试"添加函数"
25             if (mDebugInsert) {
26                 System.out.printf("== 添加节点: %d\n", a[i]);
27                 System.out.printf("== 树的详细信息: \n");
28                 tree.print();
29                 System.out.printf("\n");
30             }
31         }
32 
33         System.out.printf("== 前序遍历: ");
34         tree.preOrder();
35 
36         System.out.printf("\n== 中序遍历: ");
37         tree.inOrder();
38 
39         System.out.printf("\n== 后序遍历: ");
40         tree.postOrder();
41         System.out.printf("\n");
42 
43         System.out.printf("== 最小值: %s\n", tree.minimum());
44         System.out.printf("== 最大值: %s\n", tree.maximum());
45         System.out.printf("== 树的详细信息: \n");
46         tree.print();
47         System.out.printf("\n");
48 
49         // 设置mDebugDelete=true,测试"删除函数"
50         if (mDebugDelete) {
51             for(i=0; i<ilen; i++)
52             {
53                 tree.remove(a[i]);
54 
55                 System.out.printf("== 删除节点: %d\n", a[i]);
56                 System.out.printf("== 树的详细信息: \n");
57                 tree.print();
58                 System.out.printf("\n");
59             }
60         }
61 
62         // 销毁二叉树
63         tree.clear();
64     }
65 }
```

 

### **红黑树的Java测试程序**

前面已经给出了红黑树的测试代码(RBTreeTest.java)，这里就不再重复说明。下面是测试程序的运行结果：

```
== 原始数据: 10 40 30 60 90 70 20 50 80 
== 前序遍历: 30 10 20 60 40 50 80 70 90 
== 中序遍历: 10 20 30 40 50 60 70 80 90 
== 后序遍历: 20 10 50 40 70 90 80 60 30 
== 最小值: 10
== 最大值: 90
== 树的详细信息: 
30(B) is root
10(B) is 30's   left child
20(R) is 10's  right child
60(R) is 30's  right child
40(B) is 60's   left child
50(R) is 40's  right child
80(B) is 60's  right child
70(R) is 80's   left child
90(R) is 80's  right child
```
