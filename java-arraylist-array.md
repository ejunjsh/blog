---
title: 在Java中怎样把数组转换为ArrayList?
date: 2016-02-09 15:44:21
tags: [java,arraylist,array]
categories: java
---
# 先来一个数组
````java
Element[] array = {new Element(1),new Element(2),new Element(3)};
````

# 方法一
````java
ArrayList<Element> arrayList = new ArrayList<Element>(Arrays.asList(array));
````
首先，我们来看下ArrayList的构造方法的文档。 
ArrayList(Collection < ? extends E > c) : 构造一个包含特定容器的元素的列表，并且根据容器迭代器的顺序返回。 
所以构造方法所做的事情如下： 
1. 将容器c转换为一个数组 
2. 将数组拷贝到ArrayList中称为”elementData”的数组中 

ArrayList的构造方法的源码如下：
````java
public ArrayList(Collection<? extends E> c) {
       elementData = c.toArray();
       size = elementData.length;
 
       if (elementData.getClass() != Object[].class)
             elementData = Arrays.copyOf(elementData, size, Object[].class);
}
````
<!-- more -->

# 方法二
````java
List<Element> list = Arrays.asList(array);
````
这不是最好的，因为asList()返回的列表的大小是固定的。事实上，返回的列表不是java.util.ArrayList，而是定义在java.util.Arrays中一个私有静态类。我们知道ArrayList的实现本质上是一个数组，而asList()返回的列表是由原始数组支持的固定大小的列表。这种情况下，如果添加或删除列表中的元素，程序会抛出异常UnsupportedOperationException。

````java
list.add(new Element(4));
Exception in thread "main" java.lang.ClassCastException: java.util.Arrays$ArrayList cannot be cast to java.util.ArrayList
    at collection.ConvertArray.main(ConvertArray.java:22)
````

# 方法三
````java
List<element> list = new ArrayList<element>(array.length);
Collections.addAll(list, array);
````