---
title: 我的汇编学习之路（4）
date: 2016-5-15 22:24:11
tags: [asm,nasm]
categories: 汇编
---
前段时间，我开始为x86_64编写有关汇编编程的一系列博文。你可以通过asm标签找到它。不幸的是，我上次很忙，没有新帖，所以今天我继续写关于汇编的帖子，并且每周都会尝试这样做。

今天我们将看一下字符串和一些字符串操作。我们还是使用nasm汇编器和linux x86_64。

# 反转字符串
当然，当我们谈论汇编编程语言时，我们不能谈论字符串数据类型，实际上我们正在处理字节数组。我们尝试写简单的例子，我们将定义字符串数据，并尝试反转并将结果写入stdout。当我们开始学习新的编程语言时，这个任务似乎很简单和受欢迎。我们来看看实现。

首先，我定义初始化的数据。它将被放置在数据部分（您可以部分阅读有关章节）：
````s
section .data
		SYS_WRITE equ 1
		STD_OUT   equ 1
		SYS_EXIT  equ 60
		EXIT_CODE equ 0

		NEW_LINE db 0xa
		INPUT db "Hello world!"
````
<!-- more -->
这里我们可以看到四个常数：
* `SYS_WRITE` - 'write'系统调用号码
* `STD_OUT` - stdout文件描述符
* `SYS_EXIT` - 'exit'系统调用号码
* `EXIT_CODE` - 退出代码

系统调用列表你可以[这里](http://blog.rchapman.org/post/36801038863/linux-system-call-table-for-x86-64)找到。我们还定义了：
* `NEW_LINE` - 新行（\ n）符号
* `INPUT` - 我们的输入字符串，我们将反转

接下来我们为我们的缓冲区定义bss部分，我们将放置颠倒后的字符串：
````s
section .bss
		OUTPUT resb 12
````
好的，我们有一些数据和在哪里放置结果的缓冲区，现在我们可以定义代码的文本部分。我们从main _start代码开始：
````s
_start:
		mov rsi, INPUT
		xor rcx, rcx
		cld
		mov rdi, $ + 15
		call calculateStrLength
		xor rax, rax
		xor rdi, rdi
		jmp reverseStr
````
这里有一些新事物。让我们看看它是如何工作的：首先我们把INPUT地址放在第2行的rsi寄存器中，就像我们写入stdout并将零写入rcx寄存器一样，它将用于计算字符串的长度。在第4行，我们可以看到cld运算符。它将df标志重置为零。我们需要它，因为当我们计算字符串的长度时，我们将通过这个字符串的符号，如果df标志为0，我们将处理从左到右的字符串的符号。接下来我们称之为calculateStrLength函数。我先跳过第5行`mov rdi，$ + 15`指令，我稍后会介绍一下。现在我们来看看calculateStrLength的实现：
````s
calculateStrLength:
		;; check is it end of string
		cmp byte [rsi], 0
		;; if yes exit from function
		je exitFromRoutine
		;; load byte from rsi to al and inc rsi
		lodsb
		;; push symbol to stack
		push rax
		;; increase counter
		inc rcx
		;; loop again
		jmp calculateStrLength
````
正如您可以通过它的名称理解的，它只是计算INPUT字符串的长度并将结果存储在rcx寄存器中。首先我们检查一下rsi寄存器不指向零，如果是这样，这是字符串的结尾，我们可以退出函数。接下来是lodsb指令。很简单，只需将1个字节设置为al寄存器（16位ax的低位）并更改rsi指针。当我们执行cld指令时，lodsb每次都会将rsi从左到右移动到一个字节，所以我们将通过字符串符号移动。之后，我们将rax值推送到栈，现在它包含我们的字符串中的符号（lodsb将字节从si到al，al是低8位的rax）。为什么我们把符号推到栈？你必须记住堆栈的工作原理，它的工作原理是LIFO（后进先出）。对我们来说非常好 我们将采取第一个符号从si，推到堆叠，比第二个等等。所以在堆栈顶部会有最后一个字符串符号。比起我们只是按照符号从栈中弹出符号并写入OUTPUT缓冲区。之后，我们增加我们的计数器（rcx）并再次循环到例程的开始。

好的，我们将所有符号从字符串推到堆栈，现在我们可以跳到exitFromRoutine返回到_start那里。怎么做？我们已经为此做了退休指导。但是如果代码将是这样的：
````s
exitFromRoutine:
		;; return to _start
		ret
````
> to be continue...