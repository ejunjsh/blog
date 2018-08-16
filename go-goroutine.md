---
title: 深入go之goroutine
date: 2018-08-12 19:56:31
tags: [go,goroutine,协程,并发,并行,多线程]
categories: go
---
> goroutine是go的核心，没有goroutine，go就没什么意思了👿。goroutine离不开协程，线程和并发，所以下面会说说相关的内容。

# 协程

协程(coroutine)其实就是一个函数，方法或者例程（routine）。一般情况下函数都是在用户线程下面执行的，线程的调度由内核触发，所以函数在执行过程中，用户线程没办法控制函数的执行调度，只能任由内核主宰。协程就不同，它可以由用户线程控制调度，在任何时候调度协程的执行。函数在执行时，内核调度会陷入内核并保存当前线程的栈和上下文，然后恢复之前被停止线程继续执行，代价比较高。而协程的调度，不用陷入内核，用户线程只是保存当前协程的栈和上下文，恢复之前的被停止协程继续执行。

还有种说法是说函数是协程的一种特例。因为函数只有在return语句才会返回，而协程可以在任何时刻返回。

协程很早就提出来了，可是在现在才火起来吧，大概由于某种语言（lua）的广泛使用吧。而go更是把协程用到底，基本可以理解go的所有代码都跑在协程下，并用goroutine来代表它自己的协程。
<!-- more -->

# 协程 vs 线程

线程是处理器调度的基本单位，在CPU切分时间片的前提下，操作系统进行抢占式调度。

协程也可以理解为一种更小的调度基本单位。它由运行在用户线程的调度器来调度。

名字|调度|内存消耗|切换代价
----|-----|----|----
线程|内核进行调度|较大（1MB~8MB）|陷入内核，各种寄存器的保存和刷新
协程|用户线程调度|较小（2KB~5KB）|各种寄存器的保存和刷新

从上表可以发现，线程比协程更加耗费内存，而且还会造成陷入内核。但是协程的切换完全交给用户线程来调度，这个增加了实现的难度。还有就是协程的调度是由单个线程调度，如果处理器是多核的话，没办法充分利用。很庆幸的是，go已经实现了它自己的协程调度逻辑，并且充分利用多线程来调度goroutine。

# 要协程何用？

协程能火也是有各种理由的。

* 高并发处理。在用户空间切换上下文，不用陷入内核来做线程切换，避免不必要用户空间和内核空间的数据拷贝。
* 用同步的方式去写异步代码，高效率且不容易出错 (nodejs里面的asyn/await，就是这种)
* 非抢占式模型，能控制中断位置，不会发生由于强行切换线程导致的资源竞争。(极端情况下还是会执行抢占，防止协程长时间占用CPU，但这不是标准抢占式模型）

# 并发 VS 并行

先上图：

[![](http://idiotsky.top/images2/go-goroutine.jpg)](http://idiotsky.top/images2/go-goroutine.jpg)

* 并发：处理器被划分为一个个时间分片，多个线程在处理器中交替执行，同一个时刻，只有一个线程被执行（通用地来说，支持并发是一种系统拥有交替执行多个任务的能力的表现）
* 并行：多个线程，在多个处理器上同时执行。

> 举个最简单的例子，医院诊室看病。把病人当做线程，医生当做处理器。

> 并发：只有一个医生，病人A看了一会儿，医生让他下楼拍X光，然后病人B进来看诊，之后医生让B去做彩超，然后A此时回来了，医生继续给A看病。（任意瞬间，医生只在给其中一个人看病）

> 并行： 有3个医生，3个病人，一个病人对应一个医生，同时问诊。

如果并发交替的速度够快，就能达到“逻辑并行”的效果，对外看起来就和并行一样。

并发执行多线程并不能真的充分利用CPU，达到减少单个线程执行时间的效果，这种交替挂起执行的方式却能够给用户带来每个线程都在”同时执行“的感觉，从而增强了服务的响应速度。就像上面例子中的病人B不用一直排队等待 A拍完X光并且医生确定A的病看完了 才能去看病。

# 调度

goroutine的调度可以理解为多线程调度协程（goroutine）。所以这里调度会有三个角色：线程，调度器，协程。它们分别用M,P,G来表示吧。

[![](http://idiotsky.top/images2/go-goroutine-1.jpg)](http://idiotsky.top/images2/go-goroutine-1.jpg)

* G: 表示goroutine，存储了goroutine的执行stack信息、goroutine状态以及goroutine的任务函数等；另外G对象是可以重用的。
* P: 表示逻辑processor，P的数量决定了系统内最大可并行的G的数量（前提：系统的物理cpu核数>=P的数量，GOMAXPROCS环境变量代表的个数是P的个数，推荐值为CPU的核心数）；P的最大作用还是其拥有的各种G对象队列、链表、一些cache和状态。
* M: M代表着真正的执行计算资源。在绑定有效的p后，进入schedule循环；而schedule循环的机制大致是从各种队列、p的本地队列中获取G，切换到G的执行栈上并执行G的函数，调用goexit做清理工作并回到m，如此反复。M并不保留G状态，这是G可以跨M调度的基础。

__下面是G、P、M定义的代码片段：__
````go
//src/runtime/runtime2.go
type g struct {
        stack      stack   // offset known to runtime/cgo
        sched     gobuf
        goid        int64
        gopc       uintptr // pc of go statement that created this goroutine
        startpc    uintptr // pc of goroutine function
        ... ...
}

type p struct {
    lock mutex

    id          int32
    status      uint32 // one of pidle/prunning/...

    mcache      *mcache
    racectx     uintptr

    // Queue of runnable goroutines. Accessed without lock.
    runqhead uint32
    runqtail uint32
    runq     [256]guintptr

    runnext guintptr

    // Available G's (status == Gdead)
    gfree    *g
    gfreecnt int32

  ... ...
}

type m struct {
    g0      *g     // goroutine with scheduling stack
    mstartfn      func()
    curg          *g       // current running goroutine
 .... ..
}
````

[![](http://idiotsky.top/images2/go-goroutine-2.jpg)](http://idiotsky.top/images2/go-goroutine-2.jpg)

上图是2个M（线程），每个线程对应一个处理器（P），M是必须关联P才能执行协程（G）的。图中蓝G代表的是运行中的goroutine，灰G表示的待执行的Goroutine，待执行的Goroutine存储在 P 中的一个局部队列中，此时P执行Goroutine会这个队列中取，不用加锁，提高了并发度。（Go1.0版本中，调度器取Goroutine是去一个全局队列中取，需要加锁，线程会经常阻塞等待锁）

下面更加清晰说明整个结构

[![](http://idiotsky.top/images2/go-goroutine-5.png)](http://idiotsky.top/images2/go-goroutine-5.png)

__如果其中一个G执行的时候，发生了系统调用，阻塞了怎么办？__

[![](http://idiotsky.top/images2/go-goroutine-3.jpg)](http://idiotsky.top/images2/go-goroutine-3.jpg)

上图左边，G0中陷入系统调用，导致M0阻塞。

此时，M0放弃了它的P，让M1去处理P中剩下的Goroutine。这里的M1可能是在线程缓存中取的，或者运行中生成的。

当M0从系统调用中恢复，它会去别的M中找P来执行G0（比如说别的M阻塞丢出了P），如果没有P，那么它会把G0放到全局队列中，并且把它自己放到线程缓存中。

全局队列保存了Goroutine，当各自P中的局部队列没有Goroutine时，P会到全局队列中取Goroutine。并且即使P中局部队列有Goroutine，也会周期性地从全局队列中取Goroutine，保持全局队列中的Goroutine能够尽快被执行。

处理系统调用，也是go程序为什么跑在多线程上的一个原因，即使GOMAXPROCS是1，也可能会有多个工作线程。

__当P局部队列不均衡时怎么处理？如果有多个P，其中一个P的局部队列Goroutine执行完了。__

[![](http://idiotsky.top/images2/go-goroutine-4.jpg)](http://idiotsky.top/images2/go-goroutine-4.jpg)

如果一个P局部队列为空，那么它尝试从全局队列中取Goroutine，如全局队列为空，则会随机从其它P的局部队列中“挪”一半Goroutine到自己的队列当中， 以保证所有的M都是有任务执行的，间接做到负载均衡（可以参考go源码的findrunnable()函数 ）

__channel阻塞怎么办__

如果G被阻塞在某个channel操作上时，G会被放置到某个wait队列中，而M会尝试运行下一个runnable的G；如果此时没有runnable的G供m运行，那么m将解绑P，并进入sleep状态。当channel操作完成，在wait队列中的G会被唤醒，标记为runnable，放入到某P的队列中，绑定一个M继续执行。

__网络I/O和文件I/O怎么办__

Go runtime已经实现了[netpoller](http://morsmachine.dk/netpoller)，这使得即便G发起网络I/O操作也不会导致M被阻塞（仅阻塞G），从而不会导致大量M被创建出来。但是对于regular file的I/O操作一旦阻塞，那么M将进入sleep状态，等待I/O返回后被唤醒；这种情况下P将与sleep的M分离，再选择一个idle的M。如果此时没有idle的M，则会新创建一个M，这就是为何大量I/O操作导致大量Thread被创建的原因。

__遇到锁怎么办__

go的锁不是系统级别的MUTEX锁，而是轻量级的CAS锁，所以抢锁失败不会阻塞M，而是阻塞G(阻塞之前还自旋一下看看能不能再抢到锁)，然后这个G就会被正常的调度出去，在某个时刻又调度回来继续抢锁。

__如果一个P连续执行长时间，没有切换G，怎么处理？__

和操作系统按时间片调度线程不同，Go并没有时间片的概念。如果某个G没有进行system call调用、没有进行I/O操作、没有阻塞在一个channel操作上，那么m是如何让G停下来并调度下一个runnable G的呢？答案是：__G是被抢占调度的__。

除非极端的无限循环或死循环，否则只要G调用函数，Go runtime就有抢占G的机会。Go程序启动时，runtime会去启动一个名为sysmon的m(一般称为监控线程)，该m无需绑定p即可运行，该m在整个Go程序的运行过程中至关重要：

````go
//$GOROOT/src/runtime/proc.go

// The main goroutine.
func main() {
     ... ...
    systemstack(func() {
        newm(sysmon, nil)
    })
    .... ...
}

// Always runs without a P, so write barriers are not allowed.
//
//go:nowritebarrierrec
func sysmon() {
    // If a heap span goes unused for 5 minutes after a garbage collection,
    // we hand it back to the operating system.
    scavengelimit := int64(5 * 60 * 1e9)
    ... ...

    if  .... {
        ... ...
        // retake P's blocked in syscalls
        // and preempt long running G's
        if retake(now) != 0 {
            idle = 0
        } else {
            idle++
        }
       ... ...
    }
}

````

sysmon每20us~10ms启动一次，按照《Go语言学习笔记》中的总结，sysmon主要完成如下工作：

* 释放闲置超过5分钟的span物理内存；
* 如果超过2分钟没有垃圾回收，强制执行；
* 将长时间未处理的netpoll结果添加到任务队列；
* 向长时间运行的G任务发出抢占调度；
* 收回因syscall长时间阻塞的P；

我们看到sysmon将“向长时间运行的G任务发出抢占调度”，这个事情由retake实施：

````go
// forcePreemptNS is the time slice given to a G before it is
// preempted.
const forcePreemptNS = 10 * 1000 * 1000 // 10ms

func retake(now int64) uint32 {
          ... ...
           // Preempt G if it's running for too long.
            t := int64(_p_.schedtick)
            if int64(pd.schedtick) != t {
                pd.schedtick = uint32(t)
                pd.schedwhen = now
                continue
            }
            if pd.schedwhen+forcePreemptNS > now {
                continue
            }
            preemptone(_p_)
         ... ...
}
````

可以看出，如果一个G任务运行10ms，sysmon就会认为其运行时间太久而发出抢占式调度的请求。一旦G的抢占标志位被设为true，那么待这个G下一次调用函数或方法时，runtime便可以将G抢占，并移出运行状态，放入P中局部队列中，等待下一次被调度。


# 参考

https://zhuanlan.zhihu.com/p/32497435

https://tonybai.com/2017/06/23/an-intro-about-goroutine-scheduler/
