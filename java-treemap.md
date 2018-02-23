---
title: java数据结构-TreeMap
date: 2017-07-20 13:03:15
tags: [java,二分查找,二叉树,二叉查找树,2-3树,红黑树,数据结构]
categories: java
---
> treemap的底层实现是红黑树，红黑树是出了名难懂的，所以这篇文章先说说相关的算法先，再慢慢回归到红黑树。还有此文章有点长。👿

# 二分查找
二分查找又称折半查找，优点是比较次数少，查找速度快，平均性能好，占用系统内存较少，时间复杂度为lgN；其缺点是要求待查表为有序表，且插入删除困难。因此，折半查找方法适用于不经常变动而查找频繁的有序列表。首先，假设表中元素是按升序排列，将表中间位置记录的关键字与查找关键字比较，如果两者相等，则查找成功；否则利用中间位置记录将表分成前、后两个子表，如果中间位置记录的关键字大于查找关键字，则进一步查找前一子表，否则进一步查找后一子表。重复以上过程，直到找到满足条件的记录，使查找成功，或直到子表不存在为止，此时查找不成功。
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

当删除的节点只有1个子节点时，将该子节点替换为要删除的节点即可。
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


# 2-3树
2-3树就是一种以平衡为目的而诞生的数据结构，为了保证在新节点插入后也能保证很好的平衡。

## 定义
和二叉查找树不一样，2-3树运行每个节点保存1个或者两个的key。对于普通的2节点(2-node)，他保存1个key和左右两个自己点。对应3节点(3-node)，保存两个Key，2-3查找树的定义如下：

1. 要么为空，要么就是由很多2-3节点组成：

2. 对于2节点，该节点保存一个key及对应value，以及两个指向左右节点的节点，左节点也是一个2-3节点，所有的值都比key小，右节点也是一个2-3节点，所有的值比key要大。

3. 对于3节点，该节点保存两个key及对应value，以及三个指向左中右的节点。左节点也是一个2-3节点，所有的值均比两个key中的最小的key还要小；中间节点也是一个2-3节点，中间节点的key值在两个跟节点key值之间；右节点也是一个2-3节点，节点的所有key值比两个key中的最大的key还要大。

如果中序遍历2-3查找树，就可以得到排好序的序列。在一个完全平衡的2-3查找树中，根节点到每一个为空节点的距离都相同。
[![](http://idiotsky.me/images2/java-treemap-19.png)](http://idiotsky.me/images2/java-treemap-19.png)

## 查找
在进行2-3树的平衡之前，我们先假设已经处于平衡状态，我们先看基本的查找操作。

2-3树的查找和二叉查找树类似，我们首先和其跟节点进行比较，如果相等，则查找成功；否则根据比较的条件，在其左中右子树中递归查找，如果找到的节点为空，则未找到，否则返回。查找过程如下图：
[![](http://idiotsky.me/images2/java-treemap-20.png)](http://idiotsky.me/images2/java-treemap-20.png)

## 插入
### 往一个2-node节点插入
往2-3树中插入元素和往二叉查找树中插入元素一样，首先要进行查找，然后将节点挂到未找到的节点上。2-3树之所以能够保证在最差的情况下的效率的原因在于其插入之后仍然能够保持平衡状态。如果查找后未找到的节点是一个2-node节点，那么很容易，我们只需要将新的元素放到这个2-node节点里面使其变成一个3-node节点即可。但是如果查找的节点结束于一个3-node节点，那么可能有点麻烦。
[![](http://idiotsky.me/images2/java-treemap-21.png)](http://idiotsky.me/images2/java-treemap-21.png)

### 往一个3-node节点插入
往一个3-node节点插入一个新的节点可能会遇到很多种不同的情况，下面首先从一个最简单的只包含一个3-node节点的树开始讨论。
#### 只包含一个3-node节点
[![](http://idiotsky.me/images2/java-treemap-22.png)](http://idiotsky.me/images2/java-treemap-22.png)

如上图，假设2-3树只包含一个3-node节点，这个节点有两个key，没有空间来插入第三个key了，最自然的方式是我们假设这个节点能存放三个元素，暂时使其变成一个4-node节点，同时他包含四个子节点。然后，我们将这个4-node节点的中间元素提升，左边的节点作为其左节点，右边的元素作为其右节点。插入完成，变为平衡2-3查找树，树的高度从0变为1。

#### 节点是3-node，父节点是2-node
和第一种情况一样，我们也可以将新的元素插入到3-node节点中，使其成为一个临时的4-node节点，然后，将该节点中的中间元素提升到父节点即2-node节点中，使其父节点成为一个3-node节点，然后将左右节点分别挂在这个3-node节点的恰当位置。操作如下图：
[![](http://idiotsky.me/images2/java-treemap-23.png)](http://idiotsky.me/images2/java-treemap-23.png)

#### 节点是3-node，父节点也是3-node
当我们插入的节点是3-node的时候，我们将该节点拆分，中间元素提升至父节点，但是此时父节点是一个3-node节点，插入之后，父节点变成了4-node节点，然后继续将中间元素提升至其父节点，直至遇到一个父节点是2-node节点，然后将其变为3-node，不需要继续进行拆分。
[![](http://idiotsky.me/images2/java-treemap-24.png)](http://idiotsky.me/images2/java-treemap-24.png)

#### 根节点分裂
当根节点到子节点都是3-node节点的时候，这时如果我们要在子节点插入新的元素的时候，会一直拆分到根节点，在最后一步的时候，根节点变成了一个4-node节点，这个时候，就需要将根节点拆分为两个2-node节点，树的高度加1，这个操作过程如下：
[![](http://idiotsky.me/images2/java-treemap-25.png)](http://idiotsky.me/images2/java-treemap-25.png)

#### 本地转换
将一个4-node拆分为2-3node涉及到6种可能的操作。这4-node可能在跟节点，也可能是2-node的左子节点或者右子节点。或者是一个3-node的左，中，右子节点。所有的这些改变都是本地的，不需要检查或者修改其他部分的节点。所以只需要常数次操作即可完成2-3树的平衡。
[![](http://idiotsky.me/images2/java-treemap-26.png)](http://idiotsky.me/images2/java-treemap-26.png)

#### 性质
这些本地操作保持了2-3树的平衡。对于4-node节点变形为2-3节点，变形前后树的高度没有发生变化。只有当跟节点是4-node节点，变形后树的高度才加一。如下图所示：
[![](http://idiotsky.me/images2/java-treemap-27.png)](http://idiotsky.me/images2/java-treemap-27.png)

## 分析
[![](http://idiotsky.me/images2/java-treemap-28.png)](http://idiotsky.me/images2/java-treemap-28.png)

2-3树的查找效率与树的高度是息息相关的。
* 在最坏的情况下，也就是所有的节点都是2-node节点，查找效率为lgN
* 在最好的情况下，所有的节点都是3-node节点，查找效率为log3N约等于0.631lgN

距离来说，对于1百万个节点的2-3树，树的高度为12-20之间，对于10亿个节点的2-3树，树的高度为18-30之间。

对于插入来说，只需要常数次操作即可完成，因为他只需要修改与该节点关联的节点即可，不需要检查其他节点，所以效率和查找类似。

## 实现
直接实现2-3树比较复杂，因为：

1. 需要处理不同的节点类型，非常繁琐
2. 需要多次比较操作来将节点下移
3. 需要上移来拆分4-node节点
4. 拆分4-node节点的情况有很多种

## 小结
2-3查找树实现起来比较复杂，在某些情况插入后的平衡操作可能会使得效率降低。在2-3查找树基础上改进的红黑树不仅具有较高的效率，并且实现起来较2-3查找树简单。

但是2-3查找树作为一种比较重要的概念和思路对于后文要讲到的红黑树非常重要。

# 红黑树
上一节可以看到，2-3查找树能保证在插入元素之后能保持树的平衡状态，最坏情况下即所有的子节点都是2-node，树的高度为lgN，从而保证了最坏情况下的时间复杂度。但是2-3树实现起来比较复杂，本节介绍一种简单实现2-3树的数据结构，即红黑树（Red-Black Tree）

## 定义
红黑树的主要是想对2-3查找树进行编码，尤其是对2-3查找树中的3-nodes节点添加额外的信息。红黑树中将节点之间的链接分为两种不同类型，红色链接，他用来链接两个2-nodes节点来表示一个3-nodes节点。黑色链接用来链接普通的2-3节点。特别的，使用红色链接的两个2-nodes来表示一个3-nodes节点，并且向左倾斜，即一个2-node是另一个2-node的左子节点。这种做法的好处是查找的时候不用做任何修改，和普通的二叉查找树相同。
[![](http://idiotsky.me/images2/java-treemap-29.png)](http://idiotsky.me/images2/java-treemap-29.png)

根据以上描述，红黑树定义如下：
红黑树是一种具有红色和黑色链接的平衡查找树，同时满足：
* 红色节点向左倾斜
* 一个节点不可能有两个红色链接
* 整个树完全黑色平衡，即从根节点到所以叶子结点的路径上，黑色链接的个数都相同。

下图可以看到红黑树其实是2-3树的另外一种表现形式：如果我们将红色的连线水平绘制，那么他链接的两个2-node节点就是2-3树中的一个3-node节点了。
[![](http://idiotsky.me/images2/java-treemap-30.png)](http://idiotsky.me/images2/java-treemap-30.png)

## 表示
我们可以在二叉查找树的每一个节点上增加一个新的表示颜色的标记。该标记指示该节点指向其父节点的颜色。
[![](http://idiotsky.me/images2/java-treemap-31.png)](http://idiotsky.me/images2/java-treemap-31.png)

## 实现
### 查找
红黑树是一种特殊的二叉查找树，他的查找方法也和二叉查找树一样，不需要做太多更改。

但是由于红黑树比一般的二叉查找树具有更好的平衡，所以查找起来更快。

### 平衡化
在介绍插入之前，我们先介绍如何让红黑树保持平衡，因为一般的，我们插入完成之后，需要对树进行平衡化操作以使其满足平衡化。

#### 旋转
旋转又分为__左旋__和__右旋__。通常左旋操作用于将一个向右倾斜的红色链接旋转为向左链接。对比操作前后，可以看出，该操作实际上是将红线链接的两个节点中的一个较大的节点移动到根节点上。

左旋的动画效果如下： 
[![](http://idiotsky.me/images2/java-treemap-32.gif)](http://idiotsky.me/images2/java-treemap-32.gif)

右旋的动画效果如下：
[![](http://idiotsky.me/images2/java-treemap-33.gif)](http://idiotsky.me/images2/java-treemap-33.gif)

#### 颜色反转
当出现一个临时的4-node的时候，即一个节点的两个子节点均为红色，如下图：
[![](http://idiotsky.me/images2/java-treemap-34.png)](http://idiotsky.me/images2/java-treemap-34.png)

[![](http://idiotsky.me/images2/java-treemap-35.png)](http://idiotsky.me/images2/java-treemap-35.png)

这其实是个A，E，S 4-node连接，我们需要将E提升至父节点，操作方法很简单，就是把E对子节点的连线设置为黑色，自己的颜色设置为红色。

有了以上基本操作方法之后，我们现在对应之前对__2-3树的平衡操作来对红黑树进行平衡操作__，这两者是可以一一对应的，如下图：
[![](http://idiotsky.me/images2/java-treemap-36.png)](http://idiotsky.me/images2/java-treemap-36.png)

现在来讨论各种情况：

#### Case 1 往一个2-node节点底部插入新的节点
先热身一下，首先我们看对于只有一个节点的红黑树，插入一个新的节点的操作：
[![](http://idiotsky.me/images2/java-treemap-37.png)](http://idiotsky.me/images2/java-treemap-37.png)

这种情况很简单，只需要：
* 标准的二叉查找树遍历即可。__新插入的节点标记为红色__
* 如果新插入的节点在父节点的右子节点，则需要进行左旋操作

#### Case 2往一个3-node节点底部插入新的节点
先热身一下，假设我们往一个只有两个节点的树中插入元素，如下图，根据待插入元素与已有元素的大小，又可以分为如下三种情况：
[![](http://idiotsky.me/images2/java-treemap-38.png)](http://idiotsky.me/images2/java-treemap-38.png)

* 如果带插入的节点比现有的两个节点都大，这种情况最简单。我们只需要将新插入的节点连接到右边子树上即可，然后将中间的元素提升至根节点。这样根节点的左右子树都是红色的节点了，我们只需要调研FlipColor方法即可。其他情况经过反转操作后都会和这一样。
* 如果插入的节点比最小的元素要小，那么将新节点添加到最左侧，这样就有两个连接红色的节点了，这是对中间节点进行右旋操作，使中间结点成为根节点。这是就转换到了第一种情况，这时候只需要再进行一次FlipColor操作即可。
* 如果插入的节点的值位于两个节点之间，那么将新节点插入到左侧节点的右子节点。因为该节点的右子节点是红色的，所以需要进行左旋操作。操作完之后就变成第二种情况了，再进行一次右旋，然后再调用FlipColor操作即可完成平衡操作。

有了以上基础，我们现在来总结一下往一个3-node节点底部插入新的节点的操作步骤，下面是一个典型的操作过程图：
[![](http://idiotsky.me/images2/java-treemap-39.png)](http://idiotsky.me/images2/java-treemap-39.png)

可以看出，操作步骤如下：
1. 执行标准的二叉查找树插入操作，__新插入的节点元素用红色标识__。
2. 如果需要对4-node节点进行旋转操作
3. 如果需要，调用FlipColor方法将红色节点提升
4. 如果需要，__左旋操作使红色节点左倾__。
5. 在有些情况下，需要递归调用Case1 Case2，来进行递归操作。如下：

[![](http://idiotsky.me/images2/java-treemap-40.png)](http://idiotsky.me/images2/java-treemap-40.png)

### 分析
对红黑树的分析其实就是对2-3查找树的分析，红黑树能够保证在最坏的情况下都能保证对数的时间复杂度，也就是树的高度。

在分析之前，为了更加直观，下面是以升序，降序和随机构建一颗红黑树的动画：

以升序插入构建红黑树：
[![](http://idiotsky.me/images2/java-treemap-41.gif)](http://idiotsky.me/images2/java-treemap-41.gif)

以降序插入构建红黑树：
[![](http://idiotsky.me/images2/java-treemap-42.gif)](http://idiotsky.me/images2/java-treemap-42.gif)

随机插入构建红黑树:
[![](http://idiotsky.me/images2/java-treemap-43.gif)](http://idiotsky.me/images2/java-treemap-43.gif)

从上面三张动画效果中，可以很直观的看出，红黑树在各种情况下都能维护良好的平衡性，从而能够保证最差情况下的查找，插入效率。

下面来分析下红黑树的效率：

#### 在最坏的情况下，红黑树的高度不超过2lgN
最坏的情况就是，红黑树中除了最左侧路径全部是由3-node节点组成，即__红黑相间的路径长度是全黑路径长度的2倍__。

下图是一个典型的红黑树，从中可以看到最长的路径(红黑相间的路径)是最短路径的2倍：
[![](http://idiotsky.me/images2/java-treemap-44.png)](http://idiotsky.me/images2/java-treemap-44.png)

#### 红黑树的平均高度大约为lgN
由于红黑树是2-3查找树的一种实现，所以平均高度大约为lgN

## 小结
前面讲解了自平衡查找树中的2-3查找树，这种数据结构在插入之后能够进行自平衡操作，从而保证了树的高度在一定的范围内进而能够保证最坏情况下的时间复杂度。但是2-3查找树实现起来比较困难，红黑树是2-3树的一种简单高效的实现，他巧妙地使用颜色标记来替代2-3树中比较难处理的3-node节点问题。红黑树是一种比较高效的平衡查找树，应用非常广泛，很多编程语言的内部实现都或多或少的采用了红黑树。

# 红黑树在java中的实现
虽说上面红黑树是2-3树的一种实现，但是在java中红黑树有一个很明显的特点就是，3节点可以出现在右子树上
[![](http://idiotsky.me/images2/java-treemap-45.png)](http://idiotsky.me/images2/java-treemap-45.png)

上面图片的68节点如果是按前一章的实现的话，在插入68的时候，相当于在2节点的右边插入，是需要左旋的。可上图很明显在java的红黑树是允许的，所以不做任何处理。

由于这样的一个规则，打破了之前红黑树可以通过2-3树来理解的思路，所以java实现这里会通过另外些规则来理解。当然前面所学的是不会白费的👿

## 规则
以下是Java中实现的一些规则
1. 每个节点都有颜色（红或黑）；
2. 根节点必须是黑色的；
3. 叶子节点（null节点）是黑的，即每个节点都有两个子节点（其中一个或者两个可能是null节点）；
4. 相连节点不能都是红色（红色节点的父节点和子节点必须为黑色）；（这条在基于2-3树实现的红黑树也有）
5. 任意节点到它所有的叶子节点的路径都含有相同的黑色节点的数量。

其实在java的实现还有个规则跟前面章节是一致的，就是__插入的节点一开始都是红色的__

还有上图22节点和65节点结合规则4和5，会发现一个有趣的东西：如果一个节点只有一个子节点，那么这个子节点肯定是红色的并且没有子节点。

## 节点
下面是代表树的一个节点的代码。
````java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;//关键字
    V value;//值
    Entry<K,V> left;//指向左节点
    Entry<K,V> right;//指向右节点
    Entry<K,V> parent;//指向父节点
    boolean color = BLACK; //红黑节点标记
    //....
}
````

## 查找
红黑树查找跟普通的二叉查找树的查找是一样的。
````java
final Entry<K,V> getEntry(Object key) {
        // Offload comparator-based version for sake of performance
        if (comparator != null)
            return getEntryUsingComparator(key);
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        //从根节点开始
        Entry<K,V> p = root;
        while (p != null) {
            //比较
            int cmp = k.compareTo(p.key);
            //小于的话，继续搜索左子树
            if (cmp < 0)
                p = p.left;
            //大于的话，继续搜索右子树
            else if (cmp > 0)
                p = p.right;
            //找到就返回
            else
                return p;
        }
        return null;
    }
````

## 插入
红黑树相比二叉查找树来说，最主要的优势在于自平衡，所以在插入的时候除了找到合适位置插入外，还会做很多的平衡处理。
在java的实现中主要有上面几个规则约束，所以插入后有可能会破坏规则。
````java
public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {//如果根节点是null，直接插入
            compare(key, key); // type (and possibly null) check

            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {//如果实现了comparator就用comparator比较大小，否则用通用的Comparable比较大小
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        //没找到节点，则进行插入，小于就插入左节点，大于就插入右节点
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        //插入之后可能违反了红黑树的规则，需要进行调整
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
````
函数`fixAfterInsertion`很重要，下面注释先用2-3树的思维来了解的。
````java
private void fixAfterInsertion(Entry<K,V> x) {
        x.color = RED;//一开始插入的节点标记为红色
        //如果x的父节点为红色而且x不为根结点,就继续循环下去。
        //由于父节点是红色，所以操作只处理3节点的插入处理。
        while (x != null && x != root && x.parent.color == RED) {
            //x在3节点的左边或者中间
            if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
                Entry<K,V> y = rightOf(parentOf(parentOf(x)));
                //这里的y是红色，就意味着产生了一个4节点，所以要进行flip color，把红色往上面提升。
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    //这里代表了在3节点中间插入的情况，要进行左旋。
                    if (x == rightOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateLeft(x);
                    }
                    //这里代表了在3节点左边插入的情况，要进行右旋。
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateRight(parentOf(parentOf(x)));
                }
            } else {
                //这里的处理跟上面的相反过来了。而且出现了一种新的东西，
                //一直上面我们说的3节点都在左子树上的，现在由于没有2节点的处理，
                //所以，3节点出现在右子树上。
                Entry<K,V> y = leftOf(parentOf(parentOf(x)));
                //这里的y是红色，就意味着产生了一个4节点，所以要进行flip color，把红色往上面提升。
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    //这里代表了在右子树的3节点中间插入的情况，要进行右旋（跟左子树的3节点相反的操作）。
                    if (x == leftOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateRight(x);
                    }
                    //这里代表了在右子树3节点左边插入的情况，要进行左旋（跟左子树的3节点相反的操作）。
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateLeft(parentOf(parentOf(x)));
                }
            }
        }
        //根节点变成黑色
        root.color = BLACK;
    }
````
从注释上面可以看到，循环只处理3节点的插入操作，而对于2节点操作却产生了一种右子树的3节点的操作，这种操作跟前面提到的3节点进行了相反的处理，左旋时候变右旋，右旋时候左旋。

好了，现在用java的实现来再重新看
`````java
private void fixAfterInsertion(Entry<K,V> x) {
        x.color = RED;  //新插入的节点都是红色的

        while (x != null && x != root && x.parent.color == RED) {   //如果父节点也是红色，违反了规则4，就需要调整。
            if (parentOf(x) == leftOf(parentOf(parentOf(x)))) { //如果x的父节点是x的祖父节点的左孩子，
                Entry<K,V> y = rightOf(parentOf(parentOf(x)));  //y表示x的叔父节点（x的父节点的兄弟节点）
                /**
                * 情况1：如果x的父节点和叔父节点都是红色的
                * 则祖父节点肯定是黑色的
                * 把祖父节点变成红色的，父亲节点和叔父节点变成黑色的
                * （保证从祖父节点到其所有叶子节点的黑色节点数量保持不变）
                * 此时祖父节点从黑色变成红色，可能违反了规则4，while循环继续对祖父节点进行调整
                */
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);   //父亲节点红变黑
                    setColor(y, BLACK);             //叔父节点红变黑
                    setColor(parentOf(parentOf(x)), RED);   //祖父节点黑变红
                    x = parentOf(parentOf(x));  //while循环继续对祖父进行调整
                } else {
                    /**
                    * 情况2：x的叔父节点是黑色的（我们用p代表x的父节点，pp代表x的祖父节点，pr代表x的叔父节点）
                    * x和p是红色，pr和pp是黑色，不能直接变色，这种情况下我们把p变黑，pp变红，
                    * 然后对pp右旋转，左右分支的黑色节点数量不变
                    * 但是右旋转会使p的右孩子变成pp的左孩子，pp现在是红色，如果x是p的右孩子（红色），旋转过去就会冲突。
                    
                    * 所以需要提前判断x如果是p的右孩子，对x的父节点进行左旋转（参照上面的左旋转）
                    * x变成父节点，p变成左孩子，把红色节点移到左分支，x的右孩子是黑色，保证下面的祖父节点右旋转不会发生冲突
                    * 右旋转之后新祖父节点到各个子孙节点的黑色节点数量仍然保持不变，并且是黑色的，不会再和它的父节点冲突，调整到此结束。
                    * 即插入操作最多旋转操作两次就可以解决冲突
                    */
                    if (x == rightOf(parentOf(x))) {
                        x = parentOf(x);    //x指向父亲节点
                        rotateLeft(x);      //对x进行左旋转，把红色节点移到左边
                    }
                    setColor(parentOf(x), BLACK);   //父节点变色
                    setColor(parentOf(parentOf(x)), RED);   //祖父节点变色
                    rotateRight(parentOf(parentOf(x)));     //祖父节点右旋转
                }
            } else {    //下面这种情况对上面的左右对称，操作原理一样
                Entry<K,V> y = leftOf(parentOf(parentOf(x)));
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    if (x == leftOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateRight(x);
                    }
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateLeft(parentOf(parentOf(x)));
                }
            }
        }
        /**
        * 保证根节点是黑色
        * 假设x的父节点和叔父节点都是红色，祖父节点是黑色并且是根节点
        * 符合情况1，那么把父节点叔父节点变黑，祖父节点变红，冲突解决
        * x = parentOf(parentOf(x));此时x是根节点不会再调整，但是此时x是红色的，不满足规则2，所以把根节点置黑。
        */
        root.color = BLACK;
    }
`````

### 示例

再来个实例看看，好理解

以下图红黑树为例
[![](http://idiotsky.me/images2/java-treemap-46.png)](http://idiotsky.me/images2/java-treemap-46.png)

现在我们要增加一个节点50，放在节点47的右子树上。

[![](http://idiotsky.me/images2/java-treemap-47.png)](http://idiotsky.me/images2/java-treemap-47.png)

新增节点和父节点冲突，叔父节点是红色的，进行变色操作，把父亲节点和叔父节点都变成黑色，祖父节点变成红色，然后再对祖父节点进行调整。

````java
if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                }
````

[![](http://idiotsky.me/images2/java-treemap-48.png)](http://idiotsky.me/images2/java-treemap-48.png)

叔父节点y是黑色的，并且x是右孩子，先进行左旋转，把红色节点转移到左分支。

````java
if (x == rightOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateLeft(x);
                    }

````

[![](http://idiotsky.me/images2/java-treemap-49.png)](http://idiotsky.me/images2/java-treemap-49.png)

再把x的父节点变黑，祖父节点变红，然后把祖父节点右旋转。

````java
setColor(parentOf(x), BLACK);
setColor(parentOf(parentOf(x)), RED);
rotateRight(parentOf(parentOf(x)));
````

[![](http://idiotsky.me/images2/java-treemap-50.png)](http://idiotsky.me/images2/java-treemap-50.png)

就这样，又符合上面的规则了。其实用2-3树的思维看上面的过程，是不是跟前一章基于2-3树的红黑树的平衡过程类似呢。


## 删除
前面的章节没关于红黑树的删除，因为本身就是二叉查找树的删除操作，所以红黑树只是比二叉查找树多了一个自平衡的操作。

````java
private void deleteEntry(Entry<K,V> p) {
        modCount++;
        size--;

        //有两个孩子的话，找到它的后继，然后用后继的值更新了要删除的节点
        //这样基本等于要删除的节点被后继的值覆盖了
        if (p.left != null && p.right != null) {
            Entry<K,V> s = successor(p);
            p.key = s.key;
            p.value = s.value;
            //让p指向后继，就是方便后面把后继当成要删除的节点来删掉。
            p = s;
        } 

        //找到替换节点
        Entry<K,V> replacement = (p.left != null ? p.left : p.right);

        //替换节点不为空，代表要删除的节点有孩子
        if (replacement != null) {
            // 替换节点的父亲和要删除的节点要一致
            replacement.parent = p.parent;
            //如果要删除就是根节点
            if (p.parent == null)
                //直接根结点指向替换节点。
                root = replacement;
            //p的父亲指向替换节点。
            else if (p == p.parent.left)
                p.parent.left  = replacement;
            else
                p.parent.right = replacement;

            //删除(unlink)p，方便gc
            p.left = p.right = p.parent = null;

            if (p.color == BLACK)
                //上面不是说过，如果一个节点有一个孩子，那么这个孩子就一定是红色的
                //所以replacement一定是红色的。
                //所以替换节点用红色替换要删除的黑色节点的话，那样会违法规则5，使得
                //黑色数量减少，所以下面的函数，其实做的事情相当于setColor(replacement, BLACK);
                //把replacement变成黑色，没有任何黑色数量减少了。
                //很明显，下面函数一开始的判断条件就不满足了，直接到把replacement变成黑色
                fixAfterDeletion(replacement);
        } else if (p.parent == null) { // 如果树只有一个节点的话，直接root赋值为null
            root = null;
        } else { // 替换节点为null，代表要删除的节点没有孩子
            if (p.color == BLACK)
                //要删除的节点没有孩子，而且是黑色的话，会违反规则5的，
                //所以这里要进行调整这个节点，使他在删除之后不会影响黑色的数量。
                fixAfterDeletion(p);
            //将p的父亲所有指向p的链接指向null
            if (p.parent != null) {
                if (p == p.parent.left)
                    p.parent.left = null;
                else if (p == p.parent.right)
                    p.parent.right = null;
                //删除(unlink)p，方便gc
                p.parent = null;
            }
        }
    }
````
从上面二叉树查找树删除的章节知道，1.如果要删除节点没孩子，直接删掉即可，2.如果一个孩子的，就直接用孩子替换即可，3.对于两个孩子的节点，上面代码有个巧妙的地方，就是找到后继之后，直接用后继的值覆盖删除节点的值，之后把要删除的节点指向成后继节点，而后继节点是只有一个孩子或没有孩子，这样的后继节点刚好可以复用前面说的两种情况的代码了。

函数`fixAfterDeletion`主要用来解决违反规则5的情况，即删除了一个黑色节点，如果它没有孩子，情况更复杂，调整有3种情景：
1. 如果兄弟节点是红色的，经过变色旋转，在x节点上面增加一个红色的父亲节点，并且不破坏其他分支的黑色节点数量。
2. 兄弟节点是黑色的，如果兄弟节点的子节点都是黑色的，直接把黑色节点变红，即减少兄弟分支的黑色节点数量，然后对其父节点进行调整；
3. 兄弟节点是黑色的，但其有红色的孩子节点，不能直接变红。如果左孩子是红色节点，经过变色和右旋转把红色节点移到右边。此时再经过变色，并对x的父节点进行左旋转，在x节点的上面增加一个黑色节点，并且不破坏其他分支的黑色节点数量，调整结束。

````java
private void fixAfterDeletion(Entry<K,V> x) {
        while (x != root && colorOf(x) == BLACK) {  //节点x不是根节点并且是黑色才进行处理
            if (x == leftOf(parentOf(x))) { //x是其父节点的左孩子
                Entry<K,V> sib = rightOf(parentOf(x));  //sib表示x的兄弟节点
                
                /**
                * 如果兄弟节点是红色的，那么父节点肯定是黑色的
                * 把兄弟节点变黑，父节点变红，然后对父节点左旋转
                * 兄弟节点变成父节点，并且到右子树的黑色节点数量不变（由1黑1红变成1黑）
                * 
                * 即情景1，在x节点上增加一个父节点（红色）。
                */
                
                if (colorOf(sib) == RED) {
                    setColor(sib, BLACK);
                    setColor(parentOf(x), RED);
                    rotateLeft(parentOf(x));
                    sib = rightOf(parentOf(x)); //旋转之后重新赋值兄弟节点sib，原sib变成x的祖父节点（见左旋转动图）
                }
                
                /**
                * 进行上一步的判断处理后，此时兄弟节点肯定是黑色的。
                * 如果兄弟节点的孩子节点都是黑色的，我们就可以把兄弟节点变红。
                * 然后while循环继续调整其父节点，即情景2。
                */

                if (colorOf(leftOf(sib))  == BLACK &&
                    colorOf(rightOf(sib)) == BLACK) {
                    setColor(sib, RED);
                    x = parentOf(x);
                } else {    //兄弟节点不能直接变红的情况下，即情景3
                    /**
                    * 如果兄弟节点的左孩子是红色，右孩子是黑色
                    * 兄弟节点的左孩子变黑，兄弟节点变红，对兄弟节点右旋转，把红色节点转移到右分支
                    */
                    if (colorOf(rightOf(sib)) == BLACK) {
                        setColor(leftOf(sib), BLACK);
                        setColor(sib, RED);
                        rotateRight(sib);
                        sib = rightOf(parentOf(x)); //重新赋值兄弟节点
                    }
                    /**
                    * 经过上一步判断处理，兄弟节点是黑色，兄弟节点的左孩子是黑色，兄弟节点的右孩子是红色，
                    * 把兄弟节点变成父节点的颜色，兄弟节点的右孩子变成黑色（不破坏右分支的规则），父节点变成黑色，对父亲节点左旋转，
                    * 主要就在x节点的上面增加了一个黑色的父节点，即情景3，调整结束。
                    */
                    setColor(sib, colorOf(parentOf(x)));
                    setColor(parentOf(x), BLACK);
                    setColor(rightOf(sib), BLACK);
                    rotateLeft(parentOf(x));
                    x = root;
                }
            } else { // x节点是其父节点的右孩子，调整方法和上面的对称。
                Entry<K,V> sib = leftOf(parentOf(x));

                if (colorOf(sib) == RED) {
                    setColor(sib, BLACK);
                    setColor(parentOf(x), RED);
                    rotateRight(parentOf(x));
                    sib = leftOf(parentOf(x));
                }

                if (colorOf(rightOf(sib)) == BLACK &&
                    colorOf(leftOf(sib)) == BLACK) {
                    setColor(sib, RED);
                    x = parentOf(x);
                } else {
                    if (colorOf(leftOf(sib)) == BLACK) {
                        setColor(rightOf(sib), BLACK);
                        setColor(sib, RED);
                        rotateLeft(sib);
                        sib = leftOf(parentOf(x));
                    }
                    setColor(sib, colorOf(parentOf(x)));
                    setColor(parentOf(x), BLACK);
                    setColor(leftOf(sib), BLACK);
                    rotateRight(parentOf(x));
                    x = root;
                }
            }
        }

        setColor(x, BLACK);
    }
````
### 示例
#### 删除的节点只有一个子节点（删除390）
根据上面的引申规则，这个节点肯定是黑色，子节点是红色
[![](http://idiotsky.me/images2/java-treemap-51.png)](http://idiotsky.me/images2/java-treemap-51.png)

#### 删除红色的节点并且没有子节点（删除833）
[![](http://idiotsky.me/images2/java-treemap-52.png)](http://idiotsky.me/images2/java-treemap-52.png)

#### 删除黑色的节点并且没有子节点（删除22）
##### 兄弟节点是红色的情况
[![](http://idiotsky.me/images2/java-treemap-53.png)](http://idiotsky.me/images2/java-treemap-53.png)

变色+旋转，给x节点增加一个红色的父亲节点
````java
if (colorOf(sib) == RED) {
                    setColor(sib, BLACK);
                    setColor(parentOf(x), RED);
                    rotateLeft(parentOf(x));
                    sib = rightOf(parentOf(x));
}
````
[![](http://idiotsky.me/images2/java-treemap-54.png)](http://idiotsky.me/images2/java-treemap-54.png)

此时x的新兄弟节点是黑色，并且孩子节点全是黑色（叶子节点是黑色的），把兄弟节点变色，然后x指向父节点，while循环继续调整。
````java
if (colorOf(leftOf(sib))  == BLACK &&
                    colorOf(rightOf(sib)) == BLACK) {
                    setColor(sib, RED);
                    x = parentOf(x);
````
[![](http://idiotsky.me/images2/java-treemap-55.png)](http://idiotsky.me/images2/java-treemap-55.png)

节点是红色，跳出循环。循环外把该节点变黑。
````java
while (x != root && colorOf(x) == BLACK) {}
setColor(x, BLACK);
````

[![](http://idiotsky.me/images2/java-treemap-56.png)](http://idiotsky.me/images2/java-treemap-56.png)

方法返回后，deleteEntry方法把22节点删除，整个过程结束。

````java
if (p.parent != null) {
                if (p == p.parent.left)
                    p.parent.left = null;
                else if (p == p.parent.right)
                    p.parent.right = null;
                p.parent = null;
            }

````
[![](http://idiotsky.me/images2/java-treemap-57.png)](http://idiotsky.me/images2/java-treemap-57.png)

##### 兄弟节点是黑色的情况
上面是旋转变化过程中其实已经遇见了这种情况，并且兄弟节点的孩子节点全是黑色，可以直接变色处理，下面来看一下，兄弟节点是黑色，并且有孩子节点是红色的情况

继续上面的红黑树，下面删除65节点

[![](http://idiotsky.me/images2/java-treemap-58.png)](http://idiotsky.me/images2/java-treemap-58.png)

兄弟节点是黑色，并且有红色的孩子节点，针对x是左孩子的情况下，如果红色节点是左孩子，需要通过旋转操作移到右边

````java
if (colorOf(rightOf(sib)) == BLACK) {
                        setColor(leftOf(sib), BLACK);
                        setColor(sib, RED);
                        rotateRight(sib);
                        sib = rightOf(parentOf(x));
                    }
````

[![](http://idiotsky.me/images2/java-treemap-59.png)](http://idiotsky.me/images2/java-treemap-59.png)

然后再进行变色旋转操作，给x节点增加一个黑色的父节点。x = root结束循环。
````java
setColor(sib, colorOf(parentOf(x)));
setColor(parentOf(x), BLACK);
setColor(rightOf(sib), BLACK);
rotateLeft(parentOf(x));
x = root;
````
[![](http://idiotsky.me/images2/java-treemap-60.png)](http://idiotsky.me/images2/java-treemap-60.png)

方法返回，deleteEntry方法把65节点删除，整个过程结束。

#### 删除节点有两个孩子节点的情况
[![](http://idiotsky.me/images2/java-treemap-61.png)](http://idiotsky.me/images2/java-treemap-61.png)

删除节点55，该节点有两个孩子节点，deleteEntry方法中会查找继承者节点，即图中的65节点，把65节点的key和value赋值给55节点，然后转化为删除p指向的65节点。
````java
if (p.left != null && p.right != null) {
            Entry<K,V> s = successor(p);
            p.key = s.key;
            p.value = s.value;
            p = s;
        }
````
[![](http://idiotsky.me/images2/java-treemap-62.jpg)](http://idiotsky.me/images2/java-treemap-62.jpg)
因为继承者节点没有左孩子节点，所以这个问题又变成了删除有一个孩子节点或者无孩子节点的问题。（参照上面）

# 总结
很长一篇文章，基本包含了跟树相关的数据结构了，mark下来以后忘了回来看看👿

java除了treemap外，treeset也用了红黑树，而且hashmap和一些依赖链表作为查找的数据结构都会在最后从链表转换成红黑树来提高查找效率。

java在红黑树的实现上有别于基于2-3树实现的红黑树，但是最终产生的红黑树也是类似的。

参考 
http://www.jianshu.com/p/43b6b90555ca 
http://www.cnblogs.com/yangecnu/p/Introduce-Binary-Search-Tree.html
http://www.cnblogs.com/yangecnu/p/Introduce-2-3-Search-Tree.html
https://algs4.cs.princeton.edu/33balanced/
http://www.cnblogs.com/yangecnu/p/Introduce-Red-Black-Tree.html
https://juejin.im/post/5a6bf78a6fb9a01caf37a4bb