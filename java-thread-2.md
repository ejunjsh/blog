---
title: Java多线程(二) 同步和锁
date: 2016-08-20 22:55:34
tags: [java,多线程]
categories: java
---
> 记录，汇总

<!-- more -->
# 线程同步问题的产生
什么是线程同步问题，我们先来看一段卖票系统的代码，然后再分析这个问题：
````java
package com.sky.code.thread;


public class TicketSeller implements Runnable {
    private  int num = 100;
    public void run()
    {
        while(true)
        {
            if(num>0)
            {
                try{
                    Thread.sleep(10);
                }catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                //输出卖票信息
                System.out.println(Thread.currentThread().getName()+".....sale...."+num--);
            }
            else {
                break;
            }
        }
    }
}
````
上面是卖票线程类，下来再来看看执行类：
````java
package com.sky.code.thread;


public class TickeDemo {
    public static void main(String[] args)
    {
        TicketSeller t = new TicketSeller();//创建一个线程任务对象。

        //创建4个线程同时卖票
        Thread t1 = new Thread(t);
        Thread t2 = new Thread(t);
        Thread t3 = new Thread(t);
        Thread t4 = new Thread(t);
        //启动线程
        t1.start();
        t2.start();
        t3.start();
        t4.start();
    }
}
````
运行程序结果如下（仅截取部分数据）：
[![](http://idiotsky.me/images/java-thread-2-1.png)](http://idiotsky.me/images/java-thread-2-1.png)
从运行结果，我们就可以看出我们3个售票窗口同时卖出了96号票，这显然是不合逻辑的，其实这个问题就是我们前面所说的线程同步问题。不同的线程都对同一个数据进了操作这就容易导致数据错乱的问题，也就是线程不同步。那么这个问题该怎么解决呢？在给出解决思路之前我们先来分析一下这个问题是怎么产生的？我们声明一个线程类TicketSeller，在这个类中我们又声明了一个成员变量num也就是票的数量，然后我们通过run方法不断的去获取票数并输出，最后我们在外部类TicketDemo中创建了四个线程同时操作这个数据，运行后就出现我们刚才所说的线程同步问题，从这里我们可以看出产生线程同步(线程安全)问题的条件有两个：1.多个线程在操作共享的数据（num），2.操作共享数据的线程代码有多条（4条线程）；既然原因知道了，那该怎么解决？
>解决思路：将多条操作共享数据的线程代码封装起来，当有线程在执行这些代码的时候，其他线程时不可以参与运算的。必须要当前线程把这些代码都执行完毕后，其他线程才可以参与运算。 好了，思路知道了，我们就用java代码的方式来解决这个问题。

# 解决线程同步的两种典型方案
在java中有两种机制可以防止线程安全的发生，Java语言提供了一个synchronized关键字来解决这问题，同时在Java SE5.0引入了Lock锁对象的相关类，接下来我们分别介绍这两种方法
## 通过锁（Lock）对象的方式解决线程安全问题
在给出解决代码前我们先来介绍一个知识点：Lock，锁对象。在java中锁是用来控制多个线程访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源（但有的锁可以允许多个线程并发访问共享资源，比如读写锁，后面我们会分析）。在Lock接口出现之前，java程序是靠synchronized关键字（后面分析）实现锁功能的，而JAVA SE5.0之后并发包中新增了Lock接口用来实现锁的功能，它提供了与synchronized关键字类似的同步功能，只是在使用时需要显式地获取和释放锁，缺点就是缺少像synchronized那样隐式获取释放锁的便捷性，但是却拥有了锁获取与释放的可操作性，可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步特性。接下来我们就来介绍Lock接口的主要API方便我们学习

方法|相关描述内容 
---|--------
void lock()|获取锁，调用该方法当前线程会获取锁，当获取锁后。从该方法返回
void lockInterruptibly() throws InterruptedException|可中断获取锁和lock()方法不同的是该方法会响应中断，即在获取锁中可以中断当前线程。例如某个线程在等待一个锁的控制权的这段时间需要中断。
boolean tryLock()|尝试非阻塞获取锁，调用该方法后立即返回，如果能够获取锁则返回true，否则返回false。
boolean tryLock(long time,TimeUnit unit) throws  InterruptedException |超时获取锁，当前线程在以下3种情况返回：1.当前线程在超时时间内获取了锁2.当前线程在超时时间被中断3.当前线程超时时间结束，返回false
void unlock()|释放锁
Condition newCondition()|条件对象，获取等待通知组件。该组件和当前的锁绑定，当前线程只有获取了锁，才能调用该组件的await()方法，而调用后，当前线程将释放锁。

这里先介绍一下API，接下来我们将结合Lock接口的实现子类ReentrantLock来讲解下他的几个方法。
### ReentrantLock（重入锁)
重入锁，顾名思义就是支持重新进入的锁，它表示该锁能够支持一个线程对资源的重复加锁，也就是说在调用lock()方法时，已经获取到锁的线程，能够再次调用lock()方法获取锁而不被阻塞，同时还支持获取锁的公平性和非公平性。这里的公平是在绝对时间上，先对锁进行获取的请求一定先被满足，那么这个锁是公平锁，反之，是不公平的(但是如果不是需要，建议不要用公平锁，因为会造成一些资源的没必要等待，浪费性能)。那么该如何使用呢？看范例代码：
1.同步执行的代码跟synchronized类似功能：
````java
ReentrantLock lock = new ReentrantLock(); //参数默认false，不公平锁    
ReentrantLock lock = new ReentrantLock(true); //公平锁    
    
lock.lock(); //如果被其它资源锁定，会在此等待锁释放，达到暂停的效果    
try {    
    //操作    
} finally {    
    lock.unlock();  //释放锁  
}  
````
2.防止重复执行代码：
````java
ReentrantLock lock = new ReentrantLock();    
if (lock.tryLock()) {  //如果已经被lock，则立即返回false不会等待，达到忽略操作的效果     
    try {    
        //操作    
    } finally {    
        lock.unlock();    
   }    
}    
````
3.尝试等待执行的代码：
````java
ReentrantLock lock = new ReentrantLock(true); //公平锁    
try {    
    if (lock.tryLock(5, TimeUnit.SECONDS)) {        
        //如果已经被lock，尝试等待5s，看是否可以获得锁，如果5s后仍然无法获得锁则返回false继续执行    
       try {    
            //操作    
        } finally {    
            lock.unlock();    
        }    
    }    
} catch (InterruptedException e) {    
    e.printStackTrace(); //当前线程被中断时(interrupt)，会抛InterruptedException                     
}  
````
这里有点需要特别注意的，把解锁操作放在finally代码块内这个十分重要。如果在临界区的代码抛出异常，锁必须被释放。否则，其他线程将永远阻塞。好了，ReentrantLock我们就简单介绍到这里，接下来我们通过ReentrantLock来解决前面卖票线程的线程同步（安全）问题，代码如下：
````java
package com.sky.code.thread;


import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class TicketSellerWithLock implements Runnable {

    //创建锁对象
    private Lock ticketLock = new ReentrantLock();

    //创建锁对象(公平锁)
    //private Lock ticketLock = new ReentrantLock(true);

    private  int num = 100;
    public void run()
    {
        while(true)
        {
            ticketLock.lock();//获取锁
            if(num>0)
            {
                try{
                    Thread.sleep(10);
                    //输出卖票信息
                    System.out.println(Thread.currentThread().getName()+".....sale...."+num--);
                }catch (InterruptedException e)
                {
                    Thread.currentThread().interrupt();//继续中断异常
                }finally {
                    ticketLock.unlock();//释放锁
                }

            }
            else {
                ticketLock.unlock();//释放锁
                break;
            }
        }
    }
}

````
TicketDemo类无需变化，线程安全问题就此解决。
但是还是要说一下公平锁的问题，上面例子，不开公平锁的结果如下：
[![](http://idiotsky.me/images/java-thread-2-2.png)](http://idiotsky.me/images/java-thread-2-2.png)
开公平锁的结果如下：
[![](http://idiotsky.me/images/java-thread-2-3.png)](http://idiotsky.me/images/java-thread-2-3.png)
*你会发现不开公平锁，cpu钟爱用第一个线程做事情，而开了公平锁后，基本是各个线程交替执行。上面提到公平锁是会消耗性能的，如果CPU调度的时候选择的不是公平调度的那个线程，CPU会放弃本次调度，干别的事情，如果老是调度不到的话，是浪费CPU调度的。*

## 通过synchronied关键字的方式解决线程安全问题
在Java中内置了语言级的同步原语－synchronized，这个可以大大简化了Java中多线程同步的使用。从JAVA SE1.0开始，java中的每一个对象都有一个内部锁，如果一个方法使用synchronized关键字进行声明，那么这个对象将保护整个方法，也就是说调用该方法线程必须获得内部的对象锁。
````java
public synchronized void method{  
  //method body  
}  
````
等价于
````java
private Lock ticketLock = new ReentrantLock();  
public void method{  
 ticketLock.lock();  
 try{  
  //.......  
 }finally{  
   ticketLock.unlock();  
 }  
} 
````
从这里可以看出使用synchronized关键字来编写代码要简洁得多了。当然，要理解这一代码，我们必须知道每个对象有一个内部锁，并且该锁有一个内部条件。由锁来管理那些试图进入synchronized方法的线程，由条件来管那些调用wait的线程(wait()/notifyAll/notify())。同时我们必须明白一旦有一个线程通过synchronied方法获取到内部锁，该类的所有synchronied方法或者代码块都无法被其他线程访问直到当前线程释放了内部锁。刚才上面说的是同步方法，synchronized还有一种同步代码块的实现方式：
````java
Object obj = new Object();  
synchronized(obj){  
  //需要同步的代码  
}  
````
其中obj是对象锁，可以是任意对象。那么我们就通过其中的一个方法来解决售票系统的线程同步问题：
````java
class Ticket implements Runnable  
{  
    private  int num = 100;  
    Object obj = new Object();  
    public void run()  
    {  
        while(true)  
        {  
            synchronized(obj)  
            {  
                if(num>0)  
                {  
                    try{Thread.sleep(10);}catch (InterruptedException e){}  
                      
                    System.out.println(Thread.currentThread().getName()+".....sale...."+num--);  
                }  
            }  
        }  
    }  
}  
````
嗯，同步代码块解决，运行结果也正常。到此同步问题也就解决了，当然代码同步也是要牺牲效率为前提的：
同步的好处：解决了线程的安全问题。
同步的弊端：相对降低了效率，因为同步外的线程的都会判断同步锁。
同步的前提：同步中必须有多个线程并使用同一个锁。

# 线程间的通信机制
线程开始运行，拥有自己的栈空间，但是如果每个运行中的线程，如果仅仅是孤立地运行，那么没有一点儿价值，或者是价值很小，如果多线程能够相互配合完成工作的话，这将带来巨大的价值，这也就是线程间的通信啦。在java中多线程间的通信使用的是等待/通知机制来实现的。
## synchronied关键字等待/通知机制
是指一个线程A调用了对象O的wait()方法进入等待状态，而另一个线程B调用了对象O的notify()或者notifyAll()方法，线程A收到通知后从对象O的wait()方法返回，进而执行后续操作。上述的两个线程通过对象O来完成交互，而对象上的wait()和notify()/notifyAll()的关系就如同开关信号一样，用来完成等待方和通知方之间的交互工作。
等待/通知机制主要是用到的函数方法是notify()/notifyAll(),wait()/wait(long),wait(long,int),这些方法在上一篇文章都有说明过，这里就不重复了。当然这是针对synchronied关键字修饰的函数或代码块，因为要使用notify()/notifyAll(),wait()/wait(long),wait(long,int)这些方法的前提是对调用对象加锁，也就是说只能在同步函数或者同步代码块中使用。
## 条件对象的等待/通知机制
所谓的条件对象也就是配合前面我们分析的Lock锁对象，通过锁对象的条件对象来实现等待/通知机制。那么条件对象是怎么创建的呢？
````java
//创建条件对象  
Condition conditionObj=ticketLock.newCondition();  
````
就这样我们创建了一个条件对象。注意这里返回的对象是与该锁（ticketLock）相关的条件对象。下面是条件对象的API：

方法	|函数方法对应的描述
-----|-------           
void await()|	将该线程放到条件等待池中（对应wait()方法）
void signalAll()|	解除该条件等待池中所有线程的阻塞状态（对应notifyAll()方法）
void signal()|	从该条件的等待池中随机地选择一个线程，解除其阻塞状态（对应notify()方法）

上述方法的过程分析：一个线程A调用了条件对象的await()方法进入等待状态，而另一个线程B调用了条件对象的signal()或者signalAll()方法，线程A收到通知后从条件对象的await()方法返回，进而执行后续操作。上述的两个线程通过条件对象来完成交互，而对象上的await()和signal()/signalAll()的关系就如同开关信号一样，用来完成等待方和通知方之间的交互工作。当然这样的操作都是必须基于条件对象的锁的，当前线程只有获取了锁，才能调用该条件对象的await()方法，而调用后，当前线程将释放锁。

这里有点要特别注意的是，上述两种等待/通知机制中，无论是调用了signal()/signalAll()方法还是调用了notify()/notifyAll()方法并不会立即激活一个等待线程。它们仅仅都只是解除等待线程的阻塞状态，以便这些线程可以在当前线程解锁或者退出同步方法后，通过争夺CPU执行权实现对对象的访问。到此，线程通信机制的概念分析完，我们下面通过生产者消费者模式来实现等待/通知机制。

# 生产者消费者模式
## 单生产者单消费者模式
顾名思义，就是一个线程消费，一个线程生产。我们先来看看等待/通知机制下的生产者消费者模式：我们假设这样一个场景，我们是卖北京烤鸭店铺，我们现在只有一条生产线也只有一条消费线，也就是说只能生产线程生产完了，再通知消费线程才能去卖，如果消费线程没烤鸭了，就必须通知生产线程去生产，此时消费线程进入等待状态。在这样的场景下，我们不仅要保证共享数据（烤鸭数量）的线程安全，而且还要保证烤鸭数量在消费之前必须有烤鸭。下面我们通过java代码来实现：
北京烤鸭生产资源类KaoYaResource：
````java
package com.sky.code.thread;


public class KaoYaResource {
    private String name;
    private int count = 1;//烤鸭的初始数量  
    private boolean flag = false;//判断是否有需要线程等待的标志  

    /**
     * 生产烤鸭 
     */
    public synchronized void product(String name){
        if(flag){
            //此时有烤鸭，等待  
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace()
                ;
            }
        }
        this.name=name+count;//设置烤鸭的名称  
        count++;
        System.out.println(Thread.currentThread().getName()+"...生产者..."+this.name);
        flag=true;//有烤鸭后改变标志  
        notifyAll();//通知消费线程可以消费了  
    }

    /**
     * 消费烤鸭 
     */
    public synchronized void consume(){
        if(!flag){//如果没有烤鸭就等待  
            try{
                this.wait();
            }catch(InterruptedException e){
                
            }
        }
        System.out.println(Thread.currentThread().getName()+"...消费者........"+this.name);//消费烤鸭1  
        flag = false;
        notifyAll();//通知生产者生产烤鸭  
    }
}
````
在这个类中我们有两个synchronized的同步方法，一个是生产烤鸭的，一个是消费烤鸭的，之所以需要同步是因为我们操作了共享数据count，同时为了保证生产烤鸭后才能消费也就是生产一只烤鸭后才能消费一只烤鸭，我们使用了等待/通知机制，wait()和notify()。当第一次运行生产现场时调用生产的方法，此时有一只烤鸭，即flag=false，无需等待，因此我们设置可消费的烤鸭名称然后改变flag=true，同时通知消费线程可以消费烤鸭了，即使此时生产线程再次抢到执行权，因为flag=true，所以生产线程会进入等待阻塞状态，消费线程被唤醒后就进入消费方法，消费完成后，又改变标志flag=false，通知生产线程可以生产烤鸭了.........以此循环。
生产消费执行类Single\_Producer\_Consumer.java:
````java
package com.sky.code.thread;


public class Single_Producer_Consumer {

    public static void main(String[] args)
    {
        KaoYaResource r = new KaoYaResource();
        Producer pro = new Producer(r);
        Consumer con = new Consumer(r);
        //生产者线程
        Thread t0 = new Thread(pro);
        //消费者线程
        Thread t2 = new Thread(con);
        //启动线程
        t0.start();
        t2.start();
    }
}

class Producer implements Runnable
{
    private KaoYaResource r;
    Producer(KaoYaResource r)
    {
        this.r = r;
    }
    public void run()
    {
        while(true)
        {
            r.product("北京烤鸭");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Consumer implements Runnable
{
    private KaoYaResource r;
    Consumer(KaoYaResource r)
    {
        this.r = r;
    }
    public void run()
    {
        while(true)
        {
            r.consume();
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
````
在这个类中我们创建两个线程，一个是消费者线程，一个是生产者线程，我们分别开启这两个线程用于不断的生产消费，运行结果如下：
````
hread-0...生产者...北京烤鸭1
Thread-1...消费者........北京烤鸭1
Thread-0...生产者...北京烤鸭2
Thread-1...消费者........北京烤鸭2
Thread-0...生产者...北京烤鸭3
Thread-1...消费者........北京烤鸭3
Thread-0...生产者...北京烤鸭4
Thread-1...消费者........北京烤鸭4
Thread-0...生产者...北京烤鸭5
Thread-1...消费者........北京烤鸭5
Thread-0...生产者...北京烤鸭6
Thread-1...消费者........北京烤鸭6
Thread-0...生产者...北京烤鸭7
Thread-1...消费者........北京烤鸭7
Thread-0...生产者...北京烤鸭8
Thread-1...消费者........北京烤鸭8
Thread-0...生产者...北京烤鸭9
Thread-1...消费者........北京烤鸭9
.....
````
很显然的情况就是生产一只烤鸭然后就消费一只烤鸭。运行情况完全正常，嗯，这就是单生产者单消费者模式。上面使用的是synchronized关键字的方式实现的，那么接下来我们使用对象锁的方式实现：KaoYaResourceByLock.java
````java
package com.sky.code.thread;


import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class KaoYaResourceByLock {
    private String name;
    private int count = 1;//烤鸭的初始数量
    private boolean flag = false;//判断是否有需要线程等待的标志
    //创建一个锁对象
    private Lock resourceLock=new ReentrantLock();
    //创建条件对象
    private Condition condition= resourceLock.newCondition();
    /**
     * 生产烤鸭
     */
    public  void product(String name){
        resourceLock.lock();//先获取锁
        try{
            if(flag){
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            this.name=name+count;//设置烤鸭的名称
            count++;
            System.out.println(Thread.currentThread().getName()+"...生产者..."+this.name);
            flag=true;//有烤鸭后改变标志
            condition.signalAll();//通知消费线程可以消费了
        }finally{
            resourceLock.unlock();
        }
    }

    /**
     * 消费烤鸭
     */
    public  void consume(){
        resourceLock.lock();
        try{
            if(!flag){//如果没有烤鸭就等待
                try{
                    condition.await();
                }catch(InterruptedException e){

                }
            }
            System.out.println(Thread.currentThread().getName()+"...消费者........"+this.name);//消费烤鸭1
            flag = false;
            condition.signalAll();//通知生产者生产烤鸭
        }finally{
            resourceLock.unlock();
        }
    }
}

````
代码变化不大，我们通过对象锁的方式去实现，首先要创建一个对象锁，我们这里使用的重入锁ReestrantLock类，然后通过手动设置lock()和unlock()的方式去获取锁以及释放锁。为了实现等待/通知机制，我们还必须通过锁对象去创建一个条件对象Condition，然后通过锁对象的await()和signalAll()方法去实现等待以及通知操作。Single\_Producer\_Consumer.java代码替换一下资源类即可,运行结果一样。

## 多生产者多消费者模式
分析完了单生产者单消费者模式，我们再来聊聊多生产者多消费者模式，也就是多条生产线程配合多条消费线程。既然这样的话我们先把上面的代码Single\_Producer\_Consumer.java类修改成新类，大部分代码不变，仅新增2条线程去跑，一条t1的生产  共享资源类KaoYaResource不作更改，代码如下：
````java
package com.sky.code.thread;


public class Mutil_Producer_Consumer {
    public static void main(String[] args)
    {
        KaoYaResource r = new KaoYaResource();
        Mutil_Producer pro = new Mutil_Producer(r);
        Mutil_Consumer con = new Mutil_Consumer(r);
        //生产者线程
        Thread t0 = new Thread(pro);
        Thread t1 = new Thread(pro);
        //消费者线程
        Thread t2 = new Thread(con);
        Thread t3 = new Thread(con);
        //启动线程
        t0.start();
        t1.start();
        t2.start();
        t3.start();
    }
}

class Mutil_Producer implements Runnable
{
    private KaoYaResource r;
    Mutil_Producer(KaoYaResource r)
    {
        this.r = r;
    }
    public void run()
    {
        while(true)
        {
            r.product("北京烤鸭");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Mutil_Consumer implements Runnable
{
    private KaoYaResource r;
    Mutil_Consumer(KaoYaResource r)
    {
        this.r = r;
    }
    public void run()
    {
        while(true)
        {
            r.consume();
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

````
就多了两条线程，我们运行代码看看，结果如下：
````bash
Thread-0...生产者...北京烤鸭63
Thread-2...消费者........北京烤鸭63
Thread-3...消费者........北京烤鸭63  #消费两次了
......
......
Thread-0...生产者...北京烤鸭67     #没有被消费
Thread-1...生产者...北京烤鸭68     
Thread-2...消费者........北京烤鸭68
Thread-0...生产者...北京烤鸭69
````
不对呀，我们才生产一只烤鸭，怎么就被消费了2次啊，有的烤鸭生产了也没有被消费啊？难道共享数据源没有进行线程同步？回顾下KaoYaResource.java
共享数据count的获取方法都进行synchronized关键字同步了呀！那怎么还会出现数据混乱的现象啊？
分析：确实，我们对共享数据也采用了同步措施，而且也应用了等待/通知机制，但是这样的措施只在单生产者单消费者的情况下才能正确应用，但从运行结果来看，我们之前的单生产者单消费者安全处理措施就不太适合多生产者多消费者的情况了。那么问题出在哪里？可以明确的告诉大家，肯定是在资源共享类，下面我们就来分析问题是如何出现，又该如何解决？直接上图
[![](http://idiotsky.me/images/java-thread-2-4.png)](http://idiotsky.me/images/java-thread-2-4.png)

解决后的资源代码如下只将if改为了while：
````java
package com.sky.code.thread;


public class KaoYaResourceByMulti {
    private String name;
    private int count = 1;//烤鸭的初始数量
    private boolean flag = false;//判断是否有需要线程等待的标志

    /**
     * 生产烤鸭
     */
    public synchronized void product(String name){
        while (flag){
            //此时有烤鸭，等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace()
                ;
            }
        }
        this.name=name+count;//设置烤鸭的名称
        count++;
        System.out.println(Thread.currentThread().getName()+"...生产者..."+this.name);
        flag=true;//有烤鸭后改变标志
        notifyAll();//通知消费线程可以消费了
    }

    /**
     * 消费烤鸭
     */
    public synchronized void consume(){
        while (!flag){//如果没有烤鸭就等待
            try{
                this.wait();
            }catch(InterruptedException e){

            }
        }
        System.out.println(Thread.currentThread().getName()+"...消费者........"+this.name);//消费烤鸭1
        flag = false;
        notifyAll();//通知生产者生产烤鸭
    }
}
````
运行结果跟单线程那个一致，就不贴了。
到此，多消费者多生产者模式也完成，不过上面用的是synchronied关键字实现的，而锁对象的解决方法也一样将之前单消费者单生产者的资源类中的if判断改为while判断即可代码就不贴了哈。不过下面我们将介绍一种更有效的锁对象解决方法，我们准备使用两组条件对象（Condition也称为监视器）来实现等待/通知机制，也就是说通过已有的锁获取两组监视器，一组监视生产者，一组监视消费者。有了前面的分析这里我们直接上代码：
````java
package com.sky.code.thread;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;


public class ResourceBy2Condition {
    private String name;
    private int count = 1;
    private boolean flag = false;

    //创建一个锁对象。  
    Lock lock = new ReentrantLock();

    //通过已有的锁获取两组监视器，一组监视生产者，一组监视消费者。  
    Condition producer_con = lock.newCondition();
    Condition consumer_con = lock.newCondition();

    /**
     * 生产 
     * @param name
     */
    public  void product(String name)
    {
        lock.lock();
        try
        {
            while(flag){
                try{producer_con.await();}catch(InterruptedException e){}
            }
            this.name = name + count;
            count++;
            System.out.println(Thread.currentThread().getName()+"...生产者5.0..."+this.name);
            flag = true;
//          notifyAll();  
//          con.signalAll();  
            consumer_con.signal();//直接唤醒消费线程  
        }
        finally
        {
            lock.unlock();
        }
    }

    /**
     * 消费 
     */
    public  void consume()
    {
        lock.lock();
        try
        {
            while(!flag){
                try{consumer_con.await();}catch(InterruptedException e){}
            }
            System.out.println(Thread.currentThread().getName()+"...消费者.5.0......."+this.name);//消费烤鸭1  
            flag = false;
//          notifyAll();  
//          con.signalAll();  
            producer_con.signal();//直接唤醒生产线程  
        }
        finally
        {
            lock.unlock();
        }
    }
}  

````
从代码中可以看到，我们创建了producer\_con 和consumer\_con两个条件对象，分别用于监听生产者线程和消费者线程，在product()方法中，我们获取到锁后，
如果此时flag为true的话，也就是此时还有烤鸭未被消费，因此生产线程需要等待，所以我们调用生产线程的监控器producer_con的
await()的方法进入阻塞等待池；但如果此时的flag为false的话，就说明烤鸭已经消费完，需要生产线程去生产烤鸭，那么生产线程将进行烤
鸭生产并通过消费线程的监控器consumer_con的signal()方法去通知消费线程对烤鸭进行消费。consume()方法也是同样的道理，这里就不
过多分析了。我们可以发现这种方法比我们之前的synchronized同步方法或者是单监视器的锁对象都来得高效和方便些，之前都是使用
notifyAll()和signalAll()方法去唤醒池中的线程，然后让池中的线程又进入 竞争队列去抢占CPU资源，这样不仅唤醒了无关的线程而且又让全
部线程进入了竞争队列中，而我们最后使用两种监听器分别监听生产者线程和消费者线程，这样的方式恰好解决前面两种方式的问题所在，
我们每次唤醒都只是生产者线程或者是消费者线程而不会让两者同时唤醒，这样不就能更高效得去执行程序了吗？好了，到此多生产者多消
费者模式也分析完毕。

# 线程死锁
现在我们再来讨论一下线程死锁问题，从上面的分析，我们知道锁是个非常有用的工具，运用的场景非常多，因为它使用起来非常简单，而
且易于理解。但同时它也会带来一些不必要的麻烦，那就是可能会引起死锁，一旦产生死锁，就会造成系统功能不可用。我们先通过一个例
子来分析，这个例子会引起死锁，使得线程t1和线程t2互相等待对方释放锁。
````java
package com.sky.code.thread;


public class DeadLockDemo {
    private static String A="A";
    private static String B="B";

    public static void main(String[] args) {
        DeadLockDemo deadLock=new DeadLockDemo();

        deadLock.deadLock();

    }

    private void deadLock(){
        Thread t1=new Thread(new Runnable(){
            @SuppressWarnings("static-access")
            @Override
            public void run() {
                synchronized (A) {
                    try {
                        Thread.currentThread().sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (B) {
                        System.out.println("1");
                    }
                }
            }
        });

        Thread t2 =new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (B) {
                    synchronized (A) {
                        System.out.println("2");
                    }
                }
            }
        });

        //启动线程
        t1.start();
        t2.start();
    }
}
````
上面代码运行基本没有输出，一直卡着。
同步嵌套是产生死锁的常见情景，从上面的代码中我们可以看出，当t1线程拿到锁A后，睡眠2秒，此时线程t2刚好拿到了B锁，接着要获取A锁，但是此时A锁正好被t1线程持有，因此只能等待t1线程释放锁A，但遗憾的是在t1线程内又要求获取到B锁，而B锁此时又被t2线程持有，到此结果就是t1线程拿到了锁A同时在等待t2线程释放锁B，而t2线程获取到了锁B也同时在等待t1线程释放锁A，彼此等待也就造成了线程死锁问题。虽然我们现实中一般不会向上面那么写出那样的代码，但是有些更为复杂的场景中，我们可能会遇到这样的问题，比如t1拿了锁之后，因为一些异常情况没有释放锁（死循环），也可能t1拿到一个数据库锁，释放锁的时候抛出了异常，没有释放等等，所以我们应该在写代码的时候多考虑死锁的情况，这样才能有效预防死锁程序的出现。下面我们介绍一下避免死锁的几个常见方法：
1. 避免一个线程同时获取多个锁。
2. 避免在一个资源内占用多个 资源，尽量保证每个锁只占用一个资源。
3. 尝试使用定时锁，使用tryLock(timeout)来代替使用内部锁机制。
4. 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况。
5. 避免同步嵌套的发生

# Thread.join()
如果一个线程A执行了thread.join()语句，其含义是：当前线程A等待thread线程终止之后才能从thread.join()返回。线程Thread除了提供join()方法之外，还提供了join(long millis)和join(long millis,int nanos)两个具备超时特性的方法。这两个超时的方法表示，如果线程在给定的超时时间里没有终止，那么将会从该超时方法中返回。下面给出一个例子，创建10个线程，编号0~9，每个线程调用前一个线程的join()方法，也就是线程0结束了，线程1才能从join()方法中返回，而0需要等待main线程结束。
````java
package com.sky.code.thread;

public class JoinDemo {
    public static void main(String[] args) {
        Thread previous = Thread.currentThread();
        for(int i=0;i<10;i++){
            //每个线程拥有前一个线程的引用。需要等待前一个线程终止，才能从等待中返回
            Thread thread=new Thread(new Domino(previous),String.valueOf(i));
            thread.start();
            previous=thread;
        }
        System.out.println(Thread.currentThread().getName()+" 线程结束");
    }
}
class Domino implements Runnable{
    private Thread thread;
    public Domino(Thread thread){
        this.thread=thread;
    }

    @Override
    public void run() {
        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName()+" 线程结束");
    }

}

````
运行结果：
````
main 线程结束
0 线程结束
1 线程结束
2 线程结束
3 线程结束
4 线程结束
5 线程结束
6 线程结束
7 线程结束
8 线程结束
9 线程结束
````

> 结束

参考 http://blog.csdn.net/javazejian/article/details/50878665
所有代码在 https://github.com/ejunjsh/java-code/tree/master/src/main/java/com/sky/code/thread