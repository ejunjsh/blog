---
title: 用javap看一下final是什么
date: 2018-07-17 22:23:29
tags: [javap]
categories: java
---

> 一直很好奇`final`的类字段在class文件是怎么表示的，所以用javap看看怎么回事

# 没有final修饰的类

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

Classfile /root/test.class
  Last modified Jul 17, 2018; size 506 bytes
  MD5 checksum 0616d08e9bc479a61a8d637d6963da06
  Compiled from "test.java"
public class test
  SourceFile: "test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #7.#19         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #20.#21        //  java/lang/System.out:Ljava/io/PrintStream;
   #3 = Fieldref           #6.#22         //  test.str:Ljava/lang/String;
   #4 = Methodref          #23.#24        //  java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = String             #25            //  严
   #6 = Class              #26            //  test
   #7 = Class              #27            //  java/lang/Object
   #8 = Utf8               str
   #9 = Utf8               Ljava/lang/String;
  #10 = Utf8               <init>
  #11 = Utf8               ()V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               <clinit>
  #17 = Utf8               SourceFile
  #18 = Utf8               test.java
  #19 = NameAndType        #10:#11        //  "<init>":()V
  #20 = Class              #28            //  java/lang/System
  #21 = NameAndType        #29:#30        //  out:Ljava/io/PrintStream;
  #22 = NameAndType        #8:#9          //  str:Ljava/lang/String;
  #23 = Class              #31            //  java/io/PrintStream
  #24 = NameAndType        #32:#33        //  println:(Ljava/lang/String;)V
  #25 = Utf8               严
  #26 = Utf8               test
  #27 = Utf8               java/lang/Object
  #28 = Utf8               java/lang/System
  #29 = Utf8               out
  #30 = Utf8               Ljava/io/PrintStream;
  #31 = Utf8               java/io/PrintStream
  #32 = Utf8               println
  #33 = Utf8               (Ljava/lang/String;)V
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

上面的操作就是把常量赋值给类的静态字段`str`,这个字段之后在`main`函数会读出来。

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

# 有final修饰

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

Classfile /root/test.class
  Last modified Jul 17, 2018; size 464 bytes
  MD5 checksum 561fcdacae7be4b6da3c78fd99b87719
  Compiled from "test.java"
public class test
  SourceFile: "test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#18         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #19.#20        //  java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #21            //  严
   #4 = Methodref          #22.#23        //  java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #24            //  test
   #6 = Class              #25            //  java/lang/Object
   #7 = Utf8               str
   #8 = Utf8               Ljava/lang/String;
   #9 = Utf8               ConstantValue
  #10 = Utf8               <init>
  #11 = Utf8               ()V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               SourceFile
  #17 = Utf8               test.java
  #18 = NameAndType        #10:#11        //  "<init>":()V
  #19 = Class              #26            //  java/lang/System
  #20 = NameAndType        #27:#28        //  out:Ljava/io/PrintStream;
  #21 = Utf8               严
  #22 = Class              #29            //  java/io/PrintStream
  #23 = NameAndType        #30:#31        //  println:(Ljava/lang/String;)V
  #24 = Utf8               test
  #25 = Utf8               java/lang/Object
  #26 = Utf8               java/lang/System
  #27 = Utf8               out
  #28 = Utf8               Ljava/io/PrintStream;
  #29 = Utf8               java/io/PrintStream
  #30 = Utf8               println
  #31 = Utf8               (Ljava/lang/String;)V
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
你会发现class文件没有之前的静态块了，而是直接在`main`函数里面直接调用常量池里`final`字段。

````
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
`ldc`命令就是直接把常量池的值压如操作数栈。

接下来看看非静态的字段在加`final`或不加会不会有什么不同呢

# 不加final的字段

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

留意下`getfield`指令，主要就是获取`t`实例的`str`的值，其他的都是差不多的了

# 加`final`

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

```
`getfield`不见了，变成了熟悉的`ldc`,很明显了，加了`final`之后还是去常量池去找。


# 其他东西

## 静态字段的赋值

上面的静态字段在没有final的情况下，是生成一段静态块的，这块代码在类加载的时候执行，在加final之后，所有用到这个静态字段都变成直接去常量池里面取，所以就没必要一个一个静态块了

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

## 实例字段的赋值

明明没有加任何构造函数（使用默认构造函数），可是构造函数里面居然有那么多代码，仔细看一下，其实就是实例字段的赋值都放在这里了，只是不管有没有加`final`，都没有区别。

````
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

````

# 总结

看来字节码才是最能确定java是怎么运行的，所以与其找网上的说法不如`javap`一下看看。



