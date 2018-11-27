---
title: 说说c的volatile
date: 2018-11-27 21:07:59
tags: [c,volatile]
categories: c
---

# volatile 介绍

表示一个变量也许会被后台程序改变，关键字 volatile 是与 const 绝对对立的。它指示一个变量也许会被某种方式修改，这种方式按照正常程序流程分析是无法预知的（例如，一个变量也许会被一个中断服务程序所修改）。

变量如果加了 volatile 修饰，则会从内存重新装载内容，而不是直接从寄存器拷贝内容。 

<!-- more -->

# 为什么使用 volatile

volatile 的作用 是作为指令关键字，确保本条指令不会因编译器的优化而省略，且要求每次直接读值。

现在考虑一个问题，编译器如何对代码进行优化的？

我们看一个例子：

````c
//示例一
#include <stdio.h>
int main (void)
{
	int i = 10;
	int a = i; //优化
	int b = i;
 
	printf ("i = %d\n", b);
	return 0;
}
````

````s
//编译优化、查看汇编
gcc -O2 -S test.c 
cat test.s 
 
	.file	"test.c"
	.section	.rodata.str1.1,"aMS",@progbits,1
.LC0:
	.string	"i = %d\n"
	.section	.text.startup,"ax",@progbits
	.p2align 4,,15
	.globl	main
	.type	main, @function
main:
.LFB22:
	.cfi_startproc
	pushl	%ebp
	.cfi_def_cfa_offset 8
	.cfi_offset 5, -8
	movl	%esp, %ebp
	.cfi_def_cfa_register 5
	andl	$-16, %esp
	subl	$16, %esp
	movl	$10, 8(%esp)
	movl	$.LC0, 4(%esp)
	movl	$1, (%esp)
	call	__printf_chk
	xorl	%eax, %eax
	leave
	.cfi_restore 5
	.cfi_def_cfa 4, 4
	ret
	.cfi_endproc
.LFE22:
	.size	main, .-main
	.ident	"GCC: (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3"
	.section	.note.GNU-stack,"",@progbits
````

````c
//示例二
#include <stdio.h>
int main (void)
{
	volatile int i = 10;
	int a = i; //未优化
	int b = i;
 
	printf ("i = %d\n", b);
	return 0;
}
````

````s
//编译优化、查看汇编
gcc -O2 -S test.c 
cat test.s 
 
	.file	"test.c"
	.section	.rodata.str1.1,"aMS",@progbits,1
.LC0:
	.string	"i = %d\n"
	.section	.text.startup,"ax",@progbits
	.p2align 4,,15
	.globl	main
	.type	main, @function
main:
.LFB22:
	.cfi_startproc
	pushl	%ebp
	.cfi_def_cfa_offset 8
	.cfi_offset 5, -8
	movl	%esp, %ebp
	.cfi_def_cfa_register 5
	andl	$-16, %esp
	subl	$32, %esp
	movl	$10, 28(%esp)
	movl	28(%esp), %eax
	movl	28(%esp), %eax
	movl	$.LC0, 4(%esp)
	movl	$1, (%esp)
	movl	%eax, 8(%esp)
	call	__printf_chk
	xorl	%eax, %eax
	leave
	.cfi_restore 5
	.cfi_def_cfa 4, 4
	ret
	.cfi_endproc
.LFE22:
	.size	main, .-main
	.ident	"GCC: (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3"
	.section	.note.GNU-stack,"",@progbits
````

比较：

[![](http://idiotsky.top/images3/c-volatile-1.png)](http://idiotsky.top/images3/c-volatile-1.png)

可以清楚的看到：使用 volatile 的代码编译未优化。

volatile 指出 i 是随时可能发生变化的，每次使用它的时候必须从 i的地址中读取，因而编译器生成的汇编代码会重新从i的地址读取数据放在 b 中。而优化做法是，由于编译器发现两次从 i读数据的代码之间的代码没有对 i 进行过操作，它会自动把上次读的数据放在 b 中。而不是重新从 i 里面读。

上面例子还没有看到影响，下面这个例子可以说明volatile的作用

一个中断服务子程序中会访问到的非自动变量（Non-automatic variables)

由于访问寄存器的速度要快过RAM，所以编译器一般都会作减少存取外部RAM的优化，例如：

````c
static int i=0; //i 为非自动变量
int main(void)
{
     ...
     while (1){
if (i) dosomething();
}
｝
/* Interrupt service routine. */
void ISR_2(void)
{
      i=1;
}
````

程序的本意是希望 ISR_2 中断产生时，在main函数中调用 dosomething 函数，但是，由于编译器判断在 main 函数里面没有修改过 i，因此可能只执行一次对从i到某寄存器的读操作，然后每次if判断都只使用这个寄存器里面的“i副本”，导致 dosomething 永远也不会被调用。如果将变量加上 volatile 修饰，则编译器保证对此变量的读写操作都不会被优化（肯定执行）。此例中i也应该如此说明。

接下来看看多线程的场景

多线程应用中被几个任务共享的变量

当两个线程都要用到某一个变量且该变量的值会被改变时，应该用 volatile 声明，该关键字的作用是防止优化编译器把变量从内存装入CPU寄存器中。如果变量被装入寄存器，那么两个线程有可能一个使用内存中的变量，一个使用寄存器中的变量，这会造成程序的错误执行。volatile的意思是让编译器每次操作该变量时一定要从内存中真正取出，而不是使用已经存在寄存器中的值，如下：

````c
volatile  BOOL  bStop  =  FALSE;  //bStop  为共享全局变量
//在一个线程中：  
  while(  !bStop  )  {  ...  }  
  bStop  =  FALSE;  
  return;    
 
//在另外一个线程中，要终止上面的线程循环：  
  bStop  =  TRUE;  
  while(  bStop  ); 
````

等待上面的线程终止，如果bStop不使用volatile申明，那么这个循环将是一个死循环，因为bStop已经读取到了寄存器中，寄存器中bStop的值永远不会变成FALSE，加上volatile，程序在执行时，每次均从内存中读出bStop的值，就不会死循环了。

# 总结

volatile 关键字是一种类型修饰符，用它声明的类型变量表示可以被某些编译器未知的因素更改。volatile 提醒编译器它后面所定义的变量随时都有可能改变，因此编译后的程序每次需要存储或读取这个变量的时候，都会直接从变量地址中读取数据。如果没有 volatile 关键字，则编译器可能优化读取和存储，可能暂时使用寄存器中的值，如果这个变量由别的程序更新了的话，将出现不一致的现象。所以遇到这个关键字声明的变量，编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问。

# 参考

https://blog.csdn.net/qq_29350001/article/details/54024070