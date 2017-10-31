---
title: 我的汇编学习之路（6）
date: 2016-5-17 11:23:44
tags: [asm,nasm]
categories: 汇编
---
这是系列的第六篇，接下来要讲一下AT&T汇编语法。之前一直都是用nasm汇编，但是还有其他不同语法的汇编，如fasm，yasm等等。正如我上面写的，我们要看一下gas（GNU汇编）和讲一下与nasm的不同。GCC用的是GNU汇编，所以你会看到一个简单的hello world的汇编输出：
````c
#include <unistd.h>

int main(void) {
	write(1, "Hello World\n", 15);
	return 0;
}
````
汇编输出：
````s
	.file	"test.c"
	.section	.rodata
.LC0:
	.string	"Hello World\n"
	.text
	.globl	main
	.type	main, @function
main:
.LFB0:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	$15, %edx
	movl	$.LC0, %esi
	movl	$1, %edi
	call	write
	movl	$0, %eax
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	main, .-main
	.ident	"GCC: (Ubuntu 4.9.1-16ubuntu6) 4.9.1"
	.section	.note.GNU-stack,"",@progbits
````
看上去跟nasm汇编不同吧，让我们看看他们的不同吧。
<!-- more -->
# AT&T语法
## 段（Sections）
一般写汇编都要从段开始，先看一个简单例子：
````s
.data
    //
    // initialized data definition
    //
.text
    .global _start

_start:
    //
    // main routine
    //
````
你可能一句注意到了两个不同：
* 段定义以.符号开始
* 主函数定义以.global开头，而nasm是以global开头

还有gas用其他指令来定义数据：
````s
.section .data
    // 1 byte
    var1: .byte 10
    // 2 byte
    var2: .word 10
    // 4 byte
    var3: .int 10
    // 8 byte
    var4: .quad 10
    // 16 byte
    var5: .octa 10

    // assembles each string (with no automatic trailing zero byte) into consecutive addresses
    str1: .asci "Hello world"
    // just like .ascii, but each string is followed by a zero byte
    str2: .asciz "Hello world"
    // Copy the characters in str to the object file
    str3: .string "Hello world"

````
一般在nasm中用下面的顺序来操作数据的：
````s
mov destination, source
````
但是在GNU汇编中顺序是反的：
````s
mov source, destination
````
举个例子：
````s
;;
;; nasm syntax
;;
mov rax, rcx

//
// gas syntax
//
mov %rcx, %rax
````
这里的寄存器要以%开头，如果你用字面量的话要用$开头：
````s
movb $10, %rax
````
## 操作数的大小和操作语法
有时候我们只需要一部分数据，例如64寄存器的第一个字节，我们用下面的语法：
````s
mov ax, word [rsi]
````
同样的用法在gas，是不用在操作数上定义大小，而是在指令上：
````s
movw (%rsi), %rax
````
GNU汇编在操作上有6个后缀：
* b - 1 字节操作
* w - 2 字节操作
* l - 4 字节操作
* q - 8 字节操作
* t - 10 字节操作
* o - 16 字节操作

这条规则不但用在mov指令上，还可以用在如addl, xorb, cmpw 等指令上

## 内存访问
您可以注意到，在nasm示例中，我们在前面的例子中使用（）括号代替[]。在GAS上，要获得一个内存地址指向的值，请用括号：（％rax），例如：
````s
movq -8(%rbp),%rdi
movq 8(%rbp),%rdi
````

## 跳转
GNU汇编器支持以下操作符，用于远程函数调用和跳转：
````s
lcall $section, $offset
````
远程跳转 - 跳转到位于与当前代码段不同的段，但处于相同权限级别的指令，有时称为段间跳转。

## 注释
GNU汇编器支持3种类型的注释：
````s
 # - single line comments
// - single line comments
/* */ - for multiline comments
````

翻译 https://0xax.github.io/asm_6/