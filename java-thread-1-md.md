---
title: Java多线程(一)基础
date: 2016-08-08 22:06:08
tags: java
categories: java
---
> 对多线程复习下，汇总一下

<!-- more -->
# 什么是线程以及多线程与进程的区别
在现代操作在运行一个程序时，会为其创建一个进程。例如启动一个QQ程序，操作系统就会为其创建一个进程。而操作系统中调度的最小单位元是线程，也叫轻量级进程，在一个进程里可以创建多个线程，这些线程都拥有各自的计数器，堆栈和局部变量等属性，并且能够访问共享的内存变量。处理器在这些线程上高速切换，让使用者感觉到这些线程在同时执行。因此我们可以这样理解：
进程：正在运行的程序，是系统进行资源分配和调用的独立单位。每一个进程都有它自己的内存空间和系统资源。
线程：是进程中的单个顺序控制流，是一条执行路径一个进程如果只有一条执行路径，则称为单线程程序。一个进程如果有多条执行路径，则称为多线程程序。

# 多线程的创建与启动
创建多线程有两种方法，一种是继承Thread类重写run方法，另一种是实现Runnable接口重写run方法。
下面我们分别给出代码示例，继承Thread类重写run方法：
````java
package com.sky.code.thread;

public class NewThread extends Thread {

    @Override
    public void run(){
        System.out.println("I'm a thread that extends Thread!");
    }
}
````
实现Runnable接口重写run方法:
````java
package com.sky.code.thread;

public class NewRunnable implements Runnable{


    @Override
    public void run() {
        System.out.println("I'm a thread that implements Runnable !");
    }
}
````
怎么启动线程？
````java
package com.sky.code.thread;

public class StartThread {
    public static void main(String[] args){
        
        NewThread t1=new NewThread();
        t1.start();

        NewRunnable r=new NewRunnable();
        Thread t2=new Thread(r);
        t2.start();
    }
}
````
运行结果：
````
I'm a thread that extends Thread!
I'm a thread that implements Runnable !
````
代码相当简单，不过多解释。这里有点需要注意的是调用start()方法后并不是是立即的执行多线程的代码，而是使该线程变为可运行态，什么时候运行多线程代码是由操作系统决定的。
# 中断线程和守护线程以及线程优先级
## 什么是中断线程？
我们先来看看中断线程是什么？(该解释来自java核心技术一书，我对其进行稍微简化)，当线程的run()方法执行方法体中的最后一条语句后，并经由执行return语句返回时，或者出现在方法中没有捕获的异常时线程将终止。在java早期版本中有一个stop方法，其他线程可以调用它终止线程，但是这个方法现在已经被弃用了，因为这个方法会造成一些线程不安全的问题。我们可以把中断理解为一个标识位的属性，它表示一个运行中的线程是否被其他线程进行了中断操作，而中断就好比其他线程对该线程打可个招呼，其他线程通过调用该线程的interrupt方法对其进行中断操作，当一个线程调用interrupt方法时，线程的中断状态（标识位）将被置位（改变），这是每个线程都具有的boolean标志，每个线程都应该不时的检查这个标志，来判断线程是否被中断。而要判断线程是否被中断，我们可以使用如下代码
````java
Thread.currentThread().isInterrupted()
````
````java
while(!Thread.currentThread().isInterrupted()){  
    do something  
} 
````
但是如果此时线程处于阻塞状态（sleep或者wait），就无法检查中断状态，此时会抛出InterruptedException异常。如果每次迭代之后都调用sleep方法（或者其他可中断的方法），isInterrupted检测就没必要也没用处了，如果在中断状态被置位时调用sleep方法，它不会休眠反而会清除这一休眠状态并抛出InterruptedException。所以如果在循环中调用sleep,不要去检测中断状态，只需捕获InterruptedException。代码范例如下：
````java
public void run(){  
        while(more work to do ){  
            try {  
                Thread.sleep(5000);  
            } catch (InterruptedException e) {  
                //thread was interrupted during sleep  
                e.printStackTrace();  
            }finally{  
                //clean up , if required  
            }  
}  
````
不妥的处理方式：
````java
void myTask(){  
    ...  
   try{  
       sleep(50)  
      }catch(InterruptedException e){  
   ...  
   }  
}  
````
正确的处理方式：
````java
void myTask()throw InterruptedException{  
    sleep(50)  
} 
````
或者
````java
void myTask(){  
    ...  
    try{  
    sleep(50)  
    }catch(InterruptedException e){  
     Thread.currentThread().interrupt();  
    }  
}  
````
最后关于中断线程，我们这里给出中断线程的一些主要方法：
void interrupt()：向线程发送中断请求，线程的中断状态将会被设置为true，如果当前线程被一个sleep调用阻塞，那么将会抛出interrupedException异常。
static boolean interrupted()：测试当前线程（当前正在执行命令的这个线程）是否被中断。注意这是个静态方法，调用这个方法会产生一个副作用那就是它会将当前线程的中断状态重置为false。
boolean isInterrupted()：判断线程是否被中断，这个方法的调用不会产生副作用即不改变线程的当前中断状态。
static Thread currentThread() : 返回代表当前执行线程的Thread对象。

**这里要注意下，为啥上面的代码，在catch之后还要在中断一次，因为catch会把当前线程的中断标志重置为false，这里不重新中断一次，上层代码就不知道中断了，程序就不知道有中断的发生，下面代码可以说明这个**
````java
package com.sky.code.thread;

/**
 * Created by ejunjsh on 8/10/2017.
 */
public class TestInterrupt {

    public static void main(String[] args)
    {
        Thread t= new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(5000);
                }
                catch (InterruptedException e){
                    //catch 异常之后，输出是false
                    System.out.println("1.current interrupted flag is " +Thread.currentThread().isInterrupted());
                    Thread.currentThread().interrupt();
                    System.out.println("2.current interrupted flag is " +Thread.currentThread().isInterrupted());
                }

                try {
                    Thread.sleep(5000);
                }
                catch (InterruptedException e){
                    System.out.println("3.current interrupted flag is " +Thread.currentThread().isInterrupted());
                    Thread.currentThread().interrupt();
                    System.out.println("4.current interrupted flag is " +Thread.currentThread().isInterrupted());
                }
            }
        });

        t.start();
        //开始中断
        t.interrupt();

        try {
            t.join();
        } catch (InterruptedException e) {
        }

        System.out.println("5.current state is " +t.getState());
    }
}
````
输出结果:
````
1.current interrupted flag is false
2.current interrupted flag is true
3.current interrupted flag is false
4.current interrupted flag is true
5.current state is TERMINATED
````
很明显，`开始中断`后，catch的标志位被重置了。

## 什么是守护线程？
首先我们可以通过t.setDaemon(true)的方法将线程转化为守护线程。而守护线程的唯一作用就是为其他线程提供服务。计时线程就是一个典型的例子，它定时地发送“计时器滴答”信号告诉其他线程去执行某项任务。当只剩下守护线程时，虚拟机就退出了，因为如果只剩下守护线程，程序就没有必要执行了。另外JVM的垃圾回收、内存管理等线程都是守护线程。还有就是在做数据库应用时候，使用的数据库连接池，连接池本身也包含着很多后台线程，监控连接个数、超时时间、状态等等。最后还有一点需要特别注意的是在java虚拟机退出时Daemon线程中的finally代码块并不一定会执行哦，代码示例：
````java
package com.sky.code.thread;


public class Deamon {
    public static void main(String[] args) {
        Thread deamon = new Thread(new DaemonRunner(),"DaemonRunner");
        //设置为守护线程
        deamon.setDaemon(true);
        deamon.start();//启动线程
    }


    static class DaemonRunner implements Runnable{
        @Override
        public void run() {
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally{
                System.out.println("这里的代码在java虚拟机退出时并不一定会执行哦！");
            }
        }
    }
}
````
因此在构建Daemon线程时，不能依靠finally代码块中的内容来确保执行关闭或清理资源的逻辑。

## 什么是线程优先级
在现代操作系统中基本采用时分的形式调度运行的线程，操作系统会分出一个个时间片，线程会分配到若干时间片，当线程的时间片用完了就会发生线程调度，并等待着下一次分配。线程分配到的时间片多少也决定了线程使用处理器资源的多少，而线程优先级就是决定线程需要多或者少分配一些处理器资源的线程属性。在java线程中，通过一个整型的成员变量Priority来控制线程优先级，每一个线程有一个优先级，默认情况下，一个线程继承它父类的优先级。可以用setPriority方法提高或降低任何一个线程优先级。可以将优先级设置在MIN\_PRIORITY（在Thread类定义为1）与MAX\_PRIORITY（在Thread类定义为10）之间的任何值。线程的默认优先级为NORM_PRIORITY（在Thread类定义为5）。尽量不要依赖优先级，如果确实要用，应该避免初学者常犯的一个错误。如果有几个高优先级的线程没有进入非活动状态，低优先级线程可能永远也不能执行。每当调度器决定运行一个新线程时，首先会在具有高优先级的线程中进行选择，尽管这样会使低优先级的线程可能永远不会被执行到。因此我们在设置优先级时，针对频繁阻塞（休眠或者I/O操作）的线程需要设置较高的优先级，而偏重计算（需要较多CPU时间或者运算）的线程则设置较低的优先级，这样才能确保处理器不会被长久独占。当然还有要注意就是在不同的JVM以及操作系统上线程的规划存在差异，有些操作系统甚至会忽略对线程优先级的设定，如mac os系统或者Ubuntu系统........

# 线程的状态转化关系
1.新建状态（New）：新创建了一个线程对象。
2.就绪状态（Runnable）：线程对象创建后，其他线程调用了该对象的start()方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。
3.运行状态（Running）：就绪状态的线程获取了CPU，执行程序代码。
4.阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：
* 等待阻塞（WAITING）：运行的线程执行wait()方法，JVM会把该线程放入等待池中。
* 同步阻塞（Blocked）：运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池中。
* 超时阻塞（TIME_WAITING）：运行的线程执行sleep(long)或join(long)方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。

5.死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。
[![](http://idiotsky.me/images/java-thread-1-1.png)](http://idiotsky.me/images/java-thread-1-1.png)
图中的方法解析如下：
Thread.sleep()：在指定时间内让当前正在执行的线程暂停执行，但不会释放"锁标志"。不推荐使用。
Thread.sleep(long)：使当前线程进入阻塞状态，在指定时间内不会执行。 
Object.wait()和Object.wait(long)：在其他线程调用对象的notify或notifyAll方法前，导致当前线程等待。线程会释放掉它所占有的"锁标志"，从而使别的线程有机会抢占该锁。 当前线程必须拥有当前对象锁。如果当前线程不是此锁的拥有者，会抛出IllegalMonitorStateException异常。 唤醒当前对象锁的等待线程使用notify或notifyAll方法，也必须拥有相同的对象锁，否则也会抛出IllegalMonitorStateException异常，waite()和notify()必须在synchronized函数或synchronized中进行调用。如果在non-synchronized函数或non-synchronized中进行调用,虽然能编译通过，但在运行时会发生IllegalMonitorStateException的异常。 
Object.notifyAll()：则从对象等待池中唤醒所有等待等待线程
Object.notify()：则从对象等待池中唤醒其中一个线程。
Thread.yield()方法 暂停当前正在执行的线程对象，yield()只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行，yield()只能使同优先级或更高优先级的线程有执行的机会。 
Thread.Join()：把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。比如在线程B中调用了线程A的Join()方法，直到线程A执行完毕后，才会继续执行线程B。
好了。本篇线程基础知识介绍到此结束。

参考 http://blog.csdn.net/javazejian/article/details/50878598
所有代码在 https://github.com/ejunjsh/java-code/tree/master/com/sky/code/thread