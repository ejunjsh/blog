---
title: 我的汇编学习之路（8）
date: 2016-5-19 23:12:45
tags: [asm,nasm]
categories: 汇编
---
这是系列的第八篇（译者注：最终篇），接下来讲讲怎么在汇编使用非整型（浮点数）。
我们有一些方法可以处理浮点数据：
* fpu
* sse

首先让我们看看如何将浮点数存储在内存中。有三种浮点数据类型：
* 单精度
* 双精度
* 双倍扩展精度

正如英特尔的64-ia-32架构软件开发者-vol-1手册所述：
> The data formats for these data types correspond directly to formats specified in the IEEE Standard 754 for Binary Floating-Point Arithmetic.

内存中呈现的单精度浮点浮点数据：
* 符号 - 1位
* 指数 - 8位
* 尾数 - 23位

<!-- more -->
所以例如，如果我们有以下号码：
````
| sign  | exponent | mantissa
|-------|----------|-------------------------
| 0     | 00001111 | 110000000000000000000000
````
指数是从-128到127的8位有符号整数或从0到255的8位无符号整数。符号位为零，因此我们有正数。指数为00001111b或十进制十进制。对于单精度位移为127，这意味着我们需要计算指数 - 127或15 - 127 = -112。由于尾数的归一化二进制整数部分总是等于1，所以尾数只记录其小数部分，因此尾数或我们的数字是1,110000000000000000000000。结果值为：
````
value = mantissa * 2^-112
````
双精度数是64位内存，其中：
* 符号 - 1位
* 指数 - 11位
* 尾数52位

结果编号我们可以得到：
````
value = (-1)^sign * (1 + mantissa / 2 ^ 52) * 2 ^ exponent - 1023)
````
扩展精度为80位数，其中：
* 符号 - 1位
* 指数 - 15位
* 尾数 - 112位

阅读更多关于它 - [这里](https://en.wikipedia.org/wiki/Extended_precision)。我们来看简单的例子。

# x87 FPU
x87浮点单元（FPU）提供高性能浮点处理。它支持浮点，整数和压缩的BCD整数数据类型和浮点处理算法。x87提供以下说明：
* 数据传输指令
* 基本算术指令
* 比较指令
* 超验指令
* 加载常数指令
* x87 FPU控制指令

当然，我们不会看到x87提供的所有指令，有关其他信息，请参阅64-ia-32-architecture-software-developer-vol-1手册第8章。有几个数据传输指令：
* FDL - 加载浮点数
* FST - 存储浮点数（在ST（0）寄存器中）
* FSTP - 存储浮点和弹出（在ST（0）寄存器中）

算术指令：
* FADD - 添加浮点数
* FIADD - 将整数添加到浮点
* FSUB - 减去浮点数
* FISUB - 从浮点中减去整数
* FABS - 获得绝对的价值
* FIMUL - 乘整数和浮点数
* FIDIV - 设备整数和浮点数

等等... FPU有八个10字节寄存器组织在一个环堆栈。堆栈顶部 - 寄存器ST（0），其他寄存器为ST（1），ST（2）... ST（7）。我们通常在使用浮点数据时使用它。
例如：
````s
section .data
    x dw 1.0

fld dword [x]

````
将x的值推送到此堆栈。操作员可以是32位，64位或80位。它像普通堆栈一样工作，如果我们用fld推动另一个值，x值将在ST（1）中，新值将在ST（0）中。FPU指令可以使用这些寄存器，例如：
````s
;;
;; adds st0 value to st3 and saves it in st0
;;
fadd st0, st3

;;
;; adds x and y and saves it in st0
;;
fld dword [x]
fld dword [y]
fadd
````
我们来看简单的例子。我们将有圆半径并计算圆平方并打印：
````s
extern printResult

section .data
		radius    dq  1.7
		result    dq  0

		SYS_EXIT  equ 60
		EXIT_CODE equ 0

global _start
section .text

_start:
		fld qword [radius]
		fld qword [radius]
		fmul

		fldpi
		fmul
		fstp qword [result]

		mov rax, 0
		movq xmm0, [result]
		call printResult

		mov rax, SYS_EXIT
		mov rdi, EXIT_CODE
		syscall
````
我们试着去了解它的工作原理：首先，我们将使用预定义的半径数据和结果存储结果的数据部分。之后这2个常量用于调用exit系统调用。接下来我们看到程序的入口点 - \_start。我们用fld指令在st0和st1中存储半径值，并将这两个值与fmul指令相乘。在这个操作之后，我们将在st0寄存器中得到半径半径乘积的结果。接下来，我们将带有fldpi指令的数字π加载到st0寄存器，之后它的radius * radius值将在st1寄存器中。在st0（pi）和st1（半径* radius的值）上执行与fmul执行乘法运算后，结果将在st0寄存器中。好的，现在我们在st0寄存器中有圆形正方形，可以用fstp指令将其解析成结果。下一步是将结果传递给C函数并调用它。记住我们以前的博客文章中的汇编代码调用C函数。我们需要知道x86_64调用约定。以通常的方式，我们通过寄存器rdi（arg1），rsi（arg2）等来传递函数参数，但这里是浮点数据。有特殊寄存器：smm提供的xmm0 - xmm15。首先，我们需要将xmmN寄存器的数量放在rax寄存器（0为我们的情况），并将结果存入xmm0寄存器。现在我们可以调用C函数来打印结果：首先，我们需要将xmmN寄存器的数量放在rax寄存器（0为我们的情况），并将结果存入xmm0寄存器。现在我们可以调用C函数来打印结果：首先，我们需要将xmmN寄存器的数量放在rax寄存器（0为我们的情况），并将结果存入xmm0寄存器。现在我们可以调用C函数来打印结果：
````c
#include <stdio.h>

extern int printResult(double result);

int printResult(double result) {
	printf("Circle radius is - %f\n", result);
	return 0;
}
````
我们可以用来编译：
````
$ gcc  -g -c circle_fpu_87c.c -o c.o
$ nasm -f elf64 circle_fpu_87.asm -o circle_fpu_87.o
$ ld   -dynamic-linker /lib64/ld-linux-x86-64.so.2 -lc circle_fpu_87.o  c.o -o testFloat1
````
这篇我实在看不懂，应该是浮点数比较复杂吧，所以文章大部分是机器直译。。。
翻译 https://0xax.github.io/asm_8
已加入我的repo https://github.com/ejunjsh/asm-code