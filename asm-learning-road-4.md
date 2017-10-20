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
正如您可以通过它的名称理解的，它只是计算INPUT字符串的长度并将结果存储在rcx寄存器中。首先我们检查一下rsi寄存器不指向零，如果是这样，这是字符串的结尾，我们可以退出函数。接下来是lodsb指令。很简单，只需将1个字节设置为al寄存器（16位ax的低位）并更改rsi指针。当我们执行cld指令时，lodsb每次都会将rsi从左到右移动到一个字节，所以我们将通过字符串符号移动。之后，我们将rax值推送到栈，现在rax包含我们的字符串中的符号（lodsb将字节从rsi到al，al是低8位的rax）。为什么我们把符号推到栈？你必须记住堆栈的工作原理，它的工作原理是LIFO（后进先出）。对我们来说非常好 我们将采取第一个符号从si，推到栈，接下来第二个，等等。所以在栈顶部会有最后一个字符串符号。接下来我们从栈里弹出字符并写入OUTPUT缓冲区。之后，我们增加我们的计数器（rcx）并再次循环到`calculateStrLength`的开始。

好的，我们将所有符号从字符串推到堆栈，现在我们可以跳到exitFromRoutine返回到_start那里。怎么做？我们已经为此做了退休指导。但是如果代码将是这样的：
````s
exitFromRoutine:
		;; return to _start
		ret
````
不起作用。为什么？这很棘手 记住我们在_start中调用calculateStrLength。当我们调用函数时会发生什么？首先功能的参数从右到左推动堆栈。之后它返回地址推送到堆栈。所以函数将在执行结束后知道在哪里返回。但是看看calculateStrLength，我们将符号从我们的字符串推到堆栈，现在没有堆栈顶部的返回地址，函数不知道在哪里返回。如何与它一起 现在我们来看看这个奇怪的指令：
````s
mov rdi, $ + 15
````
首先：
* $ - 返回定义$字符串时候的内存地址
* $$ - 返回start块的内存地址

所以我们有mov rdi的位置，$ + 15，但是为什么我们在这里添加15？看，我们需要在calculateStrLength之后知道下一行的位置。我们用objdump util打开我们的文件：
````
objdump -D reverse

reverse:     file format elf64-x86-64

Disassembly of section .text:

00000000004000b0 <_start>:
  4000b0:	48 be 41 01 60 00 00 	movabs $0x600141,%rsi
  4000b7:	00 00 00
  4000ba:	48 31 c9             	xor    %rcx,%rcx
  4000bd:	fc                   	cld
  4000be:	48 bf cd 00 40 00 00 	movabs $0x4000cd,%rdi
  4000c5:	00 00 00
  4000c8:	e8 08 00 00 00       	callq  4000d5 <calculateStrLength>
  4000cd:	48 31 c0             	xor    %rax,%rax
  4000d0:	48 31 ff             	xor    %rdi,%rdi
  4000d3:	eb 0e                	jmp    4000e3 <reverseStr>
````
我们可以看到，第12行（我们的mov rdi，$ + 15）需要10个字节，函数调用在16行，用5字节，因此需要15个字节。这就是为什么我们的返回地址将是`mov rdi $+15`.现在我们可以将返回地址从rdi推送到堆栈并从函数返回：
````s
exitFromRoutine:
		;; push return addres to stack again
		push rdi
		;; return to _start
		ret
````
现在我们回到start区域，在调用`calculateStrLength`之后，我们对rax和rdi置零，然后跳到`reverseStr`标签,它的实现如下：
````s
reverseStr:
		cmp rcx, 0
		je printResult
		pop rax
		mov [OUTPUT + rdi], rax
		dec rcx
		inc rdi
		jmp reverseStr
````
这里我们检查我们的计数器，它是字符串的长度，如果它是零，我们将所有符号写入缓冲区并可以打印。如果不等于0，我们从堆栈弹出第一个符号到rax寄存器，并将其写入OUTPUT缓冲区。我们添加rdi，因为我们将写入符号到缓冲区的第一个字节。之后，我们增加用于移动OUTPUT缓冲区的rdi，减少长度计数器并跳转到标签的开头。

在执行reverseStr之后，我们在OUTPUT缓冲区有了反转后字符串，并可以用新行连同结果写入stdout：
````s
printResult:
		mov rdx, rdi
		mov rax, 1
		mov rdi, 1
		mov rsi, OUTPUT
        syscall
		jmp printNewLine

printNewLine:
		mov rax, SYS_WRITE
		mov rdi, STD_OUT
		mov rsi, NEW_LINE
		mov rdx, 1
		syscall
		jmp exit
````
退出程序:
````s
exit:
		mov rax, SYS_EXIT
		mov rdi, EXIT_CODE
		syscall
````
就是这些啦

# 字符串操作
当然还有许多其他的字符串/字节操作指令：
* REP - 重复，而rcx不为零
* MOVSB - 复制字符串（MOVSW，MOVSD等）
* CMPSB - 字节串比较
* SCASB - 字节串扫描
* STOSB - 将字节写入字符串

翻译 https://0xax.github.io/asm_4/
已加入我的repo https://github.com/ejunjsh/asm-code