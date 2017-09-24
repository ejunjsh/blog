---
title: List、Set、Map集合存放null分析
date: 2014-08-16 00:35:50
tags: java
categories: java
---
> 验证下。

<!-- more -->

# 上代码
````java
package com.sky.code.collection;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class TestNull {
    public static  void main(String[] args){
        System.out.println("测试ArrayList");
        List<Object> list = new ArrayList<Object>();
        list.add(null);
        list.add(null);
        System.out.println(list);

        System.out.println("测试LinkedList");
        List<Object> list1 = new LinkedList<Object>();
        list1.add(null);
        list1.add(null);
        System.out.println(list1);

        System.out.println("测试HashSet");
        Set<Object> set = new HashSet<Object>();
        set.add(null);
        set.add(null);
        System.out.println(set);

        System.out.println("测试TreeSet");
        Set<Object> treeSet = new TreeSet<Object>();
        try {
            treeSet.add(null);
        }catch (Exception e){
            System.out.println(e);
        }

        System.out.println("测试LinkedHashSet");
        Set<Object> set1 = new LinkedHashSet<Object>();
        set1.add(null);
        set1.add(null);
        System.out.println(set1);

        System.out.println("测试HashMap");
        Map<Object, Object> hashMap = new HashMap<>();
        hashMap.put(null, null);
        hashMap.put(null, null);
        hashMap.put(1, null);
        hashMap.put(1, 12);
        System.out.println(hashMap);

        System.out.println("测试LinkedHashMap");
        Map<Object, Object> linkedHashMap = new LinkedHashMap<>();
        linkedHashMap.put(null, null);
        linkedHashMap.put(null, null);
        linkedHashMap.put(1, null);
        linkedHashMap.put(1, 12);
        System.out.println(linkedHashMap);

        System.out.println("测试TreeMap");
        Map<Object, Object> treemap = new TreeMap<>();
        treemap.put(1, null);
        try {
            treemap.put(null, 1);
        }
        catch (Exception e){
            System.out.println(e);
        }
        treemap.put(2, 12);
        System.out.println(treemap);

        System.out.println("测试ConcurrentHashMap");
        Map<Object, Object> concurrentHashMap  = new ConcurrentHashMap<>();
        try {
            concurrentHashMap.put(null, 1);
        }catch (Exception e){
            System.out.println(e);
        }

        try {
            concurrentHashMap.put(1, null);
        }
        catch (Exception e){
            System.out.println(e);
        }
        concurrentHashMap.put(1, 12);
        System.out.println(treemap);

        System.out.println("测试Hashtable");
        Map<Object, Object> hashtable = new Hashtable<>();
        try {
            hashtable.put(null,"null");
        }
        catch (Exception e){
            System.out.println(e);
        }
        try {
            hashtable.put(3,null);
        }
        catch (Exception e){
            System.out.println(e);
        }
    }
}

````

# 结果
````
测试ArrayList
[null, null]
测试LinkedList
[null, null]
测试HashSet
[null]
测试TreeSet
java.lang.NullPointerException
测试LinkedHashSet
[null]
测试HashMap
{null=null, 1=12}
测试LinkedHashMap
{null=null, 1=12}
测试TreeMap
java.lang.NullPointerException
{1=null, 2=12}
测试ConcurrentHashMap
java.lang.NullPointerException
java.lang.NullPointerException
{1=null, 2=12}
测试Hashtable
java.lang.NullPointerException
java.lang.NullPointerException
````

# 结论
很明显tree结构的都不接受key为null,hashtable和concurrenthashmap是key和value都不支持null的，其他都就随便吧。🙂

所有代码在 https://github.com/ejunjsh/java-code/tree/master/src/main/java/com/sky/code/collection