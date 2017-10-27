---
title: 我的汇编学习之路（5）
date: 2016-5-16 15:29:39
tags: [asm,nasm]
categories: 汇编
---
这是关于x86\_64汇编的第五章，这一章主要说宏。
# 宏
NASM支持两种形式的宏：
* 单行
* 多行

所有单行宏必须从％define指令开始。格式如下：
````s
%define macro_name(parameter) value
````
Nasm宏的行为和外观与C类似。例如，我们可以创建以下单行宏：
````s
%define argc rsp + 8
%define cliArg1 rsp + 24
````
<!-- more -->
在代码中使用它：
````s
;;
;; argc will be expanded to rsp + 8
;;
mov rax, [argc]
cmp rax, 3
jne .mustBe3args
````
多行宏以％macro指令开始，并以％endmacro结尾。一般形式如下：
````s
%macro number_of_parameters
    instruction
    instruction
    instruction
%endmacro
````
例如：
````s
%macro bootstrap 1
          push ebp
          mov ebp,esp
%endmacro
````
我们可以用它：
````s
_start:
    bootstrap
````
例如我们来看看PRINT宏：
````s
%macro PRINT 1
    pusha
    pushf
    jmp %%astr
%%str db %1, 0
%%strln equ $-%%str
%%astr: _syscall_write %%str, %%strln
popf
popa
%endmacro

%macro _syscall_write 2
	mov rax, 1
        mov rdi, 1
        mov rsi, %%str
        mov rdx, %%strln
        syscall
%endmacro
````
我们来试试看宏，了解它的工作原理：在第一行我们用一个参数定义了PRINT宏。比推送所有通用寄存器（用pusha指令）和标志寄存器（用pushf指令）。之后，我们跳到%% astr标签。注意宏中定义的所有标签必须以%%开头。现在我们转到\_\_syscall\_write具有2参数的宏。我们来看\_\_syscall\_write的实现。其实就是之前博客用到的将字符串写入系统调用打印到stdout的代码。看起来像这样：
````s
;; write syscall number
mov rax, 1
;; file descriptor, standard output
mov rdi, 1
;; message address
mov rsi, msg
;; length of message
mov rdx, 14
;; call write syscall
syscall
````
在我们的\_\_syscall\_write宏中，我们定义了前两个指令，用于将1置为rax（写系统调用号）和rdi（stdout文件描述符）。然后我们把%%str放到rsi寄存器（指向字符串的指针），其中%%str是PRINT宏的第一个参数（用$parameter_number的宏参数访问）并以0结尾的本地标签（每个字符串必须以零结束）。而%%strln则计算字符串长度。之后，我们使用syscall指令调用系统调用，这就是所有。
现在我们可以用它：
````s
label: PRINT "Hello World!"
````

# 有用的标准宏
NASM支持以下标准宏：
## STRUC
我们可以使用`STRUC`和`ENDSTRUC`定义数据结构。例如：
````s
struc person
   name: resb 10
   age:  resb 1
endstruc
````
接下来我们就能创建一个数据结构实例：
````s
section .data
    p: istruc person
      at name db "name"
      at age  db 25
    iend

section .text
_start:
    mov rax, [p + person.name]
````

## %include
我们可以包括其他汇编文件，并跳转到带有％include指令的标签或调用函数。

翻译 https://0xax.github.io/asm_5/
文中的PRINT宏是有问题的,所以参考看看就好
修正的代码见我的repo https://github.com/ejunjsh/asm-code 