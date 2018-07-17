---
title: java数据结构-arraylist
date: 2017-07-16 18:03:15
tags: [java,arraylist,数据结构]
categories: java
---
# 概述
[关于Java集合的小抄](http://calvin1978.blogcn.com/articles/collection.html)中是这样描述的：

>以数组实现。节约空间，但数组有容量限制。超出限制时会增加50%容量，用System.arraycopy()复制到新的数组，因此最好能给出数组大小的预估值。默认第一次插入元素时创建大小为10的数组。

>按数组下标访问元素—get(i)/set(i,e) 的性能很高，这是数组的基本优势。

>直接在数组末尾加入元素—add(e)的性能也高，但如果按下标插入、删除元素—add(i,e), remove(i), remove(e)，则要用System.arraycopy()来移动部分受影响的元素，性能就变差了，这是基本劣势。

ArrayList是一个相对来说比较简单的数据结构，最重要的一点就是它的自动扩容，可以认为就是我们常说的“动态数组”。

来看一段简单的代码：
````java
ArrayList<String> list = new ArrayList<String>();
list.add("语文: 99");
list.add("数学: 98");
list.add("英语: 100");
list.remove(0);
````
在执行这四条语句时，是这么变化的：
[![](http://idiotsky.top/images1/java-arraylist-3.png)](http://idiotsky.top/images1/java-arraylist-3.png)
接下来看看几个常用的方法的源码分析。
<!-- more -->
# ArrayList属性
ArrayList属性主要就是当前数组长度size，以及存放数组的对象elementData数组，除此之外还有一个经常用到的属性就是从AbstractList继承过来的modCount属性，代表ArrayList集合的修改次数。
````java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable {  
    // 序列化id  
    private static final long serialVersionUID = 8683452581122892189L;  
    // 默认初始的容量  
    private static final int DEFAULT_CAPACITY = 10;  
    // 一个空对象  
    private static final Object[] EMPTY_ELEMENTDATA = new Object[0];  
    // 一个空对象，如果使用默认构造函数创建，则默认对象内容默认是该值  
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = new Object[0];  
    // 当前数据对象存放地方，当前对象不参与序列化  
    transient Object[] elementData;  
    // 当前数组长度  
    private int size;  
    // 数组最大长度  
    private static final int MAX_ARRAY_SIZE = 2147483639;  
  
    // 省略方法。。  
} 
````

# 构造函数： 
ArrayList提供了三种方式的构造器，可以构造一个空列表、构造一个指定初始容量的空列表以及构造一个包含指定collection的元素的列表，这些元素按照该collection的迭代器返回它们的顺序排列的。
````java
    // ArrayList带容量大小的构造函数。    
    public ArrayList(int initialCapacity) {
        //创建指定大小的数组
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        //创建空数组
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
   
    //默认构造函数创建，创建一个空数组
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }  
   
    // 创建一个包含collection的ArrayList    
     public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
````
# add方法
当我们在ArrayList中增加元素的时候，会使用add函数。他会将元素放到末尾。具体实现如下：
````java
public boolean add(E e) {
    //插入时先看看能不能扩容
    ensureCapacityInternal(size + 1); 
    //放到最后的元素的后面
    elementData[size++] = e;
    return true;
}
````
我们可以看到他的实现其实最核心的内容就是ensureCapacityInternal。这个函数其实就是自动扩容机制的核心。我们依次来看一下他的具体实现
````java
private void ensureCapacityInternal(int minCapacity) {
    //这里用来判断是不是空数组
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        //如果是空数组，则在插入时默认扩容成DEFAULT_CAPACITY=10
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
 
    ensureExplicitCapacity(minCapacity);
}
 
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
 
    // 超出数组容量了，扩容之
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
 
private void grow(int minCapacity) {
    // 旧容量
    int oldCapacity = elementData.length;
    // 扩展为原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //如果扩容1.5倍，还不够，就用传进来的参数。
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
````
也就是说，当增加数据的时候，如果ArrayList的大小已经不满足需求时，那么就将数组变为原长度的1.5倍，之后的操作就是把老的数组拷到新的数组里面。例如，默认的数组大小是10，也就是说当我们add10个元素之后，再进行一次add时，就会发生自动扩容，数组长度由10变为了15具体情况如下所示：
[![](http://idiotsky.top/images1/java-arraylist-2.png)](http://idiotsky.top/images1/java-arraylist-2.png)

# set和get方法
Array的set和get函数就比较简单了，先做index检查，然后执行赋值或访问操作：
````java
public E set(int index, E element) {
    rangeCheck(index);
 
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
 
public E get(int index) {
    rangeCheck(index);
 
    return elementData(index);
}
````
索引检查
````java
private void rangeCheck(int index) {
        //如果索引大于arraylist元素大小，直接爆异常
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
````

# contains方法
调用indexOf方法，遍历数组中的每一个元素作对比，如果找到对于的元素，则返回true，没有找到则返回false。
````java
public boolean contains(Object o) {  
    return indexOf(o) >= 0;  
}  
````
````java
public int indexOf(Object o) {  
    if (o == null) {  
        for (int i = 0; i < size; i++)  
            if (elementData[i]==null)  
                return i;  
    } else {  
        for (int i = 0; i < size; i++)  
            if (o.equals(elementData[i]))  
                return i;  
    }  
    return -1;  
} 
````

# remove方法
## 根据索引remove
1. 判断索引有没有越界
2. 自增修改次数
3. 将指定位置（index）上的元素保存到oldValue
4. 将指定位置（index）上的元素都往前移动一位
5. 将最后面的一个元素置空，好让垃圾回收器回收
6. 将原来的值oldValue返回

````java
public E remove(int index) {  
    rangeCheck(index);  
  
    modCount++;  
    E oldValue = elementData(index);  
  
    int numMoved = size - index - 1;  
    if (numMoved > 0)  
        System.arraycopy(elementData, index+1, elementData, index,  
                         numMoved);  
    elementData[--size] = null; // 把最后的置null，好让gc回收
  
    return oldValue;  
}  
````
注意：调用这个方法不会缩减数组的长度，只是将最后一个数组元素置空而已。

## 根据对象remove
循环遍历所有对象，得到对象所在索引位置，然后调用fastRemove方法，执行remove操作
````java
public boolean remove(Object o) {  
    if (o == null) {  
        for (int index = 0; index < size; index++)  
            if (elementData[index] == null) {  
                fastRemove(index);  
                return true;  
            }  
    } else {  
        for (int index = 0; index < size; index++)  
            if (o.equals(elementData[index])) {  
                fastRemove(index);  
                return true;  
            }  
    }  
    return false;  
} 
````
定位到需要remove的元素索引，先将index后面的元素往前面移动一位（调用System.arraycooy实现），然后将最后一个元素置空。
````java
private void fastRemove(int index) {  
    modCount++;  
    int numMoved = size - index - 1;  
    if (numMoved > 0)  
        System.arraycopy(elementData, index+1, elementData, index,  
                         numMoved);  
    elementData[--size] = null; // clear to let GC do its work  
}
````

# clear方法
添加操作次数（modCount），将数组内的元素都置空，等待垃圾收集器收集，不减小数组容量。
````java
public void clear() {  
    modCount++;  
  
    // clear to let GC do its work  
    for (int i = 0; i < size; i++)  
        elementData[i] = null;  
  
    size = 0;  
}  
````

# sublist方法
我们看到代码中是创建了一个ArrayList 类里面的一个内部类SubList对象，传入的值中第一个参数时this参数，其实可以理解为返回当前list的部分视图，真实指向的存放数据内容的地方还是同一个地方，如果修改了sublist返回的内容的话，那么原来的list也会变动。
````java
public List<E> subList(int arg0, int arg1) {  
    subListRangeCheck(arg0, arg1, this.size);  
    return new ArrayList.SubList(this, 0, arg0, arg1);  
}  
````

# trimToSize方法
1. 修改次数加1
2. 将elementData中空余的空间（包括null值）去除，例如：数组长度为10，其中只有前三个元素有值，其他为空，那么调用该方法之后，数组的长度变为3.

````java
public void trimToSize() {  
    modCount++;  
    if (size < elementData.length) {  
        elementData = (size == 0)  
          ? EMPTY_ELEMENTDATA  
          : Arrays.copyOf(elementData, size);  
    }  
}  
````

# iterator方法
interator方法返回的是一个内部类，由于内部类的创建默认含有外部的this指针，所以这个内部类可以调用到外部类的属性。
````java
public Iterator<E> iterator() {  
    return new Itr();  
}  
````
一般的话，调用完iterator之后，我们会使用iterator做遍历，这里使用next做遍历的时候有个需要注意的地方，就是调用next的时候，可能会引发ConcurrentModificationException，当修改次数，与期望的修改次数（调用iterator方法时候的修改次数）不一致的时候，会发生该异常，详细我们看一下代码实现：
````java
@SuppressWarnings("unchecked")  
public E next() {  
    checkForComodification();  
    int i = cursor;  
    if (i >= size)  
        throw new NoSuchElementException();  
    Object[] elementData = ArrayList.this.elementData;  
    if (i >= elementData.length)  
        throw new ConcurrentModificationException();  
    cursor = i + 1;  
    return (E) elementData[lastRet = i];  
}  
````
expectedModCount这个值是在用户调用ArrayList的iterator方法时候确定的，但是在这之后用户add，或者remove了ArrayList的元素，那么modCount就会改变，那么这个值就会不相等，将会引发ConcurrentModificationException异常，这个是在多线程使用情况下，比较常见的一个异常。
````java
final void checkForComodification() {  
    if (modCount != expectedModCount)  
        throw new ConcurrentModificationException();  
}  
````

# 数组拷贝
arraylist插入或删除都会涉及到数组的拷贝。
## System.arraycopy 方法
这个方法是个native方法。

参数	|说明
-----|------
src	|原数组
srcPos|	原数组
dest|	目标数组
destPos	|目标数组的起始位置
length|	要复制的数组元素的数目

## Arrays.copyOf方法
original - 要复制的数组 
newLength - 要返回的副本的长度 
newType - 要返回的副本的类型

其实Arrays.copyOf底层也是调用System.arraycopy实现的源码如下：
````java
//基本数据类型（其他类似byte，short···）  
public static int[] copyOf(int[] original, int newLength) {  
        int[] copy = new int[newLength];  
        System.arraycopy(original, 0, copy, 0,  
                         Math.min(original.length, newLength));  
        return copy;  
    }  
````

# 总结
1. ArrayList基于数组实现，可以通过下标索引直接查找到指定位置的元素，因此查找效率高，但每次插入或删除元素，就要大量地移动元素，插入删除元素的效率低。
2. 从上面add的方法中并未对null做判断，所以ArrayList中允许元素为null。
3. 注意其三个不同的构造方法。无参构造方法构造的ArrayList的容量默认为空（jdk8）在后面添加元素时才会扩容为10。带有Collection参数的构造方法，将Collection转化为数组赋给ArrayList的实现数组elementData。
4. 注意扩充容量的方法ensureCapacityInternal。ArrayList在每次增加元素（可能是1个，也可能是一组）时，都要调用该方法来确保足够的容量。当容量不足以容纳当前的元素个数时，就设置新的容量为旧的容量的1.5倍，而后用Arrays.copyof()方法将元素拷贝到新的数组。从中可以看出，当容量不够时，每次增加元素，都要将原来的元素拷贝到一个新的数组中，非常之耗时，也因此建议在事先能确定元素数量的情况下，才使用ArrayList，否则建议使用LinkedList。