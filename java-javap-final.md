---
title: 用javap看一下final是什么
date: 2018-07-17 22:23:29
tags: [javap,java]
categories: java
---

> 一直很好奇`final`的类字段在class文件是怎么表示的，所以用javap看看怎么回事,也顺便复习下字节码指令👿

# 没有`final`修饰的类的静态字段

定义一个类

```java
public class test{
  public static String str="严";

  public static void main(String[] args){
        System.out.println(str);
  }
}

```

<!-- more -->

编译查看class文件

````shell
javac test.java
javap -verbose test.class

//省略常量池
{
  public static java.lang.String str;
    flags: ACC_PUBLIC, ACC_STATIC

  public test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: getstatic     #3                  // Field str:Ljava/lang/String;
         6: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         9: return
      LineNumberTable:
        line 5: 0
        line 6: 9

  static {};
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: ldc           #5                  // String 严
         2: putstatic     #3                  // Field str:Ljava/lang/String;
         5: return
      LineNumberTable:
        line 2: 0
}

````

你会发现`public static String str="严";`直接翻译成一个静态块

````
  static {};
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: ldc           #5                  // String 严
         2: putstatic     #3                  // Field str:Ljava/lang/String;
         5: return
      LineNumberTable:
        line 2: 0
````

上面`ldc`的指令就是把常量从常量池读到操作数栈，`putstatic`指令从栈顶赋值给类的静态字段`str`,这个字段之后在`main`函数会读出来。

````
  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: getstatic     #3                  // Field str:Ljava/lang/String;
         6: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         9: return
      LineNumberTable:
        line 5: 0
        line 6: 9
````
`getstatic`是用来读取类的静态字段的，这里读出来放入操作数栈。

# 有`final`修饰的类静态字段

改一下这个类，加`final`修饰下

```java
public class test{
  public static final String str="严";

  public static void main(String[] args){
        System.out.println(str);
  }
}

```

编译查看class文件

````shell
javac test.java
javap -verbose test.class

# 省略常量池
{
  public static final java.lang.String str;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: String 严

  public test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String 严
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 5: 0
        line 6: 8
}
````
你会发现class文件没有之前的静态块了，而且也不再用`getstatic`指令获取字段的值，而是直接`ldc`指令取常量池的值。

接下来看看实例字段在加`final`或不加会不会有什么不同呢

# 不加final的实例字段

````java
public class test{
  public  String str="严";

  public static void main(String[] args){
        test t=new test();
        System.out.println(t.str);
  }
}

````

类文件

````
//省略没用的了。。。
{
  public java.lang.String str;
    flags: ACC_PUBLIC

  public test();
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String 严
         7: putfield      #3                  // Field str:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 1: 0
        line 2: 4

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #4                  // class test
         3: dup
         4: invokespecial #5                  // Method "<init>":()V
         7: astore_1
         8: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
        11: aload_1
        12: getfield      #3                  // Field str:Ljava/lang/String;
        15: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        18: return
      LineNumberTable:
        line 5: 0
        line 6: 8
        line 7: 18
}

````
构造函数里面用`ldc`初始化了实例字段`str`的值,然后在main函数里面用`getfield`指令，获取`t`实例的`str`的值。

# 加`final`的实例字段

````java
public class test{
  public final String str="严";

  public static void main(String[] args){
        test t=new test();
        System.out.println(t.str);
  }
}

````

类文件

````
//省略没用的了。。。
{
  public final java.lang.String str;
    flags: ACC_PUBLIC, ACC_FINAL
    ConstantValue: String 严

  public test();
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String 严
         7: putfield      #3                  // Field str:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 1: 0
        line 2: 4

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #4                  // class test
         3: dup
         4: invokespecial #5                  // Method "<init>":()V
         7: astore_1
         8: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
        11: aload_1
        12: invokevirtual #7                  // Method java/lang/Object.getClass:()Ljava/lang/Class;
        15: pop
        16: ldc           #2                  // String 严
        18: invokevirtual #8                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        21: return
      LineNumberTable:
        line 5: 0
        line 6: 8
        line 7: 21
}

````
`getfield`指令变成了熟悉的`ldc`,很明显了，加了`final`之后就去常量池去找，就不需要用`getfield`指令。

上面的字段是字符串，那接下来看看数字的字段会怎么样呢

# 数字的非`final`静态字段

上代码

````java
public class test{
  public static int i=100;

  public static void main(String[] args){
        System.out.println(i);
  }
}
````

javap结果

````
//省略常量池
{
  public static int i;
    flags: ACC_PUBLIC, ACC_STATIC

  public test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: getstatic     #3                  // Field i:I
         6: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
         9: return
      LineNumberTable:
        line 6: 0
        line 7: 9

  static {};
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: bipush        100
         2: putstatic     #3                  // Field i:I
         5: return
      LineNumberTable:
        line 3: 0
}

````

基本差不多，只是`ldc`换成了`bipush`。`bipush`就是把后面的操作数（100）压入栈。如果压入的不是100，而是更大或更小，那么用的指令就会不同的了，例如`iconst_1`指令，就是压入1到栈。至于main函数里面，还是用`getstatic`指令获取字段的值。

# 数字的`final`静态字段

上代码

````java
public class test{
  public static final int i=100;

  public static void main(String[] args){
        System.out.println(i);
  }
}
````

javap结果

````

{
  public static final int i;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 100

  public test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: bipush        100
         5: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
         8: return
      LineNumberTable:
        line 6: 0
        line 7: 8
}

````
没啥惊喜的，就是静态块去掉，需要获取字段的值的地方从`getstatic`变成了`bipush`。

至于实例字段是怎么样的，应该都差不多了，就不列举了，接下来看看字段类型是引用的是怎么样呢。

# 加`final`的引用类型字段

上代码

````java
public class test{
  public static final Object o=new Object();

  public static void main(String[] args){
        System.out.println(o);
  }
}
````

javap结果

````

{
  public static final java.lang.Object o;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL

  public test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: getstatic     #3                  // Field o:Ljava/lang/Object;
         6: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
         9: return
      LineNumberTable:
        line 6: 0
        line 7: 9

  static {};
    flags: ACC_STATIC
    Code:
      stack=2, locals=0, args_size=0
         0: new           #5                  // class java/lang/Object
         3: dup
         4: invokespecial #1                  // Method java/lang/Object."<init>":()V
         7: putstatic     #3                  // Field o:Ljava/lang/Object;
        10: return
      LineNumberTable:
        line 3: 0
}

````

这里直接上`final`的版本，是因为加不加`final`，其实都是一样的，都是有静态块，引用的时候不再有什么其他指令了，老老实实的用`getstatic`。


# 总结

对于字符串或者数字类型，他们都是属于字面量，编译时已经知道，他们要么存在常量池里面，要么存在指令的操作数里面，所以加`final`标识的情况下，获取值时不用再到关联的对象底下去获取了，因为`final`就是不变，直接调用相关指令去常量池，或者直接操作数取就好。

对于引用类型，这个基本要到运行时才能确认他们的引用地址，所以加不加`final`都是一样。

看来字节码才是最能确定java是怎么运行的，所以与其找网上的说法不如`javap`一下看看。



