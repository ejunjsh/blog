---
title: java数据结构-TreeMap
date: 2017-07-20 13:03:15
tags: [java,红黑树,数据结构]
categories: java
---
> treemap的底层实现是红黑树，所以先说说红黑树吧。

# 红黑树简介
红黑树又称红-黑二叉树，它首先是一颗二叉树，它具体二叉树所有的特性。同时红黑树更是一颗自平衡的排序二叉树。

我们知道一颗基本的二叉查找树他们都需要满足一个基本性质--即树中的任何节点的值大于它的左子节点，且小于它的右子节点。按照这个基本性质使得树的检索效率大大提高。我们知道在生成二叉树的过程是非常容易失衡的，最坏的情况就是一边倒（只有右/左子树），这样势必会导致二叉树的检索效率大大降低（O(n)），所以为了维持二叉树的平衡，大牛们提出了各种实现的算法，如：[AVL](http://baike.baidu.com/view/414610.htm)，[SBT](http://baike.baidu.com/view/2957252.htm)，[伸展树](http://baike.baidu.com/view/1118088.htm)，[TREAP](http://baike.baidu.com/view/956602.htm) ，[红黑树](http://baike.baidu.com/view/133754.htm?fr=aladdin#1_1)等等。

平衡二叉树必须具备如下特性：它是一棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。也就是说该二叉树的任何一个节点，其左右子树的高度都相近。下图左边那个就是平衡二叉树，而右图是一个极端不平衡的二叉树，基本等于一个链表了。
[![](http://idiotsky.me/images1/java-treemap.png)](http://idiotsky.me/images1/java-treemap.png)
<!-- more -->
红黑树顾名思义就是节点是红色或者黑色的平衡二叉树，它通过颜色的约束来维持着二叉树的平衡。对于一棵有效的红黑树二叉树而言我们必须增加如下规则：
1. 每个节点都只能是红色或者黑色
2. 根节点是黑色
3. 每个叶节点（NIL节点，空节点）是黑色的。
4. 如果一个结点是红的，则它两个子节点都是黑的。也就是说在一条路径上不能出现相邻的两个红色结点。
5. 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

这些约束强制了红黑树的关键性质: 从根到叶子的最长的可能路径不多于最短的可能路径的两倍长。结果是这棵树大致上是平衡的。因为操作比如插入、删除和查找某个值的最坏情况时间都要求与树的高度成比例，这个在高度上的理论上限允许红黑树在最坏情况下都是高效的，而不同于普通的二叉查找树。所以红黑树它是复杂而高效的，其检索效率O(log n)。下图为一颗典型的红黑二叉树。
[![](http://idiotsky.me/images1/java-treemap-1.png)](http://idiotsky.me/images1/java-treemap-1.png)
对于红黑二叉树而言它主要包括三大基本操作：左旋、右旋、着色。
[![](http://idiotsky.me/images1/java-treemap-2.gif)](http://idiotsky.me/images1/java-treemap-2.gif)
[![](http://idiotsky.me/images1/java-treemap-3.gif)](http://idiotsky.me/images1/java-treemap-3.gif)

# TreeMap数据结构
TreeMap的定义如下：
````java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
````
TreeMap继承AbstractMap，实现NavigableMap、Cloneable、Serializable三个接口。其中AbstractMap表明TreeMap为一个Map即支持key-value的集合， NavigableMap（[更多](http://docs.oracle.com/javase/7/docs/api/java/util/NavigableMap.html)）则意味着它支持一系列的导航方法，具备针对给定搜索目标返回最接近匹配项的导航方法 。
TreeMap中同时也包含了如下几个重要的属性：
````java
//比较器，因为TreeMap是有序的，通过comparator接口我们可以对TreeMap的内部排序进行精密的控制
private final Comparator<? super K> comparator;
//根节点，entry是treemap的内部类
private transient Entry<K,V> root = null;
//容器大小
private transient int size = 0;
//TreeMap修改次数
private transient int modCount = 0;
//红黑树的节点颜色--红色
private static final boolean RED = false;
//红黑树的节点颜色--黑色
private static final boolean BLACK = true;
````
对于节点Entry，它是TreeMap的内部类，它有几个重要的属性：
````java
//键
K key;
//值
V value;
//左孩子
Entry<K,V> left = null;
//右孩子
Entry<K,V> right = null;
//父亲
Entry<K,V> parent;
//颜色
boolean color = BLACK;
````
# TreeMap put()方法
在了解TreeMap的put()方法之前，我们先了解红黑树增加节点的算法。