---
title: 我的汇编学习之路（7）
date: 2016-5-18 13:11:33
tags: [asm,nasm]
categories: 汇编
---
这是系列的第七篇，接下来要讲一下如何在c中使用汇编。
其实我们有三种方法：
* 从C代码调用汇编例程
* 从汇编代码调用c例程
* 在C代码中使用内联汇编

我们来编写3个简单的Hello World程序，向我们展示如何使用assembly和C在一起。
<!-- more -->
# 从C中调用汇编
首先让我们来写简单的C程序：
````c
#include <string.h>

int main() {
	char* str = "Hello World\n";
	int len = strlen(str);
	printHelloWorld(str, len);
	return 0;
}
````
这里我们可以看到定义两个变量的C代码：我们将写入stdout的Hello world字符串和该字符串的长度。接下来我们使用这2个变量作为参数调用printHelloWorld汇编函数。当我们使用x86\_64 Linux时，我们必须知道x86\_64 linux调用协议，所以我们将知道如何编写printHelloWorld函数，如何获取输入参数等等。当我们调用函数时，前六个参数通过rdi，rsi，rdx，rcx，r8和r9通用寄存器传递，其他的通过堆栈。因此，我们可以从rdi和rsi寄存器获取第一个和第二个参数，并调用write syscall，之后从ret指令返回函数：
````s
global printHelloWorld

section .text
printHelloWorld:
		;; 1 arg
		mov r10, rdi
		;; 2 arg
		mov r11, rsi
		;; call write syscall
		mov rax, 1
		mov rdi, 1
		mov rsi, r10
		mov rdx, r11
		syscall
		ret
````
用下面命令编译：
````make
build:
	nasm -f elf64 -o casm.o casm.asm
	gcc casm.o casm.c -o casm
````

# 内联汇编
以下方法是直接在C代码中编写汇编代码。这里有特殊的语法：
````
asm [volatile] ("assembly code" : output operand : input operand : clobbers);
````
我们可以在gcc文档中阅读volatile关键字的意思：
>The typical use of Extended asm statements is to manipulate input values to produce output values. However, your asm statements may also produce side effects. If so, you may need to use the volatile qualifier to disable certain optimizations

每个操作数由约束字符串描述，后跟着一个括号，里面是c表达式。有一些限制：
* r - 通用寄存器中的变量值
* g - 允许任何寄存器，存储器或立即整数操作数，但非通用寄存器的寄存器除外。
* f - 浮点寄存器
* m - 允许存储器操作数，具有机器一般支持的任何类型的地址。
* 等等…

所以我们的hello world是这样的：
````c
#include <string.h>

int main() {
	char* str = "Hello World\n";
	long len = strlen(str);
	int ret = 0;

	__asm__("movq $1, %%rax \n\t"
		"movq $1, %%rdi \n\t"
		"movq %1, %%rsi \n\t"
		"movl %2, %%edx \n\t"
		"syscall"
		: "=g"(ret)
		: "g"(str), "g" (len));

	return 0;
}
````
这里我们能看到跟前面例子一样，定义了两个变量。首先我们赋值1给rax和rdi寄存器（write system call号和stdout）。接下来我们做同样的操作给rsi和edx，只不过操作数用$开头而不是%开头。这里表示了str被%1引用，len被%2引用，所以我们这里用%n的方式把str和len的值赋值到rsi和edx上，这里的n代表参数的数字。同样你能看到，寄存器是以%%开头的。
> This helps GCC to distinguish between the operands and registers. operands have a single % as prefix

编译如下：
````
build:
	gcc casm.c -o casm
````

# 汇编里面调用c
最后一种方法是从汇编代码调用C函数。例如，我们有以下简单的C代码与一个函数，只打印Hello world：
````c
#include <stdio.h>

extern int print();

int print() {
	printf("Hello World\n");
	return 0;
}
````
现在我们可以在我们的汇编代码中将这个函数定义为extern，并且像以前那样在调用指令中进行调用：
````s
global _start

extern print

section .text

_start:
		call print

		mov rax, 60
		mov rdi, 0
		syscall
````
编译
````
build:
	gcc  -c casm.c -o c.o
	nasm -f elf64 casm.asm -o casm.o
	ld   -dynamic-linker /lib64/ld-linux-x86-64.so.2 -lc casm.o c.o -o casm
````
现在我们可以运行我们的第三个hello world。

原文质量有些描述冗余，翻译过程进行缩减了，还有语法问题，哎。。。勉强看看吧
翻译 https://0xax.github.io/asm_7
已加入我的repo https://github.com/ejunjsh/asm-code