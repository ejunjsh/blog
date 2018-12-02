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

## status

status文件内容的格式如下（查看当前shell命令所在进程的信息）：

````
$ cat /proc/$$/status
Name:   bash
State:  S (sleeping)
Tgid:   3515
Pid:    3515
PPid:   3452
TracerPid:      0
Uid:    1000    1000    1000    1000
Gid:    100     100     100     100
FDSize: 256
Groups: 16 33 100
VmPeak:     9136 kB
VmSize:     7896 kB
VmLck:         0 kB
VmPin:         0 kB
VmHWM:      7572 kB
VmRSS:      6316 kB
VmData:     5224 kB
VmStk:        88 kB
VmExe:       572 kB
VmLib:      1708 kB
VmPMD:         4 kB
VmPTE:        20 kB
VmSwap:        0 kB
Threads:        1
SigQ:   0/3067
SigPnd: 0000000000000000
ShdPnd: 0000000000000000
SigBlk: 0000000000010000
SigIgn: 0000000000384004
SigCgt: 000000004b813efb
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000
CapBnd: ffffffffffffffff
CapAmb:   0000000000000000
Seccomp:        0
Cpus_allowed:   00000001
Cpus_allowed_list:      0
Mems_allowed:   1
Mems_allowed_list:      0
voluntary_ctxt_switches:        150
nonvoluntary_ctxt_switches:     545
````

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
`/proc/[pid]/stat`，文件，进程状态信息，用于ps命令。 
`/proc/[pid]/uid_map`，文件，用户ID映射信息，详见（man user_namespaces）。 
`/proc/[pid]/gid_map`，文件，组ID映射信息，详见（man user_namespaces）。 
`/proc/[pid]/mountinfo`，文件，挂载信息，格式为36 35 98:0 /mnt1 /mnt2 rw,noatime master:1 - ext3 /dev/root rw,errors=continue，以空格作为分隔符，从左到右各字段的意思分别是唯一挂载ID、父挂载ID、文件系统的设备主从号码、文件系统中挂载的根节点、相对于进程根节点的挂载点、挂载权限等挂载配置、可选配置、短横线表示前面可选配置的结束、文件系统类型、文件系统特有的挂载源或者为none、额外配置。 
`/proc/[pid]/mounts`，文件，挂载在当前进程的文件系统列表，格式参照（man fstab）。 
`/proc/[pid]/mountstats`，文件，挂载信息，格式形如device /dev/sda7 mounted on /home with fstype ext3 \[statistics\]。 
`/proc/[pid]/ns/`，目录，保存了每个名字空间的入口，详见（man namespaces）。 


# net目录

/proc/net，目录，网络伪文件系统相关。 

/proc/net/arp 
/proc/net/dev 
/proc/net/dev_mcast 
/proc/net/igmp 
/proc/net/rarp 
/proc/net/raw 
/proc/net/snmp 
/proc/net/tcp 
/proc/net/udp 
/proc/net/unix 
/proc/net/netfilter/nfnetlink_queue

# sys目录

/proc/sys，目录，系统变量相关信息，详细如下。 

/proc/sys/abi 
/proc/sys/debug 
/proc/sys/dev 
/proc/sys/fs 
/proc/sys/fs/binfmt_misc 
/proc/sys/fs/dentry-state 
/proc/sys/fs/dir-notify-enable 
/proc/sys/fs/dquot-max 
/proc/sys/fs/dquot-nr 
/proc/sys/fs/epoll 
/proc/sys/fs/file-max 
/proc/sys/fs/file-nr 
/proc/sys/fs/inode-max 
/proc/sys/fs/inode-nr 
/proc/sys/fs/inode-state 
/proc/sys/fs/inotify 
/proc/sys/fs/lease-break-time 
/proc/sys/fs/leases-enable 
/proc/sys/fs/mqueue 
/proc/sys/fs/nr_open 
/proc/sys/fs/overflowgid 
/proc/sys/fs/overflowuid 
/proc/sys/fs/pipe-max-size 
/proc/sys/fs/protected_hardlinks 
/proc/sys/fs/protected_symlinks 
/proc/sys/fs/suid_dumpable 
/proc/sys/fs/super-max 
/proc/sys/fs/super-nr 
/proc/sys/kernel 
/proc/sys/kernel/acct 
/proc/sys/kernel/auto_msgmni 
/proc/sys/kernel/cap_last_cap 
/proc/sys/kernel/cap-bound 
/proc/sys/kernel/core_pattern 
/proc/sys/kernel/core_uses_pid 
/proc/sys/kernel/ctrl-alt-del 
/proc/sys/kernel/dmesg_restrict 
/proc/sys/kernel/domainname 
/proc/sys/kernel/hostname 
/proc/sys/kernel/hotplug 
/proc/sys/kernel/htab-reclaim 
/proc/sys/kernel/kptr_restrict 
/proc/sys/kernel/l2cr 
/proc/sys/kernel/modprobe 
/proc/sys/kernel/modules_disabled 
/proc/sys/kernel/msgmax 
/proc/sys/kernel/msgmni 
/proc/sys/kernel/msgmnb 
/proc/sys/kernel/ngroups_max 
/proc/sys/kernel/ostype 
/proc/sys/kernel/osrelease 
/proc/sys/kernel/overflowgid 
/proc/sys/kernel/overflowuid 
/proc/sys/kernel/panic 
/proc/sys/kernel/panic_on_oops 
/proc/sys/kernel/pid_max 
/proc/sys/kernel/powersave-nap 
/proc/sys/kernel/printk 
/proc/sys/kernel/pty 
/proc/sys/kernel/pty/max 
/proc/sys/kernel/pty/nr 
/proc/sys/kernel/random 
/proc/sys/kernel/random/uuid 
/proc/sys/kernel/randomize_va_space 
/proc/sys/kernel/real-root-dev 
/proc/sys/kernel/reboot-cmd 
/proc/sys/kernel/rtsig-max 
/proc/sys/kernel/rtsig-nr 
/proc/sys/kernel/sched_rr_timeslice_ms 
/proc/sys/kernel/sched_rt_period_us 
/proc/sys/kernel/sched_rt_period_us 
/proc/sys/kernel/sem 
/proc/sys/kernel/sg-big-buff 
/proc/sys/kernel/shm_rmid_forced 
/proc/sys/kernel/shmall 
/proc/sys/kernel/shmmax 
/proc/sys/kernel/shmmni 
/proc/sys/kernel/sysctl_writes_strict 
/proc/sys/kernel/sysrq 
/proc/sys/kernel/version 
/proc/sys/kernel/threads-max 
/proc/sys/kernel/yama/ptrace_scope 
/proc/sys/kernel/zero-paged 
/proc/sys/net 
/proc/sys/net/core/bpf_jit_enable 
/proc/sys/net/core/somaxconn 
/proc/sys/proc 
/proc/sys/sunrpc 
/proc/sys/vm 
/proc/sys/vm/compact_memory 
/proc/sys/vm/drop_caches 
/proc/sys/vm/legacy_va_layout 
/proc/sys/vm/memory_failure_early_kill 
/proc/sys/vm/memory_failure_recovery 
/proc/sys/vm/oom_dump_tasks 
/proc/sys/vm/oom_kill_allocating_task 
/proc/sys/vm/overcommit_kbytes 
/proc/sys/vm/overcommit_memory 
/proc/sys/vm/overcommit_ratio 
/proc/sys/vm/panic_on_oom 
/proc/sys/vm/swappiness

# 其它

/proc/apm，文件，apm即Advanced Power Management，需要配置CONFIG_APM。 
/proc/buddyinfo，文件，用于诊断内存碎片问题。 
/proc/cmdline，文件，系统启动时传递给Linux内核的参数，如lilo、grub等boot管理模块。 
/proc/config.gz，文件，内核编译配置选项，需要配置CONFIG_IKCONFIG_PROC。 
/proc/crypto，文件，内核加密API提供的加密列表。 
/proc/cpuinfo，文件，CPU和系统架构信息，lscpu命令使用这个文件。 
/proc/devices，文件，设备相关信息。 
/proc/diskstats，文件，磁盘状态。 
/proc/dma，文件，dma即Direct Memory Access。 
/proc/driver/rtc，文件，系统运行时配置。 
/proc/execdomains，文件，执行域列表。 
/proc/fb，文件，Frame Buffer信息，需要配置CONFIG_FB。 
/proc/filesystems，文件，内核支持的文件系统类型（man filesystems）。 
/proc/fs，目录，挂载的文件系统信息。 
/proc/ide，目录，用于IDE接口。 
/proc/interrupts，文件，每个CPU每个IO的中断信息。 
/proc/iomem，文件，IO内存映射信息。 
/proc/ioports，文件，IO端口信息。 
/proc/kallsyms，文件，用于动态链接和和模块绑定的符号定义。 
/proc/kcore，文件，系统中ELF格式的物理内存。 
/proc/kmsg，文件，内核信息，dmsg命令使用这个文件。 
/proc/kpagecount，文件，每个物理页帧映射的次数，需要配置CONFIG_PROC_PAGE_MONITOR。 
/proc/kpageflags，文件，每个物理页帧的掩码，需要配置CONFIG_PROC_PAGE_MONITOR。 
/proc/ksyms，文件，同kallsyms。 
/proc/loadavg，文件，工作负荷。 
/proc/locks，文件，当前文件锁的状态。 
/proc/malloc，文件，需要配置CONFIG_DEBUG_MALLOC。 
/proc/meminfo，文件，系统内存使用统计，free命令使用了这个文件。 
/proc/modules，文件，系统加载的模块信息，相关命令为lsmod。 
/proc/mounts，文件，链接到了/self/mounts。 
/proc/mtrr，文件，Memory Type Range Registers。 
/proc/partitions，文件，分区信息。 
/proc/pci，文件，PCI接口设备。 
/proc/profile，文件，用于readprofile命令作性能分析。 
/proc/scsi，目录，SCSI接口设备。 
/proc/scsi/scsi 
/proc/scsi/\[drivername\] 
/proc/self，目录，链接到了当前进程所在的目录。 
/proc/slabinfo，文件，内核缓存信息，需要配置CONFIG_SLAB。 
/proc/stat，文件，系统信息统计。 
/proc/swaps，文件，使用的交换空间。 
/proc/sysrq-trigger，文件，可写，触发系统调用。 
/proc/sysvipc，目录，包括msg、sem、shm三个文件，为System V IPC对象。 
/proc/thread-self，文件，链接到了当前进程下的task目录中的线程文件。 
/proc/timer_list，文件，还在运行着的定时器列表。 
/proc/timer_stats，文件，定时器状态。 
/proc/tty，目录，tty设备相关。 
/proc/uptime，文件，系统更新时间和进程空闲时间。 
/proc/version，文件，内核版本信息。 
/proc/vmstat，文件，内存统计信息，以键值对形式显示。 
/proc/zoneinfo，文件，内存区块信息，用于分析虚拟内存的行为。


# 参考

https://blog.csdn.net/ieearth/article/details/72849990

https://www.cnblogs.com/likui360/p/6181927.html