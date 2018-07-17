---
title: java数据结构-linkedlist
date: 2017-07-16 19:03:15
tags: [java,linkedlist,数据结构]
categories: java
---
# LinkedList简介
* LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
* LinkedList 实现 List 接口，能对它进行队列操作。
* LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。
* LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
* LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
* LinkedList 是非同步的。
* LinkedList相对于ArrayList来说，是可以快速添加，删除元素，ArrayList添加删除元素的话需移动数组元素，可能还需要考虑到扩容数组长度。

# LinkedList属性
LinkedList本身的 的属性比较少，主要有三个，一个是size，表名当前有多少个节点；一个是first代表第一个节点；一个是last代表最后一个节点。
````java
public class LinkedList<E>  
    extends AbstractSequentialList<E>  
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable{     
    //当前有多少个节点  
    transient int size = 0;  
    //第一个节点  
    transient Node<E> first;  
    //最后一个节点  
    transient Node<E> last;  
    //省略内部类和方法。。  
}  
````
<!-- more -->
# LinkedList构造方法
默认构造方法是空的，什么都没做，表示初始化的时候size为0，first和last的节点都为空：
````java
public LinkedList() {  
}  
````
另一个构造方法是带Collection值得对象作为入参的构造函数的，下面是执行逻辑：
1. 使用this（）调用默认的无参构造函数。
2. 调用addAll（）方法，传入当前的节点个数size，此时size为0，并将collection对象传递进去
3. 检查index有没有数组越界的嫌疑
4. 将collection转换成数组对象a
5. 循环遍历a数组，然后将a数组里面的元素创建成拥有前后连接的节点，然后一个个按照顺序连起来。
6. 修改当前的节点个数size的值
7. 操作次数modCount自增1.

下面是实现的源代码：
默认构造函数
````java
public LinkedList(Collection<? extends E> c) {  
    this();  
    addAll(c);  
} 
````

调用带参数的addAll方法
````java
public boolean addAll(Collection<? extends E> c) {  
    return addAll(size, c);  
} 
````
将collection对象转换成数组链表
````java
public boolean addAll(int index, Collection<? extends E> c) {  
    checkPositionIndex(index);  
  
    Object[] a = c.toArray();  
    int numNew = a.length;  
    if (numNew == 0)  
        return false;  
  
    Node<E> pred, succ;  
    if (index == size) {  
        succ = null;  
        pred = last;  
    } else {  
        succ = node(index);  
        pred = succ.prev;  
    }  
  
    for (Object o : a) {  
        @SuppressWarnings("unchecked") E e = (E) o;  
        Node<E> newNode = new Node<>(pred, e, null);  
        if (pred == null)  
            first = newNode;  
        else  
            pred.next = newNode;  
        pred = newNode;  
    }  
  
    if (succ == null) {  
        last = pred;  
    } else {  
        pred.next = succ;  
        succ.prev = pred;  
    }  
  
    size += numNew;  
    modCount++;  
    return true;  
}  
````
# add方法
## add(E e)方法
该方法直接将新增的元素放置链表的最后面，然后链表的长度（size）加1，修改的次数（modCount）加1
````java
public boolean add(E e) {  
    linkLast(e);  
    return true;  
}  
void linkLast(E e) {  
    final Node<E> l = last;  
    final Node<E> newNode = new Node<>(l, e, null);  
    last = newNode;  
    if (l == null)  
        first = newNode;  
    else  
        l.next = newNode;  
    size++;  
    modCount++;  
}  
````
## add(int index, E element)方法
指定位置往数组链表中添加元素
1. 检查添加的位置index 有没有小于等于当前的长度链表size，并且要求大于0
2. 如果是index是等于size，那么直接往链表的最后面添加元素，相当于调用add(E e)方法
3. 如果index不等于size，则先是索引到处于index位置的元素，然后在index的位置前面添加新增的元素。

````java
public void add(int index, E element) {  
    checkPositionIndex(index);  
  
    if (index == size)  
        linkLast(element);  
    else  
        linkBefore(element, node(index));  
} 
````
把索引到的元素添加到新增的元素之后
````java
void linkBefore(E e, Node<E> succ) {  
    // assert succ != null;  
    final Node<E> pred = succ.prev;  
    final Node<E> newNode = new Node<>(pred, e, succ);  
    succ.prev = newNode;  
    if (pred == null)  
        first = newNode;  
    else  
        pred.next = newNode;  
    size++;  
    modCount++;  
}  
````
# get方法
## get方法
首先是判断索引位置有没有越界，确定完成之后开始遍历链表的元素，那么从头开始遍历还是从结尾开始遍历呢，这里其实是要索引的位置与当前链表长度的一半去做对比，如果索引位置大于当前链表长度的一半，否则从结尾开始遍历
````java
public E get(int index) {  
    checkElementIndex(index);  
    return node(index).item;  
}  

````
遍历链表元素
````java
Node<E> node(int index) {  
    // assert isElementIndex(index);  
  
    if (index < (size >> 1)) {  
        Node<E> x = first;  
        for (int i = 0; i < index; i++)  
            x = x.next;  
        return x;  
    } else {  
        Node<E> x = last;  
        for (int i = size - 1; i > index; i--)  
            x = x.prev;  
        return x;  
    }  
}  
````

## getfirst方法
直接将第一个元素返回
````java
public E getFirst() {  
    final Node<E> f = first;  
    if (f == null)  
        throw new NoSuchElementException();  
    return f.item;  
}  
````
## getlast方法
直接将最后一个元素返回
````java
public E getLast() {  
    final Node<E> l = last;  
    if (l == null)  
        throw new NoSuchElementException();  
    return l.item;  
}  
````

# remove方法
## remove（）方法
remove方法本质调用的还是removeFirst方法
````java
public E remove() {  
    return removeFirst();  
}  
````
## removeFirst（）方法
移除第一个节点，将第一个节点置空，让下一个节点变成第一个节点，链表长度减1，修改次数加1，返回移除的第一个节点。
````java
public E removeFirst() {  
    final Node<E> f = first;  
    if (f == null)  
        throw new NoSuchElementException();  
    return unlinkFirst(f);  
} 
private E unlinkFirst(Node<E> f) {  
    // assert f == first && f != null;  
    final E element = f.item;  
    final Node<E> next = f.next;  
    f.item = null;  
    f.next = null; // help GC  
    first = next;  
    if (next == null)  
        last = null;  
    else  
        next.prev = null;  
    size--;  
    modCount++;  
    return element;  
}   
````
## removeLast（）方法
移除最后一个节点，将最后一个节点置空，最后一个节点的上一个节点变成last节点，，链表长度减1，修改次数加1，返回移除的最后一个节点。
````java
public E removeLast() {  
    final Node<E> l = last;  
    if (l == null)  
        throw new NoSuchElementException();  
    return unlinkLast(l);  
}  
private E unlinkLast(Node<E> l) {  
    // assert l == last && l != null;  
    final E element = l.item;  
    final Node<E> prev = l.prev;  
    l.item = null;  
    l.prev = null; // help GC  
    last = prev;  
    if (prev == null)  
        first = null;  
    else  
        prev.next = null;  
    size--;  
    modCount++;  
    return element;  
}
````
## remove（int index）方法
先是检查移除的位置是否在链表长度的范围内，如果不在则抛出异常，根据索引index获取需要移除的节点，将移除的节点置空，让其上一个节点和下一个节点对接起来。
````java
public E remove(int index) {  
    checkElementIndex(index);  
    return unlink(node(index));  
}  
````

# set方法
检查设置元素位然后置是否越界，如果没有，则索引到index位置的节点，将index位置的节点内容替换成新的内容element，同时返回旧值。
````java
public E set(int index, E element) {  
    checkElementIndex(index);  
    Node<E> x = node(index);  
    E oldVal = x.item;  
    x.item = element;  
    return oldVal;  
}  
````
# clear方法
将所有链表元素置空，然后将链表长度修改成0，修改次数加1
````java
public void clear() {  
    // Clearing all of the links between nodes is "unnecessary", but:  
    // - helps a generational GC if the discarded nodes inhabit  
    //   more than one generation  
    // - is sure to free memory even if there is a reachable Iterator  
    for (Node<E> x = first; x != null; ) {  
        Node<E> next = x.next;  
        x.item = null;  
        x.next = null;  
        x.prev = null;  
        x = next;  
    }  
    first = last = null;  
    size = 0;  
    modCount++;  
}
````
# push和pop方法
push其实就是调用addFirst(e)方法，pop调用的就是removeFirst()方法。

# toArray方法
创建一个Object的数组对象，然后将所有的节点都添加到Object对象中，返回Object数组对象。
````java
public Object[] toArray() {  
    Object[] result = new Object[size];  
    int i = 0;  
    for (Node<E> x = first; x != null; x = x.next)  
        result[i++] = x.item;  
    return result;  
}  
````

# listIterator方法
这个方法返回的是一个内部类ListIterator，用户可以使用这个内部类变量遍历当前的链表元素，但是由于LinkedList也是非线程安全的类，所以和上一篇文章中的[java数据结构-arraylist](http://idiotsky.top/2017/07/16/java-arraylist/)  Iterator一样，多线程下面使用，也可能会产生多线程修改的异常。
````java
public ListIterator<E> listIterator(int index) {  
    checkPositionIndex(index);  
    return new ListItr(index);  
}  
````

# 总结
用[关于Java集合的小抄](http://calvin1978.blogcn.com/articles/collection.html)里面的关于linkedlist作为总结吧。
>以双向链表实现。链表无容量限制，但双向链表本身使用了更多空间，每插入一个元素都要构造一个额外的Node对象，也需要额外的链表指针操作。

>按下标访问元素－get（i）、set（i,e） 要悲剧的部分遍历链表将指针移动到位 （如果i>数组大小的一半，会从末尾移起）。

>插入、删除元素时修改前后节点的指针即可，不再需要复制移动。但还是要部分遍历链表的指针才能移动到下标所指的位置。

>只有在链表两头的操作－add（）、addFirst（）、removeLast（）或用iterator（）上的remove（）倒能省掉指针的移动。