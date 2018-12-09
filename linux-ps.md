---
title: linux的ps详解
date: 2014-02-20 18:23:33
tags: [linux,ps]
categories: linux命令
---

> 这个命令用到烂了，凑数吧👿

# 准备知识

## Linux进程状态

1. 运行(正在运行或在运行队列中等待)
2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)
3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)
5. 停止(进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行)

<!-- more -->

## 常用参数

* ps a 显示现行终端机下的所有程序，包括其他用户的程序。
* ps -A 显示所有进程。
* ps c 列出程序时，显示每个程序真正的指令名称，而不包含路径，参数或常驻服务的标示。
* ps -e 此参数的效果和指定"A"参数相同。
* ps e 列出程序时，显示每个程序所使用的环境变量。
* ps f 用ASCII字符显示树状结构，表达程序间的相互关系。
* ps -f 全格式
* ps -H 显示树状结构，表示程序间的相互关系。
* ps -N 显示所有的程序，除了执行ps指令终端机下的程序之外。
* ps s 采用程序信号的格式显示程序状况。
* ps S 列出程序时，包括已中断的子程序资料。
* ps -t<终端机编号> 　指定终端机编号，并列出属于该终端机的程序的状况。
* ps u 　以用户为主的格式来显示程序状况。
* ps x 　显示所有程序，不以终端机来区分。
* ps l  较长、较详细的将该PID 的的信息列出；

## 常用输出列说明

````shell
$ ps aux
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
root 1 0.0 0.0 10368 176 Ss May25 0:24 init [3]
root 2 0.0 0.0 0 0 S May25 0:00 [kthreadd/2336]
root 3 0.0 0.0 0 0 S May25 0:00 [khelper/2336]
root 135 0.0 0.0 12636 320 S<s May25 0:00 /sbin/udevd -d
root 569 0.0 0.0 5928 464 Ss May25 1:07 syslogd -m 0
root 580 0.0 0.1 62684 640 Ss May25 0:48 /usr/sbin/sshd
````

* USER：该 process 属于那个使用者账号的
* PID ：该 process 的号码。
* %CPU：该 process 使用掉的 CPU 资源百分比；
* %MEM：该 process 所占用的物理内存百分比；
* VSZ ：该 process 使用掉的虚拟内存量 (Kbytes)
* RSS ：该 process 占用的固定的内存量 (Kbytes)
* TTY ：该 process 是在那个终端机上面运作，若与终端机无关，则显示 ，另外， tty1-tty6 是本机上面的登入者程序，若为 pts/0 等等的，则表示为由网络连接进主机的程序。
* STAT：该程序目前的状态，主要的状态有：
    * D      //无法中断的休眠状态（通常 IO 的进程）； 
    * R      //正在运行可中在队列中可过行的； 
    * S      //处于休眠状态； 
    * T      //停止或被追踪； 
    * W      //进入内存交换 （从内核2.6开始无效）；
    * X      //死掉的进程 （基本很少见）； 
    * Z      //僵尸进程； 
    * <      //优先级高的进程 
    * N      //优先级较低的进程 
    * L      //有些页被锁进内存； 
    * s      //进程的领导者（在它之下有子进程）； 
    * l      //多线程，克隆线程（使用 CLONE_THREAD, 类似 NPTL pthreads）； 
    * +      //位于后台的进程组；
* START：该 process 被触发启动的时间；
* TIME ：该 process 实际使用 CPU 运作的时间。
* COMMAND：该程序的实际指令为何？

````shell
# ps l
F   UID    PID   PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
4     0   1885      1  20   0  15936  1772 poll_s Ss+  tty1       0:00 /sbin/agetty --noclear tty1 linux
4     0   3355   3234  20   0  52936  3980 poll_s S    pts/0      0:00 sudo -i
4     0   3356   3355  20   0  22564  5100 wait   S    pts/0      0:00 -bash
0     0   3369   3356  20   0  28916  1448 -      R+   pts/0      0:00 ps l
````

* F 代表这个程序的旗标 (flag)， 4 代表使用者为 superuser；
* STAT 代表这个程序的状态；
* UID 代表执行者身份
* PID 进程的ID号！
* PPID 父进程的ID；
* PRI指进程的执行优先权(Priority的简写)，其值越小越早被执行；
* NI 这个进程的nice值，其表示进程可被执行的优先级的修正数值。
* WCHAN 目前这个程序是否正在运作当中，若为 - 表示正在运作；
* TTY 登入者的终端机位置；
* TIME 使用掉的 CPU 时间。
* CMD 所下达的指令名称


# 常用例子

## 根据用户过滤进程

在需要查看特定用户进程的情况下，我们可以使用 -u 参数。比如我们要查看用户'sky'的进程，可以通过下面的命令：

````shell
$ ps -u sky
   PID TTY          TIME CMD
  3171 ?        00:00:00 systemd
  3174 ?        00:00:00 (sd-pam)
  3273 ?        00:00:00 sshd
  3274 pts/0    00:00:00 bash
  3361 pts/0    00:00:00 ps
````

## 通过cpu和内存使用来过滤进程

也许你希望把结果按照 CPU 或者内存用量来筛选，这样你就找到哪个进程占用了你的资源。要做到这一点，我们可以使用 aux 参数，来显示全面的信息:

````shell
$ ps -aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.8  0.0  37676  5692 ?        Ss   04:43   0:03 /sbin/init
root          2  0.0  0.0      0     0 ?        S    04:43   0:00 [kthreadd]
root          3  0.0  0.0      0     0 ?        S    04:43   0:00 [ksoftirqd/0]
root          5  0.0  0.0      0     0 ?        S<   04:43   0:00 [kworker/0:0H]
root          7  0.0  0.0      0     0 ?        S    04:43   0:00 [rcu_sched]
root          8  0.0  0.0      0     0 ?        S    04:43   0:00 [rcu_bh]
root          9  0.0  0.0      0     0 ?        S    04:43   0:00 [migration/0]
root         10  0.0  0.0      0     0 ?        S    04:43   0:00 [watchdog/0]
root         11  0.0  0.0      0     0 ?        S    04:43   0:00 [watchdog/1]
root         12  0.0  0.0      0     0 ?        S    04:43   0:00 [migration/1]
root         13  0.0  0.0      0     0 ?        S    04:43   0:00 [ksoftirqd/1]
root         14  0.0  0.0      0     0 ?        S    04:43   0:00 [kworker/1:0]
root         15  0.0  0.0      0     0 ?        S<   04:43   0:00 [kworker/1:0H]
...
````

默认的结果集是未排好序的。可以通过 --sort命令来排序。

根据 CPU 使用来升序排序

````shell
$ ps -aux --sort -pcpu
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.7  0.0  37676  5692 ?        Ss   04:41   0:03 /sbin/init
root       1632  0.7  0.6 567304 49336 ?        Ssl  04:41   0:03 /usr/bin/dockerd -H fd://
root         84  0.2  0.0      0     0 ?        S    04:41   0:00 [kworker/2:1]
root        579  0.1  0.0  46552  5960 ?        Ss   04:41   0:00 /lib/systemd/systemd-udevd
root        905  0.1  0.1 189980  9892 ?        Ssl  04:41   0:00 /usr/bin/vmtoolsd
root       1917  0.1  0.2 302464 16640 ?        Ssl  04:41   0:00 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-conta
root          2  0.0  0.0      0     0 ?        S    04:41   0:00 [kthreadd]
root          3  0.0  0.0      0     0 ?        S    04:41   0:00 [ksoftirqd/0]
root          5  0.0  0.0      0     0 ?        S<   04:41   0:00 [kworker/0:0H]
root          7  0.0  0.0      0     0 ?        S    04:41   0:00 [rcu_sched]
...
````

根据 内存使用 来升序排序

````shell
$ ps -aux --sort -pmem
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       1632  0.6  0.6 567304 49336 ?        Ssl  04:41   0:03 /usr/bin/dockerd -H fd://
root       1056  0.0  0.3 300408 29972 ?        Ssl  04:41   0:00 /usr/lib/snapd/snapd
root       1917  0.1  0.2 302464 16640 ?        Ssl  04:41   0:00 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-conta
root        905  0.1  0.1 189980  9892 ?        Ssl  04:41   0:00 /usr/bin/vmtoolsd
root       1082  0.0  0.1  85436  9172 ?        Ss   04:41   0:00 /usr/bin/VGAuthService
root       3169  0.0  0.0  92800  6784 ?        Ss   04:41   0:00 sshd: sky [priv]
root       1043  0.0  0.0 275868  6236 ?        Ssl  04:41   0:00 /usr/lib/accountsservice/accounts-daemon
root       1189  0.0  0.0 277180  6004 ?        Ssl  04:41   0:00 /usr/lib/policykit-1/polkitd --no-debug
root        579  0.1  0.0  46552  5960 ?        Ss   04:41   0:00 /lib/systemd/systemd-udevd
root          1  0.6  0.0  37676  5692 ?        Ss   04:41   0:03 /sbin/init
root       1630  0.0  0.0  65508  5380 ?        Ss   04:41   0:00 /usr/sbin/sshd -D
...
````

我们也可以将它们合并到一个命令

````shell
$ ps -aux --sort -pcpu,+pmem
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       1632  0.6  0.6 567304 49336 ?        Ssl  04:41   0:03 /usr/bin/dockerd -H fd://
root          1  0.5  0.0  37676  5692 ?        Ss   04:41   0:03 /sbin/init
root         84  0.2  0.0      0     0 ?        S    04:41   0:01 [kworker/2:1]
root        905  0.1  0.1 189980  9892 ?        Ssl  04:41   0:00 /usr/bin/vmtoolsd
root       1917  0.1  0.2 302464 16640 ?        Ssl  04:41   0:00 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-conta
root          2  0.0  0.0      0     0 ?        S    04:41   0:00 [kthreadd]
root          3  0.0  0.0      0     0 ?        S    04:41   0:00 [ksoftirqd/0]
root          5  0.0  0.0      0     0 ?        S<   04:41   0:00 [kworker/0:0H]
root          7  0.0  0.0      0     0 ?        S    04:41   0:00 [rcu_sched]
root          8  0.0  0.0      0     0 ?        S    04:41   0:00 [rcu_bh]
root          9  0.0  0.0      0     0 ?        S    04:41   0:00 [migration/0]
root         10  0.0  0.0      0     0 ?        S    04:41   0:00 [watchdog/0]
root         11  0.0  0.0      0     0 ?        S    04:41   0:00 [watchdog/1]
root         12  0.0  0.0      0     0 ?        S    04:41   0:00 [migration/1]
root         13  0.0  0.0      0     0 ?        S    04:41   0:00 [ksoftirqd/1]
root         14  0.0  0.0      0     0 ?        S    04:41   0:00 [kworker/1:0]
root         15  0.0  0.0      0     0 ?        S<   04:41   0:00 [kworker/1:0H]
...
````

## 通过进程的命令来查找进程

使用 -C 参数，后面跟你要找的进程的命令的名字。比如想显示一个名为docker-containerd的进程的信息，就可以使用下面的命令：

````shell
$ ps -C docker-containerd
   PID TTY          TIME CMD
  1917 ?        00:00:00 docker-containe
````

如果想要看到更多的细节，我们可以使用-f参数来查看格式化的信息列表：

````shell
$ ps -f -C docker-containerd
UID         PID   PPID  C STIME TTY          TIME CMD
root       1917   1632  0 04:41 ?        00:00:00 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --met
````

## 树形显示进程

有时候我们希望以树形结构显示进程，可以使用 -axjf 参数。

````shell
$ ps -axjf
  PPID    PID   PGID    SID TTY       TPGID STAT   UID   TIME COMMAND
     0      2      0      0 ?            -1 S        0   0:00 [kthreadd]
     2      3      0      0 ?            -1 S        0   0:00  \_ [ksoftirqd/0]
     2      5      0      0 ?            -1 S<       0   0:00  \_ [kworker/0:0H]
     2      7      0      0 ?            -1 S        0   0:00  \_ [rcu_sched]
     2      8      0      0 ?            -1 S        0   0:00  \_ [rcu_bh]
     2      9      0      0 ?            -1 S        0   0:00  \_ [migration/0]
     2     10      0      0 ?            -1 S        0   0:00  \_ [watchdog/0]
     2     11      0      0 ?            -1 S        0   0:00  \_ [watchdog/1]
     2     12      0      0 ?            -1 S        0   0:00  \_ [migration/1]
     2     13      0      0 ?            -1 S        0   0:00  \_ [ksoftirqd/1]
     2     14      0      0 ?            -1 S        0   0:00  \_ [kworker/1:0]
     2     15      0      0 ?            -1 S<       0   0:00  \_ [kworker/1:0H]
     2     16      0      0 ?            -1 S        0   0:00  \_ [watchdog/2]
...
````

## 显示安全信息

如果想要查看现在有谁登入了你的服务器。可以使用ps命令加上相关参数:

````shell
$ ps -eo pid,user,args
   PID USER     COMMAND
     1 root     /sbin/init
     2 root     [kthreadd]
     3 root     [ksoftirqd/0]
     5 root     [kworker/0:0H]
     7 root     [rcu_sched]
     8 root     [rcu_bh]
     9 root     [migration/0]
    10 root     [watchdog/0]
    11 root     [watchdog/1]
...
````

参数 -e 显示所有进程信息，-o 参数控制输出。Pid,User 和 Args参数显示PID，运行应用的用户和该应用的命令行参数。

能够与-e 参数 一起使用的关键字是args, cmd, comm, command, fname, ucmd, ucomm, lstart, bsdstart 和 start。

## 格式化输出root用户（真实的或有效的UID）创建的进程

系统管理员想要查看由root用户运行的进程和这个进程的其他相关信息时，可以通过下面的命令:

````shell
$ ps -U root -u root u
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.0  37676  5692 ?        Ss   04:41   0:03 /sbin/init
root          2  0.0  0.0      0     0 ?        S    04:41   0:00 [kthreadd]
root          3  0.0  0.0      0     0 ?        S    04:41   0:00 [ksoftirqd/0]
root          5  0.0  0.0      0     0 ?        S<   04:41   0:00 [kworker/0:0H]
root          7  0.0  0.0      0     0 ?        S    04:41   0:00 [rcu_sched]
root          8  0.0  0.0      0     0 ?        S    04:41   0:00 [rcu_bh]
root          9  0.0  0.0      0     0 ?        S    04:41   0:00 [migration/0]
root         10  0.0  0.0      0     0 ?        S    04:41   0:00 [watchdog/0]
root         11  0.0  0.0      0     0 ?        S    04:41   0:00 [watchdog/1]
root         12  0.0  0.0      0     0 ?        S    04:41   0:00 [migration/1]
root         13  0.0  0.0      0     0 ?        S    04:41   0:00 [ksoftirqd/1]
root         15  0.0  0.0      0     0 ?        S<   04:41   0:00 [kworker/1:0H]
root         16  0.0  0.0      0     0 ?        S    04:41   0:00 [watchdog/2]
root         17  0.0  0.0      0     0 ?        S    04:41   0:00 [migration/2]
````

-U 参数按真实用户ID(RUID)筛选进程，它会从用户列表中选择真实用户名或 ID。真实用户即实际创建该进程的用户。

-u 参数用来筛选有效用户ID（EUID）。

最后的u参数用来决定以针对用户的格式输出，由User, PID, %CPU, %MEM, VSZ, RSS, TTY, STAT, START, TIME 和 COMMAND这几列组成。

# 参考

http://yanue.net/post-87.html
https://www.cnblogs.com/wxgblogs/p/6591980.html
https://linux.cn/article-4743-1.html