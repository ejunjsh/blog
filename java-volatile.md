---
title: java volatile 解惑
date: 2016-08-27 20:23:44
tags: [java,多线程,volatile]
categories: java
---
> volatile 总结很好的文章。

# 前言
volatile关键字可能是Java开发人员“熟悉而又陌生”的一个关键字。本文将从volatile关键字的作用、开销和典型应用场景以及Java虚拟机对volatile关键字的实现这几个方面为读者全面深入剖析volatile关键字。

volatile字面上有“挥发性的，不稳定的”意思，它是用于修饰可变共享变量（Mutable Shared Variable）的一个关键字。所谓“共享”是指一个变量能够被多个线程访问（包括读/写），所谓“可变”是指变量的值可以发生变化。换而言之，volatile关键字用于修饰多个线程并发访问的同一个变量，这些线程中至少有一个线程会更新这个变量的值。我们称volatile修饰的变量为volatile变量。我们知道锁的作用包括保障原子性、保障可见性以及保障有序性。volatile常被称为“轻量级锁”，其作用与锁有类似的地方——volatile也能够保障原子性（仅保障long/double型变量访问操作的原子性）、保障可见性以及保障有序性。

本文所提及的“Java虚拟机”如无特别说明，均特指Oracle公司的HotSpot Java虚拟机。

<!-- more -->
# 保障long/double型变量访问操作的原子性
不可分割的操作被称为原子操作（Atomic Operation）。所谓不可分割（Indivisible）是指一个操作从其执行线程以外的其他线程看来，该操作要么已经完成要么尚未开始，也就是说其他线程不会看到该操作的中间结果。如果一个操作是原子操作，那么我们就称该操作具有原子性（Atomicity）。

Java语言规范（Java Language Specification，JLS）规定，Java语言中针对long/double型以外的任何变量（包括基础类型变量和引用型变量）进行的读、写操作都是原子操作，即Java语言规范本身并不规定针对long/double型变量进行读、写操作具有原子性。一个long/double型变量的读/写操作在32位Java虚拟机下可能会被分解为两个子步骤（比如先写低32位，再写高32位）来实现，这就导致一个线程对long/double型变量进行的写操作的中间结果可以被其他线程所观察到，即此时针对long/double型变量的访问操作不是原子操作。清单1所示的实验展示了这点。

清单1 long/double型变量写操作的原子性问题Demo
````java
/**
 * 
 * 本Demo必须使用32位Java虚拟机才能看到非原子操作的效果. <br>
 * 运行本Demo时也可以指定虚拟机参数“-client”
 * @author Viscent Huang
 * 
 */
public class NonAtomicAssignmentDemo implements Runnable {

    static long value = 0;

    private final long valueToSet;

    public NonAtomicAssignmentDemo(long valueToSet) {

        this.valueToSet = valueToSet;

    }

    public static void main(String[] args) {

        // 线程updateThread1将data更新为0

        Thread updateThread1 = new Thread(new NonAtomicAssignmentDemo(0L));

        // 线程updateThread2将data更新为-1

        Thread updateThread2 = new Thread(new NonAtomicAssignmentDemo(-1L));

        updateThread1.start();

        updateThread2.start();

        // 不进行实际输出的OutputStream

        final DummyOutputStream dos = new DummyOutputStream();

        try (PrintStream dummyPrintSteam = new PrintStream(dos);) {

            // 共享变量value的快照（即瞬间值）

            long snapshot;

            while (0 == (snapshot = value) || -1 == snapshot) {

                // 不进行实际的输出，仅仅是为了阻止JIT编译器做循环不变表达式外提优化

                dummyPrintSteam.print(snapshot);

            }

            System.err.printf("Unexpected data: %d(0x%016x)", snapshot,
                    snapshot);

        }

        System.exit(0);

    }

    static class DummyOutputStream extends OutputStream {

        @Override

        public void write(int b) throws IOException {

            // 不实际进行输出

        }

    }

    @Override

    public void run() {

        for (;;) {

            value = valueToSet;

        }

    }

}
````
使用32位（而不是64位）Java虚拟机运行清单1所示的Demo我们可以看到该程序的输出是：
````
Unexpected data: 4294967295(0x00000000ffffffff)
````
或者，
````
Unexpected data: -4294967296(0xffffffff00000000)
````
可见，main线程读取到共享变量value的值可能既不是0（对应无符号16进制数0x0000000000000000）也不是-1（对应无符号16进制数0xffffffffffffffff），而是其他两个线程更新value时的“中间结果”——4294967295（对应无符号16进制数0x00000000ffffffff）或者-4294967296（对应无符号16进制数0xffffffff00000000），即一个线程对value变量的低（Lower）32位（4个字节）更新与另外一个线程对value变量的高（Higher）32位（4个字节）更新所“混合”出来的一个非预期的错误结果。因此，上述Demo对共享变量value的写操作并非一个原子操作。这是由于：Java平台中，long/double型变量会占用64位（8个字节）的存储空间，而32位的Java虚拟机对这种变量的写操作可能会被分解为两个子步骤来实施，比如先写低32位，再写高32位。那么，多个线程试图共享同一个这样的变量时就可能出现一个线程在写高32位的时候，另外一个线程恰好正在写低32位，而此刻第三个线程读取这个变量时所读取到的变量值仅仅是其他两个线程更新这个变量的中间结果。

32位虚拟机下，一个long/double型变量读操作同样也可能会被分解为两个子步骤来实现，比如先读取低32位到寄存器中，再读取高32位到寄存器中。这种实现同样也会导致与上述Demo所展示的相似的效果，即一个线程可以读取到其他线程对long/double型变量写操作的中间结果。因此，在这种Java虚拟机实现下，long/double型变量读操作同样也不是原子操作。

上述Demo更多的是从系统（Java虚拟机）层面展示原子性问题。那么，在业务层面我们是否也可能遇到类似上述的原子性问题呢？如清单2所示，假设线程T1通过执行updateHostInfo方法来更新主机信息（HostInfo），线程T2则通过执行connectToHost方法来读取主机信息，并据此与相应的主机建立网络连接。那么，updateHostInfo方法中的操作（更新主机IP地址和端口号）必须是一个原子操作，即这个操作必须是“不可分割”的。否则，可能出现这样的情形：假设hostInfo的初始值表示的是IP地址为“192.168.1.101”、端口号为8081的主机，T1执行updateHostInfo方法试图将hostInfo更新为IP地址为“192.168.1.100”、端口号为8080的主机的时候，T2可能刚好执行connectToHost方法，那么此时由于T1可能刚刚执行完语句①而未开始语句②（即只更新完IP地址而尚未更新端口号），因此T2可能读取到IP地址为“192.168.1.100”、而端口号却仍然为8081的主机信息，即T2读取到了一个错误的主机信息（IP地址为“192.168.1.100”的主机上面并没有开启侦听端口8081，它开启的8080）从而无法建立网络连接！这里的错误是由于updateHostInfo方法中的操作不是原子操作（不具备“不可分割”的特性）而使其他线程读取了脏数据（错误的主机信息）导致的。

清单2 业务层面的原子操作问题Demo
````java
public class AtomicityExample {

    private HostInfo hostInfo;

    public void updateHostInfo(String ip, int port) {

        // 以下操作不是原子操作

        hostInfo.setIp(ip);// 语句①

        hostInfo.setPort(port);// 语句②

    }

    public void connectToHost() {

        String ip = hostInfo.getIp();

        int port = hostInfo.getPort();

        connectToHost(ip, port);

    }

    private void connectToHost(String ip, int port) {

        // ...

    }

    public static class HostInfo {

        private String ip;

        private int port;

        public HostInfo(String ip, int port) {

            this.ip = ip;

            this.port = port;

        }

        // ...

    }

}
````

当然，上述原子性问题都可以通过加锁解决。不过，Java语言规范特别地规定针对volatile修饰的long/double型变量进行的读、写操作也具有原子性。换而言之，volatile关键字能够保障long/double型变量访问操作的原子性。需要注意的是，volatile对原子性的保障仅限于共享变量写和读操作本身。对共享变量进行的赋值操作实际上往往是一个复合操作，volatile并不能保障这些赋值操作的原子性。例如，如下针对volatile变量counter1赋值语句：
````java
volatile counter1 = counter2 + 1;
````
如果counter2是一个局部变量，那么上述赋值语句实际上就是针对counter1的写操作，因此在volatile关键字的作用下上述赋值操作具有原子性。如果counter2也是一个共享变量，那么上述赋值语句就不具有原子性。这是由于此时上述语句实际上可以被分解为如下几个子操作（伪代码表示）：
````java
r1 = counter2; //子操作①：将共享变量counter2的值加载到寄存器r1
r1 = r1 + 1;//子操作②：将寄存器r1的值增加1
counter1 = r1;//子操作③：将寄存器r1的值写入共享变量counter1（内存）
````
volatile关键字并不像锁那样具有排他性，在写操作方面，其对原子性的保障也仅仅作用于上述的子操作③（变量写操作）。因此，一个线程在执行到子操作③的时候，其他线程可能已经更新了共享变量counter2的值，这就使得子操作③的执行线程实际上是向共享变量counter1写入了一个旧值。

因此，对volatile变量的赋值操作其表达式右边不能包含任何共享变量（包括被赋值的volatile变量本身）。

依照Java语言规范，对volatile修饰的long/double型变量的进行的读操作也具有原子性。因此，我们说volatile能够保障long/double型变量访问操作的原子性。

# 保障可见性
可见性（Visibility）是指一个线程（读线程）是否或者在什么情况下能够读取到其他线程（写线程）对共享变量所做的更新。由于软件、硬件的原因，一个线程（写线程）对共享变量进行更新之后，其他线程（读线程）再来读取该变量的时候，这些读线程可能无法读取到写线程对共享变量所做的更新，清单3展示了这点。

清单3 可见性问题Demo
````java
public class VisibilityDemo {

    public static void main(String[] args) throws InterruptedException {

        CountingThread backgroundThread = new CountingThread();

        backgroundThread.start();

        Thread.sleep(1000);

        backgroundThread.cancel();

        backgroundThread.join();

        System.out.printf("count:%s", backgroundThread.count);

    }

}

class CountingThread extends Thread {

    // 线程停止标志

    private boolean ready = false;

    public int count = 0;

    @Override

    public void run() {

        while (!ready) {

            count++;

        }

    }

    public void cancel() {

        ready = true;

    }

}
````
该Demo中，我们为子线程backgroundThread（类型为CountingThread）设置了一个停止标记ready。当ready值为true时，子线程通过使其run方法返回而实现线程的终止。然而，使用Java虚拟机的server模式运行上述Demo，我们可以发现该Demo中的子线程并没有像我们预期的那样在1秒钟之后终止而是一直在运行！由此可见，主线程（main线程）对共享变量ready所做的更新（将ready设置为true）并没有被子线程backgroundThread所读取到。究其原因，这是HotSpot虚拟机的C2编译器（Just In Time编译器）在将字节码动态编译为本地机器码的过程中执行循环不变量外提（ Loop-invariant code motion）优化的结果：由于该Demo中的共享变量ready并没有采用volatile修饰，因此C2编译器会认为该变量并不会被多个线程访问（实际上有多个线程访问该变量），于是C2编译器为了提升代码执行效率而将CountingThread.run()中的while循环语句优化为与如下伪代码等效的机器码：
````java
     if (!ready) {// 对变量ready的判断被提升到循环语句之外

            while (true) {

                count++;

            }

        }
````
这种优化可以通过查看C2编译器所产生的汇编代码来确认，如图1所示。不幸的是，这种优化导致了死循环！
图1 C2编译器循环不变量外提优化所产生的汇编代码
[![](http://idiotsky.top/images1/java-volatile-1.jpg)](http://idiotsky.top/images1/java-volatile-1.jpg)

如果我们采用volatile修饰上述Demo中的ready变量，那么C2编译器便会“意识”到ready是一个共享变量，因此就不会对CountingThread.run()中的while循环语句执行循环不变量外提优化从而避免了死循环。

当然，硬件的因素也可能导致可见性问题。处理器为了提高内存写操作的效率而引入的硬件部件写缓冲器（Store Buffer）和无效化队列（Invalidate Queue）都可能导致一个线程对共享变量所做的更新无法被后续线程所读取到。

Java语言规范规定，对于同一个volatile变量，一个线程（写线程）对该变量进行更新，其他线程（读线程）随后对该变量进行读取，这些线程总是可以读取到写线程对该变量所做的更新。换而言之，写线程更新一个volatile变量，读线程随后来读取该变量，那么这些读线程能够读取到写线程对该变量所做的更新这一点是有保障的（而不是碰运气！）。不过，由于volatile并不具有锁那样的排他性，因此volatile并不能够保障读线程所读取到变量值是共享变量的最新值：读线程在读取一个volatile变量的那一刻，其他线程（写线程）可能又恰好更新了该变量，因此读线程所读取到共享变量值仅仅是一个相对新值，即其他线程更新过的值（不一定是最新值）。

# 小结
以上我们介绍了volatile关键字对long/double型变量访问操作的原子性保障以及对可见性的保障。接下来我们将介绍volatile对有序性的保障，并通过介绍Java内存模型中的Happens-before关系这一概念来深入理解volatile对可见性和有序性的保障。

# 保障有序性
一个处理器上的线程所执行的一组操作在其他处理器上的线程看来可能是乱序的（Out-of-order），即这些线程对这组操作中的各个操作的感知顺序（观察到的顺序）与程序顺序（目标代码中指定的顺序）不一致。

下面我们看一个乱序实验，如清单4所示。

清单4 JIT编译器指令重排序Demo
````java
/**
 * 
 * 再现JIT指令重排序的Demo
 *
 * 
 * 
 * @author Viscent Huang
 * 
 */

@ConcurrencyTest(iterations = 200000)

public class JITReorderingDemo {

    private int externalData = 1;

    private Helper helper;

    @Actor

    public void createHelper() {

        helper = new Helper(externalData);

}

    @Observer({

            @Expect(desc = "Helper is null", expected = -1),

            @Expect(desc = "Helper is not null,but it is not initialized",

                    expected = 0),

            @Expect(desc = "Only 1 field of Helper instance was initialized",

                    expected = 1),

            @Expect(desc = "Only 2 fields of Helper instance were initialized",

                    expected = 2),

            @Expect(desc = "Only 3 fields of Helper instance were initialized",

                    expected = 3),

            @Expect(desc = "Helper instance was fully initialized", expected = 4) })

    public int consume() {

        int sum = 0;

        /*
         * 
         * 由于我们未对共享变量helper进行任何处理（比如采用volatile关键字修饰该变量），
         * 
         * 因此，这里可能存在可见性问题，即当前线程读取到的变量值可能为null。
         * 
         */

        final Helper observedHelper = helper;

        if (null == observedHelper) {

            sum = -1;

        } else {

            sum = observedHelper.payloadA + observedHelper.payloadB

                    + observedHelper.payloadC + observedHelper.payloadD;

        }

        return sum;

    }

    static class Helper {

        int payloadA;

        int payloadB;

        int payloadC;

        int payloadD;

        public Helper(int externalData) {

            this.payloadA = externalData;

            this.payloadB = externalData;

            this.payloadC = externalData;

            this.payloadD = externalData;

        }

    }

    public static void main(String[] args) throws InstantiationException,

            IllegalAccessException {

        // 调用测试工具运行测试代码

        TestRunner.runTest(JITReorderingDemo.class);

    }

}
````
清单4中的程序非常简单（读者可以忽略其中的注解，因为那是给测试工具用的）：createHelper方法会将实例变量helper更新为一个新创建的Helper实例；consume方法会读取helper所引用的Helper实例，并计算该实例的所有字段（payloadA~payloadD）的值之和作为其返回值。该程序的main方法调用测试工具TestRunner的runTest方法的作用是让测试工具安排一些线程并发地执行createHelper方法和consume方法，并统计consume方法多次执行的返回值。由于createHelper方法创建Helper实例的时候使用的构造器参数externalData值为1，因此这样看来consume方法的返回值似乎“理所当然”地应该是4。然而，事实却并不总是如此。使用如下命令以server模式并设置Java虚拟机参数“-XX:-UseCompressedOops”运行清单4所示的程序[1]：
````shell
java -server -XX:-UseCompressedOops JITReorderingD
````
我们可以看到类似如下的输出[2]：
````shell
expected:-1 occurrences:8 ==>Helper is null

expected:0 occurrences:2 ==>Helper is not null,but it is not initialized

expected:1 occurrences:0 ==>Only 1 field of Helper instance was initialized

expected:2 occurrences:1 ==>Only 2 fields of Helper instance were initialized

expected:3 occurrences:4 ==>Only 3 fields of Helper instance were initialized

expected:4 occurrences:199985 ==>Helper instance was fully initialized
````
上面的输出中，expected后面的数字表示consume方法的返回值，相应的occurrences表示出现相应返回值的次数。

不难看出这次程序运行时，有几次consume方法的返回值并不为4：有的为3（出现4次）、有的为2（出现1次），甚至还有的为0（出现2次）！这说明consume方法的执行线程有时候读取到了一个未初始化完毕（或者正在初始化）的Helper实例：Helper实例不为null，但是其部分实例字段的字段值仍然为其默认值而非Helper类的构造器中指定的初始值。下面我们分析其中的原因。

我们知道，createHelper方法中的唯一一条语句：
````java
helper = new Helper(externalData);
````
可以分解为以下几个子操作（伪代码表示）：
````java
objRef = allocate(Helper.class);//子操作①：分配Helper实例所需的内存空间，并获得一个指向该空间的引用

inovkeConstructor(objRef)；//子操作②：调用Helper类的构造器初始化objRef引用指向的Helper实例

helper = objRef；//子操作③：将Helper实例引用objRef赋值给实例变量helper
````
通过查看Java字节码不难发现createHelper方法中指定的程序顺序就是上述的先初始化Helper实例（子操作②）再将相应的实例的引用赋值给实例变量helper（子操作③）。然而，consume方法的执行线程却观察到了未初始化完毕的Helper实例，这说明该线程对createHelper方法所执行的操作的感知顺序与该方法所指定的程序顺序不一致，即产生了乱序。

查看上述程序运行过程中JIT编译器动态生成的汇编代码（相当于机器码），如图2所示，我们可以发现JIT编译器编译字节码的时候并不是每次都按照上述源代码顺序（这里同时也是程序顺序）生成相应的机器码（汇编代码）：JIT编译器将子操作③相应的指令重排到子操作②相应的指令之前，即JIT编译器在初始化Helper实例之前可能已经将对该实例的引用写入helper实例变量。这就导致了其他线程（consume方法的执行线程）看到helper实例变量（不为null）的时候，该实例变量所引用的对象可能还没有被初始化或者未初始化完毕（即相应构造器中的代码未执行结束）。这就解释了为什么我们在运行上述程序的时候，consume方法的返回值有时候并不是4。
图2 JIT编译器重排序Demo中的汇编代码片段
[![](http://idiotsky.top/images1/java-volatile-2.jpg)](http://idiotsky.top/images1/java-volatile-2.jpg)

虽然乱序有利于充分发挥处理器的指令执行效率，但是正如上述实验所展示的，它也可能导致程序正确性的问题。所以，为了保障程序的正确性，有时候我们需要确保线程对一组操作的感知顺序与这组操作的程序顺序保持一致，即保障这组操作的有序性。上述实验中，为了确保consume方法的执行线程看到的Helper实例总是初始化完毕的，我们需要确保createHelper方法所执行的操作的有序性。为此，我们只需要用volatile关键字来修饰实例变量helper即可，而无需借助锁。这里，volatile关键字所起的作用是通过禁止子操作②被JIT编译器以及处理器重排序（指令重排序、内存重排序）到子操作③之后，从而保障了有序性。

Java语言规范规定，对于访问（读、写）同一个volatile变量的多个线程而言，一个线程（写线程）在写volatile变量前所执行的内存读、写操作在随后读取该volatile变量的其他线程（读线程）看来是有序的。设X、Y是普通（非volatile）共享变量，其初始值均为0，V是volatile变量，其初始值为false，r1、r2是局部变量，线程T1和T2先后访问V，如图3所示。那么，T1对V的更新以及更新V前所执行的操作在T2看来是有序的：在T2看来T1对X、Y和V的写操作就像是完全依照程序顺序执行的。换而言之，如果T2读取到V的值为true，那么该线程所读取到的X和Y的值必然为分别为1和2。相反，如果V不是volatile变量，那么上述这种保证就不存在，即T2读取到V的值为true时，T2所读取到X和Y的值可能并非1和2。

图3 volatile关键字的有序性保障示例代码
[![](http://idiotsky.top/images1/java-volatile-3.jpg)](http://idiotsky.top/images1/java-volatile-3.jpg)

上述例子中，我们假设只有一个线程更新V（另外一个线程读取V），如果有更多的线程并发更新V，那么由于volatile并不具有排他性，因此在T2读取V的时候T1之外的其他线程可能已经又更新了共享变量X、Y，这就使得T2在其读取到V的值为true的情况下，其读取到X和Y的值可能不是1和2。不过，这种现象是数据竞争的结果，这与volatile能够保障有序性本身并不矛盾。

# Happens-before关系
了解Java内存模型（Java Memory Model）中的定义的Happens-before关系（Happens-before Relationship）这一概念有助于我们进一步理解volatile变量对可见性和有序性的保障。

Java内存模型定义了一些动作（Action）。这些动作包括变量的读/写、锁的申请（lock）与释放（unlock）以及线程的启动（Thread.start()调用）和加入（Thread.join()调用）等。如果动作A和动作B之间存在Happens-before关系，那么动作A的执行结果对动作B可见。反之，如果动作A和动作B之间不存在Happens-before关系，那么动作A的执行结果对B来说不一定是可见的。下文我们用“→”来表示Happens-before关系，例如“A→B”表示动作A与动作B存在Happens-before关系。

Java内存模型中的volatile变量规则（Volatile Variable Rule）规定，对一个volatile变量的写操作happens-before后续（Subsequent）每一个针对该变量的读操作。这里有两点需要注意：首先，针对同一个volatile变量的写、读操作之间才有happens-before关系，不同volatile变量之间的写、读操作并无happens-before关系；其次，针对同一个volatile变量的写、读操作必须具有时间上的先后关系，即一个线程先写另外一个线程再来读这样这两个动作之间才能够有happens-before关系。因此，对于图2可有wV→rV，即动作wV（写volatile变量V）的结果对rV（读volatile变量V）可见。

Java内存模型中程序顺序规则（Program Order Rule）规定同一个线程中的每一个动作都happens-before该线程中程序顺序上排在该动作之后的每一个动作。因此，对于图3可有如下的happens-before关系：

````
wX→wY （hb1）

wY→wV（hb2）

rV→rX （hb3）

rX→rY（hb4）
````
Happens-before关系具有传递性，即如果A→B，B→C，那么就有A→C。因此，由hb1和hb2可得出以下happens-before关系：
````
wX→wV（hb5）
````
再根据volatile变量规则，可有happens-before关系：
````
wV→rV（hb6）
````
进一步根据happens-before关系的传递性由hb5和hb6可得出以下happens-before关系：
````
wX→rV（hb7）
````
同样根据happens-before关系的传递性由hb7和hb3可得出以下happens-before关系：
````
wX→rX（hb8）
````
同理，我们也可以推断出以下happens-before关系：
````
wY→rY（hb9）
````
由此可见，线程T1对普通共享变量X和Y所做的更新对线程T2来说都是可见的。这种可见性是在volatile变量规则、程序顺序规则以及happens-before关系的传递性的共同作用下得以保障的。因此，我们说volatile关键字不仅仅保障写线程对volatile变量所做的更新的可见性（hb6），它还保障了写线程在写volatile变量前对其他非volatile变量所做的更新的可见性（hb8和hb9）。

理解了Happens-before关系这一概念之后，我们可以思考这样一个问题：volatile关键字对可见性和有序性的保障是否适用于数组呢？例如，对于volatile修饰的一个int数组vArr，线程A执行“vArr[0]=1;”，接着，线程B再来读取vArr的第1个元素，那么此时线程B所读取到元素值是否一定是“1”呢（这里我们假设只有线程A和线程B这两个线程访问vArr）？答案是“不一定”：此时线程A和线程B从volatile关键字的角度来看都只是读线程（读取volatile变量vArr），即这两个线程之间并不存在Happens-before关系，因此线程A对vArr第1个元素的更新对线程B来说不一定是可见的。这个例子中，要保障对数组元素的更新的可见性，我们可以使用java.util.concurrent.atomic.AtomicIntegerArray类。

# 小结
上面介绍了volatile对有序性的保障，并通过介绍Java内存模型中的Happens-before关系这一概念来进一步介绍volatile对可见性和有序性的保障。通过前面的介绍，我们知道volatile关键字的作用包括保障long/double型变量访问操作的原子性、保障可见性和保障有序性。接下来将介绍Java虚拟机对volatile关键字的实现，volatile关键字的开销以及volatile的典型应用场景。

# Java虚拟机对volatile的实现
本节会涉及较多的术语，如表1所示。

表1 本节术语
[![](http://idiotsky.top/images1/java-volatile-4.jpg)](http://idiotsky.top/images1/java-volatile-4.jpg)

Java虚拟机对long/double型变量访问操作的原子性保障是通过使用原子指令（本身就具有原子性的处理器指令）实现的。下面通过一个实验来进一步介绍这点，该实验所需的Java代码如清单6所示。

清单6 Java虚拟机对volatile语义的实现实验Java代码
````java
public class AtomicJVMImpl {

    static long normalLong = 0L;

    static volatile long volatileLong = 0L;

    public static void main(String[] args) {

        long v1 = 0, v2 = 0;

        for (int i = 0; i < 100100; i++) {

            normalWrite(i);

            volatileWrite(i);

            v1 = normalRead() + i;

            v2 = volatileRead();

        }

        System.out.println(v1 + "," + v2);

    }

    public static void normalWrite(long value) {

        normalLong = value;

    }

    public static void volatileWrite(long value) {

        volatileLong = value;

    }

    public static long volatileRead() {

        return volatileLong;

    }

    public static long normalRead() {

        return normalLong;

    }

}
````
32位Java虚拟机（JIT编译器）执行（动态编译）normalWrite方法中的普通long/double型变量写操作时使用的机器码（x86汇编语言表示）如图4所示。
图4 32位Java虚拟机x86处理器下对普通long/double型变量写操作的实现
[![](http://idiotsky.top/images1/java-volatile-5.jpg)](http://idiotsky.top/images1/java-volatile-5.jpg)

可见，32位Java虚拟机在x86处理器平台下对普通long/double型变量（这里是long型变量）的写操作是通过两个子操作——先写低32位再写高32位实现的。32位Java虚拟机在某些处理器平台下可能仍然使用一条指令（比如在ARM处理器平台下使用strd指令）来实现普通long/double型变量的写操作，但是这条指令可能不是原子指令，因此在Java语言这一层次来观察，此时的普通long/double型变量写操作同样也不是原子操作。32位Java虚拟机（JIT编译器）在x86处理器平台下实现volatileWrite方法中的volatile long/double型变量写操作时使用的是一个原子指令（vmovsd），如图5所示。

图5 32位Java虚拟机x86处理器下对volatile long/double型变量写操作的实现
[![](http://idiotsky.top/images1/java-volatile-6.jpg)](http://idiotsky.top/images1/java-volatile-6.jpg)

类似的，Java虚拟机对long/double型变量读操作的原子性保障也是通过使用原子指令实现的。例如，32位Java虚拟机在x86平台下会使用vmovsd这个原子指令来实现volatile修饰的long/double型变量的读操作，而对普通long/double型变量的读操作则是使用2条mov指令实现。

Java虚拟机对可见性和有序性的保障则是通过使用内存屏障实现的。

处理器在其执行内存写操作的时候，往往是先将数据写入其写缓冲器中，而不是直接写入高速缓存。由于一个处理器上的写缓冲器中的内容无法被其他处理器所读取，因此写线程必须确保其对volatile变量所做的更新以及其更新volatile变量前对其他共享变量所做的更新（以下统称为对共享变量所做的更新）到达该处理器的高速缓存（而不是仍然停留在写缓冲器中）。这样，写线程的这些更新通过缓存一致性协议被其他处理器上的线程所读取才成为可能。为此，Java虚拟机（JIT编译器）会在volatile变量写操作之后插入一个StoreLoad内存屏障。这个内存屏障的其中一个作用就是将其执行处理器的写缓冲器中的当前内容写入高速缓存。

由于无效化队列的存在，处理器从其高速缓存中读取到的共享变量值可能是过时的。因此，为了确保读线程能够读取到写线程对共享变量所做的更新（包括volatile变量），读线程的执行处理器必须在读取volatile变量前确保无效化队列中内容被应用到该处理器的高速缓存中，即根据无效化队列中的内容将该处理器中相应的缓存行设置为无效，从而使写线程对共享变量所做的更新能够被反映到该处理器的高速缓存上。为此，Java虚拟机（JIT编译器）会在volatile变量读操作前插入一个LoadLoad内存屏障。有的处理器（例如x86处理器和ARM处理器）并没有引入无效化队列，因此在这些处理器上上述LoadLoad内存屏障就不再被需要。

可见，volatile关键字对可见性的保障是通过Java虚拟机（JIT编译器）在写线程和读线程中配对地使用内存屏障实现的，如图6所示。

图6 Java虚拟机（JIT编译器）为实现volatile语义而插入的内存屏障
[![](http://idiotsky.top/images1/java-volatile-7.jpg)](http://idiotsky.top/images1/java-volatile-7.jpg)

volatile关键字对有序性的保障也是通过Java虚拟机（JIT编译器）在写线程和读线程中配对地使用内存屏障实现的。为了使写线程对共享变量所做的更新在读线程看来是有序的（即感知顺序与程序顺序保持一致），Java虚拟机首先必须保证写线程程序顺序上排在写volatile变量之前的对其他共享变量的更新先于对volatile变量的更新反映到该线程所在的处理器的高速缓存上。换而言之，Java虚拟机必须确保程序顺序上排在volatile变量写操作之前的其他写操作不能够被编译器/处理器通过指令重排序和（或）内存重排序被重排序到该volatile变量写操作之后。为此，Java虚拟机（JIT编译器）会在volatile变量写操作之前插入LoadStore+StoreStore内存屏障，这个组合内存屏障禁止了volatile变量写操作与该操作之前的任何读、写操作之间的重排序（包括指令重排序和内存重排序）。其次，Java虚拟机（JIT编译器）必须确保读线程在读取完写线程对volatile变量所做的更新之后才开始读取写线程在更新该volatile变量前对其他共享变量所做的更新。换而言之，Java虚拟机必须确保程序顺序上排在volatile变量读操作之后的其他共享变量的读、写操作不能够被编译器/处理器通过指令重排序和（或）内存重排序被重排序到该volatile变量读操作之前。为此，Java虚拟机（JIT编译器）会在volatile变量读操作之后插入一个LoadLoad+LoadStore内存屏障，这个组合内存屏障禁止了volatile变量读操作与该操作之后的任何读、写操作之间的重排序（包括指令重排序和内存重排序）。可见，Java虚拟机是通过使写线程和读线程配对地使用内存屏障来实现volatile对有序性的保障的，如图6所示。

图7和图8展示了Java虚拟机（32位）在ARM处理器平台下对清单1中的volatileWrite、volatileRead方法进行JIT编译时插入的内存屏障情况。

图7 Java虚拟机在ARM处理器平台下在volatile变量写操作前后插入的内存屏障
[![](http://idiotsky.top/images1/java-volatile-8.jpg)](http://idiotsky.top/images1/java-volatile-8.jpg)
图7中，JIT编译器在volatile写操作（“vstr d7, [r5, #96]”指令）前插入的“dmb sy”指令相当于LoadStore+StoreStore内存屏障。JIT编译器在volatile写操作后插入的“dsb sy”指令相当于StoreLoad内存屏障。

图8 Java虚拟机在ARM处理器平台下在volatile变量读操作后插入的内存屏障
[![](http://idiotsky.top/images1/java-volatile-9.jpg)](http://idiotsky.top/images1/java-volatile-9.jpg)

图8中，JIT编译器在volatile读操作（“vldr d7, [r5, #96]”指令）后插入的“dmb sy”指令相当于LoadStore+LoadLoad内存屏障。由于ARM处理器并没有使用无效化队列，因此JIT编译器在volatile读操作前并不需要插入LoadLoad内存屏障。

# volatile的开销
上一节我们讲到Java虚拟机（JIT编译器）会在volatile变量写操作之后插入一个StoreLoad内存屏障。StoreLoad内存屏障是一个全能型内存屏障，它是内存屏障中功能最强大开销也最大的一个内存屏障。该内存屏障除了能够将写缓冲器中条目写入高速缓存之外，还能够将无效化队列中的内容应用到高速缓存中。而这两个操作的开销较大，在某些处理器上（例如ARM处理器）该内存屏障可能还会导致处理器流水线（Pipeline）停顿。由于Java虚拟机并不需要在普通变量写操作之后插入内存屏障，而临界区中的写操作除了有内存屏障的开销之外，还有锁的申请与释放的开销，因此volatile变量写操作的开销介于普通变量写操作和临界区中的写操作之间。

如果处理器引入了无效化队列，那么Java虚拟机需要在volatile变量读操作前插入一个LoadLoad内存屏障。另外，Java虚拟机（JIT编译器）在volatile变量读操作之后插入的LoadLoad+LoadStore内存屏障会阻止处理器执行某些优化（比如重排序和预先加载数据）。而临界区中的读操作不仅仅有内存屏障的开销，还有锁的申请与释放的开销。因此，volatile变量读操作的开销介于普通变量读操作和临界区中的读操作之间。

普通共享变量的值可能会被JIT编译器缓存到寄存器中，即对于任意一个线程，该线程第一次读取某个普通共享变量是一次内存读操作（比如x86处理器上的mov指令），随后重复读取这个共享变量则是从寄存器中读取。根据volatile关键字的语义，volatile变量是不能够被缓存到寄存器中，即每个volatile变量读操作都是一次内存读操作，同一个线程即使是连续多次读取同一个volatile变量，这当中的每次读取操作都是从内存中读取的。因此，从整体上看，volatile变量的读取开销要比普通共享变量的开销要大。

# volatile的典型应用场景
## 间接保障复合操作的原子性与可见性
对于清单2中的可见性和原子性问题，虽然我们可以通过对updateHostInfo方法和connectToHost方法进行加锁来加以解决，但是借助volatile关键字我们既可以保障可见性和原子性又可以避免锁的开销，如清单7所示。

清单7 使用volatile间接保障复合操作的原子性与可见性实例
````java
public class AtomicityExample1 {

    private volatile HostInfo hostInfo;

    public void updateHostInfo(String ip, int port) {

        HostInfo newHostInfo = new HostInfo(ip, port);

        this.hostInfo = newHostInfo;

    }

    public void connectToHost() {

        String ip = hostInfo.getIp();

        int port = hostInfo.getPort();

        connectToHost(ip, port);

    }

    private void connectToHost(String ip, int port) {

        // ...

}

    public static class HostInfo {

        private String ip;

        private int port;

        public HostInfo(String ip, int port) {

            this.ip = ip;

            this.port = port;

        }

        public String getIp() {

            return ip;

        }

        public int getPort() {

            return port;

        }

        // ...

    }

}
````
清单7中的updateHostInfo方法运用了不可变对象（Immutable Object）模式：它在更新主机IP地址和端口号的时候并不调用HostInfo类的相应set方法，而是先创建新的HostInfo实例，再将该实例（的引用）赋值给实例变量hostInfo，由此实现了主机信息的更新。由于这个赋值操作本身就是一个原子操作，因此我们只需要再使这个赋值操作的结果对其他线程可见即可保障线程安全。为此，我们只需要将实例变量hostInfo声明为volatile变量即可。

## 保障对象的安全发布
volatile的一个典型应用就是用于正确地实现基于双重检查锁定（Double checked locking）法的单例类（Singleton），如清单3所示。用双重检查锁定法来实现单例类的目的在于既能够实现延迟加载（Lazy Load，以减少不必要的开销）又能够尽量减少锁的开销。清单8中，采用volatile来修饰静态变量instance目的有两个：保障可见性和保障有序性。尽管对instance变量的赋值是在一个临界区中进行的，但是第1次检查的if语句并没有处于临界区之中。也就是说，语句③对instance的写操作与语句①对instance的读操作这两个操作之间并不存在happens-before关系，因此，语句③的执行结果对语句①来说不一定是可见的。为了确保语句③对instance的写操作的结果对语句①（第1次检查）可见，我们只需要采用volatile来修饰instance即可。从有序性的角度来看，在没有采用volatile修饰instance的情况下，语句①的执行线程即使读取到instance不为null（其他线程执行语句③的结果），那么由于重排序（JIT重排序和/活内存重排序）的作用，instance所引用的对象仍然可能是未初始化完毕的，这就可能导致程序的正确性问题。采用volatile修饰instance之后，在volatile保障有序性的作用下，语句①的执行线程一旦看到instance不为null，那么instance所引用的对象必然是初始化完毕的。此时，我们称instance所引用的对象被安全地发布。

清单8 使用volatile正确实现基于双重检查锁定法的单例类
````java
public class DCLSingleton {

    private static volatile DCLSingleton instance;

    // 省略其他字段

    // 私有构造器

    private DCLSingleton() {

}

    public DCLSingleton getInstance() {

        if (null == instance) {// 语句①： 第1次检查，不加锁

            synchronized (DCLSingleton.class) {

                if (null == instance) {// 语句②： 第2次检查，加锁

                    instance = new DCLSingleton();// 语句③：实例化

                }

            }

        }

        return instance;

    }

    // 省略其他public方法

}
````

# 总结
volatile关键字的作用包括保障long/double型变量访问操作的原子性、保障可见性以及保障有序性。Java虚拟机在实现volatile关键字的语义时通常会借助一些特殊的处理器指令（原子指令和内存屏障）。volatile变量访问的开销介于普通变量访问和在临界区中进行的变量访问之间。volatile的典型运用场景包括间接保障复合操作的原子性、保障对象的安全发布等。

from https://zhuanlan.zhihu.com/p/30295283
