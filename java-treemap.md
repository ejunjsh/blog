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
## 节点
下面是代表树的一个节点的代码，很简单，省略了一些不重要的方法。
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
红黑树相比二叉查找树来说，最主要的优势在于自平衡，所以在插入的时候会做很多的平衡处理
````java
public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {
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
        if (cpr != null) {
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
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        //自平衡函数
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
````
自平衡处理在函数`fixAfterInsertion`里面
````java
private void fixAfterInsertion(Entry<K,V> x) {
        x.color = RED;//一开始插入的节点标记为红色
        //如果x的父节点为红色而且x不为根结点
        while (x != null && x != root && x.parent.color == RED) {
            if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
                Entry<K,V> y = rightOf(parentOf(parentOf(x)));
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    if (x == rightOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateLeft(x);
                    }
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateRight(parentOf(parentOf(x)));
                }
            } else {
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
        root.color = BLACK;
    }
````

> to be continue....

参考 
http://www.jianshu.com/p/43b6b90555ca 
http://www.cnblogs.com/yangecnu/p/Introduce-Binary-Search-Tree.html
http://www.cnblogs.com/yangecnu/p/Introduce-2-3-Search-Tree.html
https://algs4.cs.princeton.edu/33balanced/
http://www.cnblogs.com/yangecnu/p/Introduce-Red-Black-Tree.html