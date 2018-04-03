---
title: linux命令-gdb
date: 2017-12-28 11:31:23
tags: [linux,gdb]
categories: linux命令
---
> 备忘，mark 👿

# GDB 基础知识

## GDB 相关概念
GDB, 是 The GNU Project Debugger 的缩写, 是 Linux 下功能全面的调试工具。GDB 支持断点、单步执行、打印变量、观察变量、查看寄存器、查看堆栈等调试手段。在 Linux 环境软件开发中，GDB 是主要的调试工具，用来调试 C 和 C++ 程序。

## GDB 的进入和退出
如果要调试程序，需要在 gcc 编译可执行程序时加上 -g 参数，首先我们编译 bugging.c 程序，生成可执行文件：
````shell
gcc -g -o bugging bugging.c
````
输入 gdb bugging 进入 gdb 调试 bugging 程序的界面：
````shell
gdb bugging
````
在 gdb 命令行界面，输入run 执行待调试程序：
````shell
(gdb) run
````
<!-- more -->
在 gdb 命令行界面，输入quit 退出 gdb：
````shell
(gdb) quit
````
上述步骤的输出
````shell
sky@ubuntu:~/gdbtest$ gcc -g -o bugging bugging.c
sky@ubuntu:~/gdbtest$ gdb bugging
GNU gdb (Ubuntu 8.0.1-0ubuntu1) 8.0.1
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
---Type <return> to continue, or q <return> to quit---
Type "apropos word" to search for commands related to "word"...
Reading symbols from bugging...done.
(gdb) run
Starting program: /home/sky/gdbtest/bugging
1+2+3+...+100= 1431657159
[Inferior 1 (process 27193) exited normally]
(gdb) q
sky@ubuntu:~/gdbtest$
````
## GDB 命令行界面使用技巧
### 命令补全
任何时候都可以使用`TAB`进行补全，如果只有一个待选选项则直接补全；否则会列出可选选项，继续键入命令，同时结合`TAB`即可快速输入命令。

### 部分 gdb 常用命令一览表
命令	|简写形式	|说明
-----|--------|------
list|	l	|查看源码
backtrace|	bt、where	|打印函数栈信息
next|	n|	执行下一行
step|	s	|一次执行一行，遇到函数会进入
finish	|	|运行到函数结束
continue|	c	|继续运行
break|	b|	设置断点
info breakpoints	|	|显示断点信息
delete|	d	|删除断点
print	|p	|打印表达式的值
run	|r	|启动程序
until	|u	|执行到指定行
info|	i	|显示信息
help|	h	|帮助信息

### 查询用法
在 gdb 命令行界面，使用 `(gdb) help command` 可以查看命令的用法。

### 执行 Shell 命令
在 gdb 命令行界面可以执行外部的 Shell 命令：
````shell
(gdb) !shell 命令
````
例如查看当前目录的文件：
````shell
(gdb) !ls
bugging  bugging.c
(gdb)
````
# GDB 断点
## 重新进入 debugging 调试界面
````shell
gdb bugging
````
## 查看源码
list 命令用来显示源文件中的代码。

### 通过行号查看源码
list 行号，显示某一行附近的代码：
````shell
(gdb) list 2
1       /* bugging.c */
2
3       #include <stdio.h>
4
5       int foo(int n)
6       {
7
8           int sum;
9           int i;
10
(gdb)
````
list 文件名 : 行号，显示某一个文件某一行附近的代码，用于多个源文件的情况。

### 通过函数查看源码
list 函数名，显示某个函数附近的代码：
````shell
(gdb) list main
15
16          return sum;
17      }
18
19      int main(int argc, char** argv)
20      {
21          int result = 0;
22          int N = 100;
23
24          result = foo(N);
(gdb)
````
list 文件名 : 函数名，显示某一个文件某个函数附近的代码，用于多个源文件的情况。

## 设置断点
break 命令用来设置断点。

### 通过行号设置断点
break 行号，断点设置在该行开始处，__注意：该行代码未被执行：__
````shell
(gdb) break 19
Breakpoint 1 at 0x680: file bugging.c, line 19.
(gdb)
````
break 文件名 : 行号，适用于有多个源文件的情况。

### 通过函数设置断点
break 函数名，断点设置在该函数的开始处，__断点所在行未被执行：__
````shell
(gdb) break foo
Breakpoint 2 at 0x651: file bugging.c, line 11.
(gdb)
````
break 文件名 : 函数名，适用于有多个源文件的情况。

## 查看断点信息
info breakpoints 命令用于显示当前断点信息。
````shell
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000000680 in main at bugging.c:19
2       breakpoint     keep y   0x0000000000000651 in foo at bugging.c:11
(gdb)
````
其中每一项的信息：
* Num 列代表断点编号，该编号可以作为 delete/enalbe/disable 等控制断点命令的参数
* Type 列代表断点类型，一般为 breakpoint
* Disp 列代表断点被命中后，该断点保留(keep)、删除(del)还是关闭(dis)
* Enb 列代表该断点是 enable(y) 还是 disable(n)
* Address 列代表该断点处虚拟内存的地址
* What 列代表该断点在源文件中的信息

## 删除断点
delete 命令用于删除断点。
### 删除指定断点
delete Num，删除指定断点，断点编号可通过 info breakpoints 获得：
````shell
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000000680 in main at bugging.c:19
2       breakpoint     keep y   0x0000000000000651 in foo at bugging.c:11
(gdb) delete 2
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000000680 in main at bugging.c:19
(gdb)
````
### 删除所有断点
delete，不带任何参数，默认删除所有断点。

## 关闭和启用断点
disable 命令用于关闭断点，有些断点可能暂时不需要但又不想删除，便可以 disable 该断点。
enable 命令用于启用断点。

### 关闭所有断点
disable，不带任何参数，默认关闭所有断点。

### 关闭指定断点
disable Num，关闭指定断点，断点编号可通过 info breakpoints 获得：
````shell
(gdb) disable 1
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep n   0x0000000000000680 in main at bugging.c:19
(gdb)
````
### 启用所有断点
enable，不带任何参数，默认启用所有断点。

### 启用指定断点
enable Num，启用指定断点，断点编号可通过 info breakpoints 获得。
````shell
(gdb) enable 1
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000000680 in main at bugging.c:19
(gdb)
````
disable 和 enable 命令影响的是 info breakpoints 的 Enb 列，表示该断点是启用还是关闭

## 断点启用的更多方式
enable 命令还可以用来设置断点被执行的次数，比如当断点设在循环中的时候，某断点可能多次被命中。

### 断点 hit 一次之后关闭该断点
````shell
enable once Num
````
### 断点 hit 一次之后删除该断点
````shell
enable delete Num
````
实验中我们可以如下图测试该功能：
````shell
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000000680 in main at bugging.c:19
3       breakpoint     keep y   0x0000000000000651 in foo at bugging.c:11
(gdb) enable once 1
(gdb) enable delete 3
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     dis  y   0x0000000000000680 in main at bugging.c:19
3       breakpoint     del  y   0x0000000000000651 in foo at bugging.c:11
(gdb)
````
这两个命令影响的是 info breakpoints 的 Disp 列，表示该断点被命中之后的行为

## 断点小结
断点是调试最基本的方法之一，这一节主要介绍了断点相关的知识。主要是几个断点相关的命令。
* list
* info breakpoints
* break
* delete
* disable 和 enable
* enable once 和 enable delete

不熟悉命令的时候，记得在 gdb 命令行下键入 help info breakpoints 等命令，查询帮助文档。

# 单步调试
如果已经设置断点后，执行`run`,就会运行到断点：
````shell
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     dis  y   0x0000000000000680 in main at bugging.c:19
3       breakpoint     del  y   0x0000000000000651 in foo at bugging.c:11
(gdb) run
Starting program: /home/sky/gdbtest/bugging

Breakpoint 1, main (argc=1, argv=0x7fffffffdfd8) at bugging.c:21
21          int result = 0;
(gdb)
````

## 继续单步调试
上面输出会停到`main`函数这个断点，`next`或者`n`，就会单步运行：
````shell
Breakpoint 1, main (argc=1, argv=0x7fffffffdfd8) at bugging.c:21
21          int result = 0;
(gdb) n
22          int N = 100;
(gdb) next
24          result = foo(N);
(gdb)
````

## 显示变量
如果想显示某个变量的值，用`display <变量名>` ：
````shell
Temporary breakpoint 3, foo (n=100) at bugging.c:11
11          for (i=0; i<=n; i++)
(gdb) display sum
1: sum = 1431652109
(gdb)
````
上面代码我想显示`sum`这个变量，而且单步调试的时候，这个变量会一直显示。
````shell
(gdb) n
13              sum = sum+i;
1: sum = 1431652109
(gdb) n
11          for (i=0; i<=n; i++)
1: sum = 1431652109
(gdb) n
13              sum = sum+i;
1: sum = 1431652109
(gdb) n
11          for (i=0; i<=n; i++)
1: sum = 1431652110
(gdb)
````
## 退出单步调试
用`continue`或者`c`,退出单步调试
````shell
(gdb) c
Continuing.
1+2+3+...+100= 1431657159
[Inferior 1 (process 27595) exited normally]
(gdb)
````

# bugging.c代码
这是上面调试用的代码，此代码用来输出`1+2+3+...+100`的和，很明显上面最后的结果是一个很大值，为什么呢，你可以看上面单步调试里面显示某个变量的值还有以下注释👿
````c
/* bugging.c */

#include <stdio.h>

int foo(int n)
{

    int sum;  //没有初始化为0，所以这个变量的值为不确定
    int i;

    for (i=0; i<=n; i++)
    {
        sum = sum+i;
    }

    return sum;
}

int main(int argc, char** argv)
{
    int result = 0;
    int N = 100;

    result = foo(N);

    printf("1+2+3+...+%d= %d\n", N, result);

    return 0;

}
````

# 总结
很好用的工具用来调试c的代码，缺点当然就是没有图形界面咯。