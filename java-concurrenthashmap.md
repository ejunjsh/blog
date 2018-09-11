---
title: java concurrenthashmap解析
date: 2018-07-28 15:31:20
tags: [conccurenthashmap,数据结构,java]
categories: java
---
> 只剩下这个没有mark了，偏偏这时候，面试被问到，悲剧了😢

<!-- more -->

我们对JDK1.8版本的ConcurrentHashMap进行说明，1.8版本的ConcurrentHashMap相比之前的版本主要做了两处改进:

- 使用CAS代替分段锁。
- 红黑树，这一点和HashMap是一致的。


# put

最核心的便是put方法:

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}
```

最后一个参数为onlyIfAbsent，表示只有在key对应的value不存在时才将value加入，所以putVal是put和putIfAbsent两个方法的真正实现。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    //volatile读
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            //初始化
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            //节点添加
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

table便是其数据的存放载体:

```java
transient volatile Node<K,V>[] table;
```

它是volatile的。

## 初始化

如果table为空或大小为0，那么将对其进行初始化操作，initTable:

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    //volatile读
    while ((tab = table) == null || tab.length == 0) {
        //volatile读
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    //sizeCtl设为当前大小的3 / 4
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

sizeCtl是ConcurrentHashMap的初始化，扩容操作中一个至关重要的控制变量，其声明:

```java
private transient volatile int sizeCtl;
```

其取值可能为:

- 0: 初始值。


- -1: 正在进行初始化。
- 负值(小于-1): 表示正在进行扩容，因为ConcurrentHashMap支持多线程并行扩容。
- 正数: 表示下一次触发扩容的临界值大小，即当前值 * 0.75(负载因子)。

从源码中可以看出，ConcurrentHashMap只允许一个线程进行初始化操作，当其它线程竞争失败(sizeCtl < 0)时便会进行自旋，直到竞争成功(初始化)线程完成初始化，那么此时table便不再为null，也就退出了while循环。

Thread.yield方法用于提示CPU可以放弃当前线程的执行，当然这只是一个提示(hint)，这里对此方法的调用是一个优化手段。

对SIZECTL字段CAS更新的成功便标志者线程赢得了竞争，可以进行初始化工作了，剩下的就是一个数组的构造过程，一目了然。

## 头结点设置

如果key对应的bin为空，那么我们只需要将给定的节点 设为头结点即可，这里对应putVal源码中的下面的部分:

```java
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
    if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
        break;
}
```

这里tabAt是一次volatile读，casTabAt为CAS操作。

## 节点添加

如果key对应的bin不为 null，那么就说明需要进行节点添加，从源码可以看出，这里对bin的头结点进行了加锁操作。我的理解为，这里需要**遍历整个链表或搜索红黑树以判断给定的节点(值)是否已存在，同时需要记录链表节点的个数，以决定是否需要将其转化为红黑树**。

## 转为红黑树

指putVal源码中的:

```java
if (binCount != 0) {
    if (binCount >= TREEIFY_THRESHOLD)
        treeifyBin(tab, i);
    if (oldVal != null)
        return oldVal;
    break;
}
```

注意，这段代码是在上述(节点添加部分)同步代码块之外执行的。

TREEIFY_THRESHOLD表示将链表转为红黑树的链表长度的临界值，默认为8.

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            //扩容
            tryPresize(n << 1);
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p = new TreeNode<K,V>(e.hash, e.key, e.val, null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
```

### 扩容

如果当前bin的个数未达到MIN_TREEIFY_CAPACITY，那么不再转为红黑树，转而进行扩容。MIN_TREEIFY_CAPACITY默认为64.tryPresize:

```java
private final void tryPresize(int size) {
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
        tableSizeFor(size + (size >>> 1) + 1);
    int sc;
    //volatile读，没有正在进行初始化或扩容的操作
    while ((sc = sizeCtl) >= 0) {
        Node<K,V>[] tab = table; int n;
        //这里实际上进行了初始化工作
        if (tab == null || (n = tab.length) == 0) {
            n = (sc > c) ? sc : c;
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if (table == tab) {
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        //已达到最大值，无法再进行扩容
        else if (c <= sc || n >= MAXIMUM_CAPACITY)
            break;
        else if (tab == table) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                //竞争失败
                Node<K,V>[] nt;
                //判断是否已经完成
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null || transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            //竞争成功
            else if (U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
        }
    }
}
```

前面提到过了，ConcurrentHashMap支持多线程并行扩容，具体来说，是支持**多线程将节点从老的数组拷贝到新的数组**，而新数组创建仍是一个线程完成(不然多个线程创建多个对象，最后只使用一个，这不是浪费是什么?)

竞争成功的线程为transfer方法的nextTab参数传入null，这将导致新数组的创建。竞争失败的线程将会判断当前节点转移工作是否已经完成，如果已经完成，那么意味着扩容的完成，退出即可，如果没有完成，那么此线程将会进行辅助转移。

判断是否已经完成的条件只能理解(nt = nextTable) == null || transferIndex <= 0两个。

#### 转移

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    //1. 分片
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    //nextTab初始化，CAS保证了只会有一个线程执行这里的代码
    if (nextTab == null) {
        try {
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            //分片的最大下标i实际上就是在这里完成减一的，因为从下面可以看出，每处理完一个桶位便将advance设为true			 //从而便又进入了内层循环，但是注意，当最后一次(即bound)处理完成时，i会被再次减一，从而导致进入下面的			//分支再次读取transferIndex，这就说明了转移线程会在转移完一个分片后继续尝试剩余的分片(桶位)
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                //所有bin均转移完毕
                i = -1;
                advance = false;
            }
            //申请分片
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ? nextIndex - stride : 0))) {
                //bound表示此分片的截止(最小)下标
                bound = nextBound;
                //i表示此分片的最大下标
                i = nextIndex - 1;
                //advance意为前进，跳出内层循环
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            //进入到这里就意味着所有的桶位都已被处理完毕或是被包含在某个转移线程的申请分片中(即待转移)
            int sc;
            if (finishing) {
                //进行收尾工作，此工作一定是由最后一个分片申请线程进行的，这里用volatile写将nextTable置为null
                //，table指向新数组
                nextTable = null;
                table = nextTab;
                //sizeCtl设为新数组大小的3 / 4
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            //转移线程开始转移之前会将sizeCtl自增，转移完成之后自减，所以判断转移是否已经完成的方式便是sizeCtl是			  //否等于初始值
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    //还有其它线程尚未转移完成，直接退出，将收尾工作交给最后完成的那个线程
                    return;
                //进行到这里就说明当前线程为最后一个完成的线程，有意思的是这里又将advance置为true且i置为n(原)
                //数组的大小，作用就是最后再全部扫描一遍所有的桶位，看是否还有漏网之鱼
                finishing = advance = true;
                i = n;
            }
        }
        else if ((f = tabAt(tab, i)) == null)
            //2.
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            synchronized (f) {
                //3. 转移算法
                //双重检查
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                        //runBit代表了当前桶位是否需要移动
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        //这里是找出最后一个和头结点的移动属性相同的
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        //构造无需移动和需要移动的链表
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        //设置到新数组
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        //将原数组的当前桶位设为MOVED，即已处理完(转移)
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                            (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                            (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```

##### 分片

每个线程针对一个分片来进行转移操作，所谓的一个分片其实就是bin数组的一段。默认的最小分片大小为16，如果所在机器 只有一个CPU核心，那么就取16，否则取(数组大小 / 8 / CPU核心数)与16的较大者。

##### transferIndex

全局变量transferIndex表示低于此值的bin尚未被转移，分片的申请便是通过对此变量的CAS操作来完成，初始值为原数组大小，减为0表示 所有桶位均已转移完毕。

##### ForwardingNode

从transfer方法的源码可以看出，当一个桶位(原数组)处理完时，会将其头结点设置一个ForwardingNode。简略版源码:

```java
static final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab) {
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }
}
```

其哈希值为MOVED。到这里我们便可以理解putVal方法这部分源码的作用了:

```java
else if ((fh = f.hash) == MOVED)
    tab = helpTransfer(tab, f);
```

helpTransfer方法的实现和tryPresize方法的相关代码很像，在此不再赘述。

##### 转移算法

我们还是以链表为例，对于2的整次幂扩容来说，节点的转移其实只有两种情况:

- 无需转移，即扩容前后节点的桶位不变。
- 扩容后的桶位号为扩容前 + 原数组的大小，假设原数组大小为8，扩容后为16，有节点哈希值为11，原先在桶位3，那么扩容后位3 + 8 = 11.

所以关键便在于如何判断是否需要转移。还是以大小8和16为例，8的取余mask为:

0111

而16的mask为:

1111

所以我们只要用哈希值 & 8，判断结果是否为零即可。

### 红黑树

再来回顾一下treeifyBin方法的相关源码:

```java
else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
    synchronized (b) {
        //双重检查
        if (tabAt(tab, index) == b) {
            TreeNode<K,V> hd = null, tl = null;
            for (Node<K,V> e = b; e != null; e = e.next) {
                TreeNode<K,V> p = new TreeNode<K,V>(e.hash, e.key, e.val, null, null);
                if ((p.prev = tl) == null)
                    hd = p;
                else
                    tl.next = p;
                tl = p;
            }
            setTabAt(tab, index, new TreeBin<K,V>(hd));
        }
    }
}
```

可见，向红黑树的转换是在锁的保护下进行的，通过一个for循环将所有的节点以TreeNode包装起来，注意，在循环里只是通过next属性进行连接，此时实际上还是一个链表形态，而真正的转化是在TreeBin的构造器中完成的。

和ForwardingNode一样，TreeBin同样具有特殊的哈希值:

```java
static final int TREEBIN   = -2;
```

# get

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 && (e = tabAt(tab, (n - 1) & h)) != null) {
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                //命中头结点
                return e.val;
        }
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            //遍历当前桶位的节点链表
            if (e.hash == h && ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

有意思的在于第二个分支，即哈希值小于零。从上面put方法部分可以得知，共有两种情况节点的哈希值小于0:

- ForwardingNode，已被转移。
- TreeBin，红黑树节点。

##  ForwardingNode

find方法源码:

```java
Node<K,V> find(int h, Object k) {
    outer: for (Node<K,V>[] tab = nextTable;;) {
        Node<K,V> e; int n;
        if (k == null || tab == null || (n = tab.length) == 0 ||
            (e = tabAt(tab, (n - 1) & h)) == null)
            return null;
        for (;;) {
            int eh; K ek;
            if ((eh = e.hash) == h && ((ek = e.key) == k || (ek != null && k.equals(ek))))
                return e;
            if (eh < 0) {
                if (e instanceof ForwardingNode) {
                    //跳转到nextTable搜索
                    tab = ((ForwardingNode<K,V>)e).nextTable;
                    continue outer;
                }
                else
                    //红黑树
                    return e.find(h, k);
            }
            if ((e = e.next) == null)
                return null;
        }
    }
}
```

## 红黑树

TreeBin.find:

```java
final Node<K,V> find(int h, Object k) {
    if (k != null) {
        for (Node<K,V> e = first; e != null; ) {
            int s; K ek;
            if (((s = lockState) & (WAITER|WRITER)) != 0) {
                if (e.hash == h && ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
                e = e.next;
            }
            else if (U.compareAndSwapInt(this, LOCKSTATE, s, s + READER)) {
                TreeNode<K,V> r, p;
                try {
                    p = ((r = root) == null ? null : r.findTreeNode(h, k, null));
                } finally {
                    Thread w;
                    if (U.getAndAddInt(this, LOCKSTATE, -READER) ==
                        (READER|WAITER) && (w = waiter) != null)
                        LockSupport.unpark(w);
                }
                return p;
            }
        }
    }
    return null;
}
```

这里使用了读写锁的方式，而加锁的方式和AQS一个套路。当可以获得读锁时，采用搜索红黑树的方法进行节点搜索，这样时间复杂度是O(LogN)，而如果获得读锁失败(即表示当前有其它线程正在**改变树的结构**，比如进行红黑树的再平衡)，那么将采用线性的搜索策略。

为什么可以进行线性搜索呢?因为红黑树的节点TreeNode继承自Node，所以**仍然保留有next指针(即线性遍历的能力)**。这一点可以从put-转为红黑树-红黑树一节得到反映，线性搜索的线程安全性通过next属性来保证:

```java
volatile Node<K,V> next;
```

TreeBin的构造器同样对树的结构进行了改变，ConcurrentHashMap使用volatile读写来保证线程安全的发布。

从读写锁的引入可以看出，ConcurrentHashMap为保证最大程度的并行执行作出的努力。putTreeVal方法只有在更新树的结构时才会动用锁:

```java
lockRoot();
try {
    root = balanceInsertion(root, x);
} finally {
    unlockRoot();
}
```

除此之外，由于读没有加锁，所以线程可以看到正在进行迁移的桶，但这其实并不会影响正确性，因为迁移是构造了新的链表，并不会影响原有的桶。

# 计数

在putVal方法的结尾通过调用addCount方法(略去大小检查，扩容部分，这里我们只关心计数)进行计数:

```java
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended = U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            fullAddCount(x, uncontended);
            return;
        }
    }
}
```

计数的关键便是counterCells属性:

```java
private transient volatile CounterCell[] counterCells;
```

CounterCell是ConcurrentHashMapd的内部类:

```java
@sun.misc.Contended static final class CounterCell {
    volatile long value;
    CounterCell(long x) { value = x; }
}
```

Contended注解的作用是将类的字段以64字节的填充行包围以解决伪共享问题。其实这里的计数方式就是改编自LongAdder，以最大程度地降低CAS失败空转的几率。

条件判断:

```java
if ((as = counterCells) != null || !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
    //...
}
```

非常有意思 ，如果counterCells为null，那么尝试用baseCount进行计数，如果事实上只有一个线程或多个线程单竞争的频率较低，对baseCount的CAS操作并不会失败，所以可以得到结论 : **如果竞争程度较低(没有CAS失败)，那么其实用的是volatile变量baseCount来计数，只有当线程竞争严重(出现CAS失败)时才会改用LongAdder的方式**。

baseCount声明如下:

```java
private transient volatile long baseCount;
```

再来看一下什么条件下会触发fullAddCount方法:

```java
if (as == null || (m = as.length - 1) < 0 || (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
    !(uncontended = U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
    //...
}
```

ThreadLocalRandom.getProbe()的返回值决定了线程和哪一个CounterCell相关联，查看源码可以发现，此方法返回的其实是Thread的下列字段的值:

```java
@sun.misc.Contended("tlr")
int threadLocalRandomProbe;
```

我们暂且不管这个值是怎么算出来，将其当做一个**线程唯一**的值即可。所以fullAddCount执行的条件是(或):

- CounterCell数组为null。
- CounterCell数组大小为0.
- CounterCell数组线程对应的下标值为null。
- CAS更新线程特定的CounterCell失败。

fullAddCount方法的实现其实和LongAdder的父类Striped64的longAccumulate大体一致:

```java
private final void fullAddCount(long x, boolean wasUncontended) {
    int h;
    if ((h = ThreadLocalRandom.getProbe()) == 0) {
        ThreadLocalRandom.localInit();      // force initialization
        h = ThreadLocalRandom.getProbe();
        wasUncontended = true;
    }
    boolean collide = false;                // True if last slot nonempty
    for (;;) {
        CounterCell[] as; CounterCell a; int n; long v;
        //1.
        if ((as = counterCells) != null && (n = as.length) > 0) {
            if ((a = as[(n - 1) & h]) == null) {
                if (cellsBusy == 0) {            // Try to attach new Cell
                    CounterCell r = new CounterCell(x); // Optimistic create
                    if (cellsBusy == 0 && U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                        boolean created = false;
                        try {               // Recheck under lock
                            CounterCell[] rs; int m, j;
                            if ((rs = counterCells) != null &&
                                (m = rs.length) > 0 &&
                                rs[j = (m - 1) & h] == null) {
                                rs[j] = r;
                                created = true;
                            }
                        } finally {
                            cellsBusy = 0;
                        }
                        if (created)
                            //新Cell创建成功，退出方法
                            break;
                        continue;           // Slot is now non-empty
                    }
                }
                collide = false;
            }
            else if (!wasUncontended)       // CAS already known to fail
                wasUncontended = true;      // Continue after rehash
            else if (U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))
                break;
            else if (counterCells != as || n >= NCPU)
                collide = false;            // At max size or stale
            else if (!collide)
                collide = true;
            else if (cellsBusy == 0 && U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                try {
                    //扩容
                    if (counterCells == as) {// Expand table unless stale
                        CounterCell[] rs = new CounterCell[n << 1];
                        for (int i = 0; i < n; ++i)
                            rs[i] = as[i];
                        counterCells = rs;
                    }
                } finally {
                    cellsBusy = 0;
                }
                collide = false;
                continue;                   // Retry with expanded table
            }
            //rehash
            h = ThreadLocalRandom.advanceProbe(h);
        }
        //2.
        else if (cellsBusy == 0 && counterCells == as && U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
            boolean init = false;
            try {
                //获得锁之后再次检测是否已被初始化
                if (counterCells == as) {
                    CounterCell[] rs = new CounterCell[2];
                    rs[h & 1] = new CounterCell(x);
                    counterCells = rs;
                    init = true;
                }
            } finally {
                //锁释放
                cellsBusy = 0;
            }
            if (init)
                //计数成功，退出方法
                break;
        }
      //3. 
        else if (U.compareAndSwapLong(this, BASECOUNT, v = baseCount, v + x))
            break;                          // Fall back on using base
    }
}
```

从源码中可以看出，在初始情况下probe其实是0的，也就是说在一开始的时候都是更新到第一个cell中的，直到出现CAS失败。

整个方法的逻辑较为复杂，我们按照上面列出的fullAddCount执行条件进行对应说明。

## cell数组为null或empty

容易看出，这里对应的是fullAddCount方法的源码2处。cellBusy的定义如下:

```java
private transient volatile int cellsBusy;
```

这里其实将其当做锁来使用，即只允许在某一时刻只有一个线程正在进行CounterCell数组的初始化或扩容，其值为1说明有线程正在进行上述操作。

默认创建了大小为2的CounterCell数组。

## 下标为null或CAS失败

这里便对应源码的1处，各种条件分支不再展开详细描述，注意一下几点:

### rehash

当Cell数组不为null和empty时，每次循环便会导致重新哈希值，这样做的目的是用再次生成哈希值的方式降低线程竞争。

### 最大CounterCell数

取NCPU:

```java
static final int NCPU = Runtime.getRuntime().availableProcessors();
```

不过从上面扩容部分源码可以看出，最大值并不一定是NCPU，因为采用的是2倍扩容，准确来说是最小的大于等于NCPU的2的整次幂(初始大小为2)。

注意下面这个分支:

```java
else if (counterCells != as || n >= NCPU)
    collide = false;
```

此分支会将collide置为false，从而致使下次循环`else if (!collide)`必定得到满足，这也就保证了扩容分支不会被执行。

## baseCount分支

还会尝试对此变量进行更新，有意思。

# size

```java
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 : (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE : (int)n);
}
```

核心在于sumCount方法:

```java
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```

求和的时候**带上了baseCount**，剩下的就 一目了然了。
