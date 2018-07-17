---
title: linux的写时复制
date: 2018-03-13 23:32:23
tags: [linux,linux内核]
categories: linux
---
> 前面一篇文章又给自己挖坑，所以必须再mark一篇，这篇文章很好的说了关于写时复制的原理。

当调用fork()系统调用创建一个子进程时，Linux并不会为子进程创建新的物理内存空间，而是公用父进程的物理内存。这是因为Linux的内核开发者觉得，调用者调用fork()系统调用后会立刻调用exec()系统调用执行新的程序，这样旧的物理内存内容就没有什么作用了（因为新的程序与旧的程序完全没有关联），所以为子进程复制父进程的物理内存内容是一件徒劳无功的事情。

所以Linux的做法就是：父子进程共用同一物理内存。如下图：

[![](http://idiotsky.top/images2/linux-copy-on-write-1.png)](http://idiotsky.top/images2/linux-copy-on-write-1.png) 
<!-- more -->

但操作系统的要求是：进程之间的内存应该要独立，就是读写A进程的内存空间不应该影响B进程的内存内容。读操作是不会改变内存中的内容，所以对于读操作来说，共享物理内存是安全的。但是对于写操作就不一样，如果父子进程共用了相同的物理内存，那么对子进程的内存进行写操作同时会影响到父进程，所以违反了操作系统的要求。

Linux的解决方案是：把共用的物理内存设置为只读，因为读操作不会改变内存的内容，所以对于父子进程都是允许的。而当父子进程其中一个进行写操作时，因为内存被设置为只读，所以CPU会触发 “page fault” 的错误，从而调用内核的`do_page_fault()`函数。而`do_page_fault()`函数又会调用`do_wp_page()`函数去进行复制父进程内存的内容。

`do_wp_page()`函数先进行一些安全监测，然后调用`__do_wp_page()`函数做最后的复制操作。去掉一些监测后，`__do_wp_page()`函数的代码如下图：

[![](http://idiotsky.top/images2/linux-copy-on-write-3.jpg)](http://idiotsky.top/images2/linux-copy-on-write-3.jpg) 

`__do_wp_page()`首先会申请一块新的物理内存，然后复制旧的物理内存页的内容到新的物理内存也中，然后设置虚拟内存与物理内存的映射关系。最后把父子进程的物理内存设置可读写，这样父子进程相同的虚拟内存都指向不同的物理内存，所以达到进程之间内存隔离的目的。如下图：

[![](http://idiotsky.top/images2/linux-copy-on-write-2.png)](http://idiotsky.top/images2/linux-copy-on-write-2.png) 