---
title: java数据结构-hashmap
date: 2017-07-15 18:03:15
tags: [java,hashmap,数据结构]
categories: java
---
[![one image describes how hashmap works](http://idiotsky.me/images/hashmap.png)](http://idiotsky.me/images/hashmap.png)
<!-- more -->
从这张图可以看到，hashmap是由数组和链表组成的。接下来先说下什么是hash

# 什么是hash算法
很多javaer在使用HashMap时，知道这个数据结构非常好用，存取速度很快，而且任何类型的键值对都能往里面塞，非常方便。但是幕后的实现机制，可能并不理解。

HashMap的底层数据结构是数组，数组中存放着链表。要保证键值对能快速插入，并保证通过键能快速获取，就必须要将键转换成数组索引，也就是说需要有将任意键转换成Integer类型数据的能力。而这个转换算法就是hash算法。

# jdk中hash算法
为了让任何键都能转成整数，jdk为每个对象提供了一个hashCode方法，这个方法提供了将键转换成整数的算法。

拨开云雾见月明，让我们走入源码，看看jdk中几个关键类的hash算法实现。

## Integer
````java
private final int value;

public int hashCode() {
    return value;
}
````
非常简单，Integer的hash算法就是直接获取它的数值。效果不错，毕竟int整数范围很大，分散广冲突小。

## Double
````java
private final double value;

public int hashCode() {
	long bits = doubleToLongBits(value);
	return (int)(bits ^ (bits >>> 32));
}
````
由于double不能当成索引，所以需要转换成整数

* 由于double数据类型底层采用64位bit码表示，采用IEEE浮点标准编码。如果将它使用8字节整数编码方式，就能获取一个long类型的数字
* long类型作为索引范围太大，需要转为int类型。这里简单的获取低32位容易导致散列不均，因为高位部分没有被利用。所以这里采用逻辑右移32位，让高32位和低32位进行XOR操作，导致高位低位都能被利用到
* 最后得到的数字强转int，只保留已经被充分打乱的低32位

最后得到的int类型整数充分打乱，散列均匀。
其中采用XOR操作而不是其他AND、OR等，主要是因为XOR有更好的打乱效果

## Boolean
````java
private final boolean value;

public int hashCode() {
	return value ? 1231 : 1237;
}
````
简单粗暴，直接采用两个质数作为true或false的索引。这两个质数足够大，用来作为索引时，出现碰撞的可能性低

## String
````java
private final char value[];

public int hashCode() {
	int hash = 0;
	for (int i = 0; i < value.length; i++) {
		hash = hash * 31 + value[i];
	}
	return hash;
}
````
设字符串位s，利用公式s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]实现的打乱算法。这个算法保证了字符串每个字符都能充分利用到，而且采用质数31作为基数，逐位加权，打乱整个字符的权重和排列位置，使得散列效果更好

如果不能理解上述公式和实现算法的关系，可以将这个公式分解：
````
f(n)=s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]

f(n)=f(n-1)*31+s[n-1]
````

利用后面这个递推公式，可以很容易写出递归算法：
````java
public int hashRecur(String s) {
	int len = s.length();
	if (len == 0) {
		return 0;
	}
	return 31 * hashRecur(s.substring(0, len - 1)) + s.charAt(len - 1);
}
````
根据这个递归算法不难转成迭代算法：
````java
public int hashLoop(String s) {
	int hash = 0;
	for (int i = 0; i < s.length(); i++) {
		hash = hash * 31 + s.charAt(i);
	}
	return hash;
}
````
这个算法就跟jdk算法基本一致了（jdk算法做了一点点优化）

## 自定义对象
如果没有实现hashCode方法，会采用系统默认的：使用对象的物理地址，转换成int整数，作为索引。这种方式并不灵活，有时候需要根据情况选择一些属性来标识这个对象
````java
public class Person {

	String name;
	
	int age;
	
	String id;
	
	boolean male;

	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + age;
		result = prime * result + ((id == null) ? 0 : id.hashCode());
		result = prime * result + (male ? 1231 : 1237);
		result = prime * result + ((name == null) ? 0 : name.hashCode());
		return result;
	}
}
````
这是eclipse自动补全的方案，也是系统推荐的。原理跟String的hashCode一致，为每个元素求出hashCode，按位加权求和，保证散列均匀

# 数组索引
上面说了那么多，都是如何从一个值转化成一个32位的整数，那下面就讲解通过这个值如何得到数组索引。
按上面的方式求出了每个对象的索引M，看来可以直接插入数组了。不过有个问题，这些整数是32位有符号类型，可能很大，可能是负数，而数组往往会设定一个指定大小N，需要保证M能分散到0 ~ N-1中。

为了能从M中截取其中一部分，最简单的就是使用M mod N，N为2的k次幂。为了保证负数也能适用，采用M & N-1

这样仍然有个问题，它的原理是截取了N以内的范围作为最终结果，导致32位整数M的高位并没有被利用到
````java
public int hash(Object key) {
    int M;
    int hash = (key == null) ? 0 : (M = key.hashCode()) ^ (M >>> 16);
    return hash & (N-1)
}
````
所以，就看到了java8中HashMap的第二道hash算法。思路是：将32位整数M逻辑右移16位，让高16位和低16位进行亦或操作，充分打乱，最后截取N以内整数。这样获取的索引，才是均匀打散的。

# 冲突
上面求到数组索引后，很明显就是给这个数组的某个索引元素赋值。那假如另外又来一个值，求出的索引等于之前某个已经赋值的数组元素，这就发生了冲突了,就像图上面的红色方块，而解决冲突的方法就是通过链表把相同索引的元素用链表关联起来。这种做法在hash冲突中叫做链地址法（拉链法）。

# 扩容
如果当表中的75%已经被占用，即视为需要扩容了
````java
(threshold = capacity * load factor ) < size
````
