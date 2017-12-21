---
title: java数据结构-TreeMap
date: 2017-07-20 13:03:15
tags: [java,二分查找,二叉树,二叉查找树,2-3树,红黑树,数据结构]
categories: java
---
> treemap的底层实现是红黑树，红黑树是出了名难懂的，所以这篇文章先说说相关的算法先，再慢慢回归到红黑树。还有此文章有点长。👿

# 二分查找
二分查找又称折半查找，优点是比较次数少，查找速度快，平均性能好，占用系统内存较少；其缺点是要求待查表为有序表，且插入删除困难。因此，折半查找方法适用于不经常变动而查找频繁的有序列表。首先，假设表中元素是按升序排列，将表中间位置记录的关键字与查找关键字比较，如果两者相等，则查找成功；否则利用中间位置记录将表分成前、后两个子表，如果中间位置记录的关键字大于查找关键字，则进一步查找前一子表，否则进一步查找后一子表。重复以上过程，直到找到满足条件的记录，使查找成功，或直到子表不存在为止，此时查找不成功。
````java
public static int binarySearch(Integer[] srcArray, int des) {
    //定义初始最小、最大索引
    int low = 0;
    int high = srcArray.length - 1;
    //确保不会出现重复查找，越界
    while (low <= high) {
        //计算出中间索引值
        int middle = (high + low)>>>1 ;//防止溢出
        if (des == srcArray[middle]) {
            return middle;
        //判断下限
        } else if (des < srcArray[middle]) {
            high = middle - 1;
        //判断上限
        } else {
            low = middle + 1;
        }
    }
    //若没有，则返回-1
    return -1;
}
````
[![](http://idiotsky.me/images1/java-treemap.jpg)](http://idiotsky.me/images1/java-treemap.jpg)

# 二叉树
在计算机科学中，二叉树是每个节点最多有两个子树的树结构。通常子树被称作“左子树”（left subtree）和“右子树”（right subtree）。二叉树常被用于实现二叉查找树，平衡树，红黑树和二叉堆。二叉树的度就是分支的数目。没有分叉的二叉树节点的度就是0度。如果一个节点只有一个分叉就是1度。两个分叉就是2度的子树。
[![](http://idiotsky.me/images1/java-treemap-1.png)](http://idiotsky.me/images1/java-treemap-1.png)

## 斜树
* 左斜树：所有结点都只有左子树的二叉树；
* 右斜树：所有结点都只有右子树的二叉树；
* 其实在业务逻辑中如果真有这样的需求，那直接使用`线性表`就可以了；
[![](http://idiotsky.me/images1/java-treemap-2.png)](http://idiotsky.me/images1/java-treemap-2.png)

## 满二叉树
* 所有分支结点的度为２；
* 所有叶子结点只能分布在最下层上；
* 同样深度的二叉树中，满二叉树的结点数最多，叶子结点数最多；
[![](http://idiotsky.me/images1/java-treemap-3.png)](http://idiotsky.me/images1/java-treemap-3.png)

## 完全二叉树
* 二叉树从根结点出发，按着层次从左到右遍历，第i个结点的位置与满二叉树中对应结点位置完全相同；
* 所有叶子结点只能出现在最下两层；
* 若结点度为1，则该结点只有左子树，不存在只有右子树的情况；
* 结点分配`先左后右`；
[![](http://idiotsky.me/images1/java-treemap-4.png)](http://idiotsky.me/images1/java-treemap-4.png)

## 二叉树的遍历
* 前序：先访问根结点 -> 遍历左子树 -> 遍历右子树；先访问根结点
* 中序：遍历左子树 -> 访问根结点 -> 遍历右子树；中间访问根结点
* 后序：遍历左子树 -> 遍历右子树 -> 后访问根结点；后访问根结点
* 层序：自上而下，从左到右逐层访问结点；
[![](http://idiotsky.me/images1/java-treemap-5.png)](http://idiotsky.me/images1/java-treemap-5.png)

# 二叉查找树
二叉查找树（Binary Search Tree），也称有序二叉树（ordered binary tree）,排序二叉树（sorted binary tree），是指一棵空树或者具有下列性质的二叉树：
1. 若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
2. 若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
3. 任意节点的左、右子树也分别为二叉查找树。
4. 没有键值相等的节点（no duplicate nodes）。

如下一个二叉树
[![](http://idiotsky.me/images1/java-treemap-6.png)](http://idiotsky.me/images1/java-treemap-6.png)

在此基础上，加上节点之间的大小关系，就是二叉查找树：
[![](http://idiotsky.me/images1/java-treemap-7.png)](http://idiotsky.me/images1/java-treemap-7.png)

## 查找
查找操作和二分查找类似，将key和节点的key比较，如果小于，那么就在Left Node节点查找,如果大于，则在Right Node节点查找，如果相等，直接返回Value。
[![](http://idiotsky.me/images1/java-treemap-8.png)](http://idiotsky.me/images1/java-treemap-8.png)

## 插入
插入和查找类似，首先查找有没有和key相同的，如果有，更新；如果没有找到，那么创建新的节点。
[![](http://idiotsky.me/images1/java-treemap-9.png)](http://idiotsky.me/images1/java-treemap-9.png)

下面是插入动画效果：
[![](http://idiotsky.me/images1/java-treemap-10.gif)](http://idiotsky.me/images1/java-treemap-10.gif)

随机插入形成树的动画如下，可以看到，插入的时候树还是能够保持近似平衡状态：
[![](http://idiotsky.me/images1/java-treemap-11.gif)](http://idiotsky.me/images1/java-treemap-11.gif)

## 最大最小值
如下图可以看出，二叉查找树的最大最小值是有规律的：
[![](http://idiotsky.me/images1/java-treemap-12.png)](http://idiotsky.me/images1/java-treemap-12.png)

从图中可以看出，二叉查找树中，最左和最右节点即为最小值和最大值

## 删除
删除元素操作在二叉树的操作中应该是比较复杂的。首先来看下比较简单的删除最大最小值得方法。

以删除最小值为例，我们首先找到最小值，及最左边左子树为空的节点，然后返回其右子树作为新的左子树。操作示意图如下：
[![](http://idiotsky.me/images1/java-treemap-13.png)](http://idiotsky.me/images1/java-treemap-13.png)

删除最大值也是类似。

现在来分析一般情况，假定我们要删除指定key的某一个节点。这个问题的难点在于：删除最大最小值的操作，删除的节点只有1个子节点或者没有子节点，这样比较简单。但是如果删除任意节点，就有可能出现删除的节点有0个，1 个，2个子节点的情况，现在来逐一分析。

当删除的节点没有子节点时，直接将该父节点指向该节点的link设置为null。
[![](http://idiotsky.me/images1/java-treemap-14.png)](http://idiotsky.me/images1/java-treemap-14.png)

当删除的节点只有1个子节点时，将该自己点替换为要删除的节点即可。
[![](http://idiotsky.me/images1/java-treemap-15.png)](http://idiotsky.me/images1/java-treemap-15.png)

当删除的节点有2个子节点时，问题就变复杂了。

假设我们删除的节点t具有两个子节点。因为t具有右子节点，所以我们需要找到其右子节点中的最小节点，替换t节点的位置。这里有四个步骤：
1. 保存带删除的节点到临时变量t
2. 将t的右节点的最小节点min(t.right)保存到临时节点x
3. 将x的右节点设置为deleteMin(t.right)，该右节点是删除后，所有比x.key最大的节点。
4. 将x的做节点设置为t的左节点。

整个过程如下图：
[![](http://idiotsky.me/images1/java-treemap-16.png)](http://idiotsky.me/images1/java-treemap-16.png)

随着删除的进行，二叉树会变得不太平衡，下面是动画演示。
[![](http://idiotsky.me/images1/java-treemap-17.gif)](http://idiotsky.me/images1/java-treemap-17.gif)

## 分析
二叉查找树的运行时间和树的形状有关，树的形状又和插入元素的顺序有关。在最好的情况下，节点完全平衡，从根节点到最底层叶子节点只有lgN个节点。在最差的情况下，根节点到最底层叶子节点会有N各节点。在一般情况下，树的形状和最好的情况接近。
[![](http://idiotsky.me/images1/java-treemap-18.png)](http://idiotsky.me/images1/java-treemap-18.png)

## 小结
它和二分查找一样，插入和查找的时间复杂度均为lgN，但是在最坏的情况下仍然会有N的时间复杂度。原因在于插入和删除元素的时候，树没有保持平衡。所以我们希望在最坏的情况也有较好的时间复杂度，那就要求树能够尽可能平衡，也就产生了2-3树和红黑树等平衡树。

> to be continue....

参考 
http://www.jianshu.com/p/43b6b90555ca 
http://www.cnblogs.com/yangecnu/p/Introduce-Binary-Search-Tree.html