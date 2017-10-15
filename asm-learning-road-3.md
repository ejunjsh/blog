---
title: 我的汇编学习之路（3）
date: 2016-5-14 14:21:33
tags: [asm,nasm]
categories: 汇编
---
栈是在内存中是一个特殊的区域,它的主要操作是lifo（后进先出）

我们有16个通用的寄存器，用来存储临时数据。它们分别是RAX, RBX, RCX, RDX, RDI, RSI, RBP, RSP 和 R8-R15。对于真实的应用程序来说，这16个寄存器太少了，所以我们用栈存储数据。另外，栈还有其他用法：当你调用一个函数，函数的返回地址拷贝到栈。当函数执行完后，地址从栈拷贝到命令计数器（RIP），应用程序就可以从函数调用的下个位置继续执行。
举个例子：
````s
global _start

section .text

_start:
		mov rax, 1
		call incRax
		cmp rax, 2
		jne exit
		;;
		;; Do something
		;;

incRax:
		inc rax
		ret
````
<!-- more -->
这里我能看到，程序运行时，rax等于1.当调用incRax,一个给rax加一的函数之后，rax等于2.函数调用后，从第八行继续，那里我们比较了rax是否等于2.我们也可以在[这里](https://en.wikipedia.org/wiki/X86_calling_conventions#x86-64_calling_conventions)看到，函数的前六个参数使用寄存器来传递的，他们是：
* `rdi` - 第一个参数
* `rsi` - 第二个参数
* `rdx` - 第三个参数
* `rcx` - 第四个参数
* `r8` - 第五个参数
* `r9` - 第六个参数

接下来的参赛将用栈来传递。所以假如我有以下一个函数：
````c
int foo(int a1, int a2, int a3, int a4, int a5, int a6, int a7)
{
    return (a1 + a2 - a3 - a4 + a5 - a6) * a7;
}
````
前六个参数使用寄存器传递，第七个用栈来传递。

# 栈指针
之前写的，我们有16个通用寄存器，其中两个是有趣的：RSP和RBP。RBP是个基础指针寄存器。它指向当前栈帧的底部。RSP是栈地址，指向当前栈帧顶部。

## 命令
我们有两个命令用来操作栈：
* `push 参数` - 增加栈指针（RSP），存储参数到栈地址指向的位置。
* `pop 参数` - 拷贝栈地址指向位置的数据到参数。

让我们看一个例子吧：
````s
global _start

section .text

_start:
		mov rax, 1
		mov rdx, 2
		push rax
		push rdx

		mov rax, [rsp + 8]

		;;
		;; Do something
		;;
````
这里我们能看到我们把1放到rax，2放到rdx，之后用这些寄存器依次放入到栈。