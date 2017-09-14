---
title: 用cglib生成的代理类取不到注解的问题
date: 2014-07-19 20:35:44
tags: java
categories: java
---
> 经常用cglib来创建代理类来实现aop的功能，可是，当想用反射来取得代理类所代理的类的注解的时候，却怎么也取不到。。。。

<!-- more -->
然后搜了下stackoverflow，http://stackoverflow.com/questions/1706751/retain-annotations-on-cglib-proxies
用`@Inherited`,注解自己的注解（绕~~）
````java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation {
}
````
然后用这个注解注解你要代理的类，那样通过反射可以拿到被代理的注解。
原来CGLIB 返回的代理类是被代理的类的子类，加上这个标志就可以令子类继承这个注解，`@Inherited` 字面意思就是有继承的意思。