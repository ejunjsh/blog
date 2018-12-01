---
title: 深入Linux proc文件系统
date: 2018-11-28 22:38:53
tags: [linux,proc]
categories: linux
---
> 继续深入学习Linux👿

在Linux上，proc是一个伪文件系统，提供了访问内核数据的方法，一般挂载在“/proc”目录，其中的大部分内容是只读的，挂载（mount）信息可能为：

````
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
````

下面列举“/proc”文件系统下的文件和目录。

# pid目录

`/proc/[pid]`目录，pid为进程的数字ID，是个数值，每个运行着的进程都有这么一个目录。

## cmdline

`/proc/[pid]/cmdline`是一个只读文件，包含进程的完整命令行信息。如果这个进程是zombie进程，则这个文件没有任何内容。举例如下：

````
# ps -ef | grep 2948
root       2948      1  0 Nov05 ?        00:00:04 /usr/sbin/libvirtd --listen

# cat /proc/2948/cmdline
/usr/sbin/libvirtd--listen
````

<!-- more -->

## comm

`/proc/[pid]/comm`包含进程的命令名。举例如下：

````
# cat /proc/2948/comm

libvirtd
````

## cwd

`/proc/[pid]/cwd`是进程当前工作目录的符号链接。举例如下：

````
# ls -lt /proc/2948/cwd
lrwxrwxrwx 1 root root 0 Nov  9 12:14 /proc/2948/cwd -> /
````

## environ

`/proc/[pid]/environ`显示进程的环境变量。举例如下：

````
# strings /proc/2948/environ
LANG=POSIX
LC_CTYPE=en_US.UTF-8
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NOTIFY_SOCKET=@/org/freedesktop/systemd1/notify
LIBVIRTD_CONFIG=/etc/libvirt/libvirtd.conf
LIBVIRTD_ARGS=--listen
LIBVIRTD_NOFILES_LIMIT=2048
````

## exe

`/proc/[pid]/exe`为实际运行程序的符号链接。举例如下：

````
# ls -lt /proc/2948/exe
lrwxrwxrwx 1 root root 0 Nov  5 13:04 /proc/2948/exe -> /usr/sbin/libvirtd
````

## fd

`/proc/[pid]/fd`是一个目录，包含进程打开文件的情况。举例如下：

````
# ls -lt /proc/3801/fd
total 0
lrwx------. 1 root root 64 Apr 18 16:51 0 -> socket:[37445]
lrwx------. 1 root root 64 Apr 18 16:51 1 -> socket:[37446]
lrwx------. 1 root root 64 Apr 18 16:51 10 -> socket:[31729]
lrwx------. 1 root root 64 Apr 18 16:51 11 -> socket:[34562]
lrwx------. 1 root root 64 Apr 18 16:51 12 -> socket:[39978]
lrwx------. 1 root root 64 Apr 18 16:51 13 -> socket:[34574]
lrwx------. 1 root root 64 Apr 18 16:51 14 -> socket:[39137]
lrwx------. 1 root root 64 Apr 18 16:51 15 -> socket:[39208]
lrwx------. 1 root root 64 Apr 18 16:51 16 -> socket:[39221]
lrwx------. 1 root root 64 Apr 18 16:51 17 -> socket:[41080]
lrwx------. 1 root root 64 Apr 18 16:51 18 -> socket:[40014]
lrwx------. 1 root root 64 Apr 18 16:51 19 -> socket:[34617]
lrwx------. 1 root root 64 Apr 18 16:51 20 -> socket:[34620]
lrwx------. 1 root root 64 Apr 18 16:51 23 -> socket:[42357]
lr-x------. 1 root root 64 Apr 18 16:51 3 -> /dev/urandom
lrwx------. 1 root root 64 Apr 18 16:51 4 -> socket:[37468]
lrwx------. 1 root root 64 Apr 18 16:51 5 -> socket:[37471]
lrwx------. 1 root root 64 Apr 18 16:51 6 -> socket:[289532]
lrwx------. 1 root root 64 Apr 18 16:51 7 -> socket:[31728]
lrwx------. 1 root root 64 Apr 18 16:51 8 -> socket:[37450]
lrwx------. 1 root root 64 Apr 18 16:51 9 -> socket:[37451]
l-wx------. 1 root root 64 Apr 13 16:35 2 -> /root/.vnc/localhost.localdomain:1.log
````

目录中的每一项都是一个符号链接，指向打开的文件，数字则代表文件描述符。

## limits

`/proc/[pid]/limits`显示当前进程的资源限制。举例如下：

````
# cat /proc/2948/limits
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        0                    unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             6409                 6409                 processes
Max open files            1024                 4096                 files
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
Max file locks            unlimited            unlimited            locks
Max pending signals       6409                 6409                 signals
Max msgqueue size         819200               819200               bytes
Max nice priority         0                    0
Max realtime priority     0                    0
Max realtime timeout      unlimited            unlimited            us
````

Soft Limit表示kernel设置给资源的值，Hard Limit表示Soft Limit的上限，而Units则为计量单元。

## maps

`/proc/[pid]/maps`显示进程的内存区域映射信息。举例如下：

````
address           perms offset  dev   inode        pathname
00400000-00452000 r-xp 00000000 08:02 173521      /usr/bin/dbus-daemon
00651000-00652000 r--p 00051000 08:02 173521      /usr/bin/dbus-daemon
00652000-00655000 rw-p 00052000 08:02 173521      /usr/bin/dbus-daemon
00e03000-00e24000 rw-p 00000000 00:00 0           [heap]
00e24000-011f7000 rw-p 00000000 00:00 0           [heap]
...
35b1800000-35b1820000 r-xp 00000000 08:02 135522  /usr/lib64/ld-2.15.so
35b1a1f000-35b1a20000 r--p 0001f000 08:02 135522  /usr/lib64/ld-2.15.so
35b1a20000-35b1a21000 rw-p 00020000 08:02 135522  /usr/lib64/ld-2.15.so
35b1a21000-35b1a22000 rw-p 00000000 00:00 0
35b1c00000-35b1dac000 r-xp 00000000 08:02 135870  /usr/lib64/libc-2.15.so
35b1dac000-35b1fac000 ---p 001ac000 08:02 135870  /usr/lib64/libc-2.15.so
35b1fac000-35b1fb0000 r--p 001ac000 08:02 135870  /usr/lib64/libc-2.15.so
35b1fb0000-35b1fb2000 rw-p 001b0000 08:02 135870  /usr/lib64/libc-2.15.so
...
f2c6ff8c000-7f2c7078c000 rw-p 00000000 00:00 0    [stack:986]
...
7fffb2c0d000-7fffb2c2e000 rw-p 00000000 00:00 0   [stack]
7fffb2d48000-7fffb2d49000 r-xp 00000000 00:00 0   [vdso]
````

* address字段表示进程中内存映射占据的地址空间，格式为十六进制的BeginAddress-EndAddress。 
* perms字段表示权限，共四个字符，依次为rwxs或rwxp，其中r为read，w为write，x为execute，s为shared，p为private，对应位置没有权限时用一个短横线代替。 
* offset字段表示内存映射地址在文件中的字节偏移量。 
* dev字段表示device，格式为major:minor。 
* inode字段表示对应device的inode，0表示内存映射区域没有关联的inode，如未初始化的BSS数据段就是这种情况。 
* pathname字段用于内存映射的文件，对于ELF格式的文件来说，可以通过命令readelf -l查看ELF程序头部的Offset字段，与maps文件的offset字段作对比。pathname可能为空，表示匿名映射，这种情况下难以调试进程，如gdb、strace等命令。除了正常的文件路径之外，pathname还可能是下面的值：

````
[stack]    初始进程（主线程）的stack
[stack:<tid>]    线程ID为tid的stack. 对应于/proc/[pid]/task/[tid]/路径
[vdso]    Virtual Dynamically linked Shared Object
[heap]    进程的heap
````

## root

`/proc/[pid]/root`是进程根目录的符号链接。举例如下：

````
# ls -lt /proc/2948/root
lrwxrwxrwx 1 root root 0 Nov  9 12:14 /proc/2948/root -> /
````

## stack

`/proc/[pid]/stack`显示当前进程的内核调用栈信息，只有内核编译时打开了CONFIG_STACKTRACE编译选项，才会生成这个文件。举例如下：

````
# cat /proc/2948/stack
[<ffffffff80168375>] poll_schedule_timeout+0x45/0x60
[<ffffffff8016994d>] do_sys_poll+0x49d/0x550
[<ffffffff80169abd>] SyS_poll+0x5d/0xf0
[<ffffffff804c16e7>] system_call_fastpath+0x16/0x1b
[<00007f4a41ff2c1d>] 0x7f4a41ff2c1d
[<ffffffffffffffff>] 0xffffffffffffffff
````

## statm

`/proc/[pid]/statm`显示进程所占用内存大小的统计信息，包含七个值，度量单位是page（page大小可通过getconf PAGESIZE得到）。举例如下：

````
# cat /proc/2948/statm  
72362 12945 4876 569 0 24665 0
````

各个值含义：
a）进程占用的总的内存；
b）进程当前时刻占用的物理内存；
c）同其它进程共享的内存；
d）进程的代码段；
e）共享库（从2.6版本起，这个值为0）；
f）进程的堆栈；
g）dirty pages（从2.6版本起，这个值为0）。


## syscall

`/proc/[pid]/syscall`显示当前进程正在执行的系统调用。举例如下：

````
# cat /proc/2948/syscall
7 0x7f4a452cbe70 0xb 0x1388 0xffffffffffdff000 0x7f4a4274a750 0x0 0x7ffd1a8033f0 0x7f4a41ff2c1d
````

第一个值是系统调用号（7代表poll），后面跟着6个系统调用的参数值（位于寄存器中），最后两个值依次是堆栈指针和指令计数器的值。如果当前进程虽然阻塞，但阻塞函数并不是系统调用，则系统调用号的值为-1，后面只有堆栈指针和指令计数器的值。如果进程没有阻塞，则这个文件只有一个“running”的字符串。

内核编译时打开了CONFIG_HAVE_ARCH_TRACEHOOK编译选项，才会生成这个文件。

## wchan

`/proc/[pid]/wchan`显示当进程休眠时，内核当前运行的函数。举例如下：

````
# cat /proc/2948/wchan
kauditd_
````

## 其他

`/proc/[pid]/task`，目录，每个线程一个子目录，目录名为线程ID。 

> to be continue...