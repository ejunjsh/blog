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
这里我们能看到我们把1放到rax，2放到rdx，之后用这些寄存器依次放入到栈。栈是LIFO（后进先出）。所以这个栈或者应用程序将是下面这个结构：
[![](http://idiotsky.top/images1/asm-learning-road-3-1.png)](http://idiotsky.top/images1/asm-learning-road-3-1.png)
然后我们从堆栈中复制具有地址rsp + 8的值。这意味着我们获取堆栈顶部的地址，加上8并将该地址的值复制到rax。之后rax值将为1。

# 例子
我们来看一个例子。我们将编写简单的程序，这将得到两个命令行参数。将得到两个参数总和并打印结果。
````s
section .data
		SYS_WRITE equ 1
		STD_IN    equ 1
		SYS_EXIT  equ 60
		EXIT_CODE equ 0

		NEW_LINE   db 0xa
		WRONG_ARGC db "Must be two command line argument", 0xa
````
首先我们.data用一些值定义section。这里我们有四个常量用于linux系统调用，sys\_write，sys\_exit等等。还有两个字符串：第一个是新行符号，第二个是错误消息。

我们来看看这个.text部分，它由程序代码组成：
````s
section .text
        global _start

_start:
		pop rcx
		cmp rcx, 3
		jne argcError

		add rsp, 8
		pop rsi
		call str_to_int

		mov r10, rax
		pop rsi
		call str_to_int
		mov r11, rax

		add r10, r11
````
我们来试试看看这里发生了什么：在_start标签第一条指令从栈中获取第一个值并将其放到rcx寄存器之后。如果我们使用命令行参数运行应用程序，则它们将按照以下顺序运行在堆栈中：
````
[rsp] - top of stack will contain arguments count.
[rsp + 8] - will contain argv[0]
[rsp + 16] - will contain argv[1]
and so on...
````
所以我们得到命令行参数count并把它放到rcx。之后，我们比较rcx和3.如果它们不相等，我们跳转到刚刚打印错误消息的argcError标签：
````s
argcError:
    ;; sys_write syscall
    mov     rax, 1
    ;; file descritor, standard output
	mov     rdi, 1
    ;; message address
    mov     rsi, WRONG_ARGC
    ;; length of message
    mov     rdx, 34
    ;; call write syscall
    syscall
    ;; exit from program
	jmp exit
````
当我们有两个参数时，为什么我们与3比较？这很简单。第一个参数是一个程序名，所有这些都是我们传递给程序的命令行参数。好的，如果我们通过了两个命令行参数，我们将在10行旁边。这里我们将rsp移到8，从而缺少第一个参数 - 程序的名称。现在rsp指向我们传递的第一个命令行参数。我们用pop命令获取它，并将其转换为rsi寄存器并调用函数将其转换为整数。接下来我们阅读`str_to_int`实现。在我们的函数结束工作后，我们在rax寄存器中有整数值，并将其保存在r10寄存器中。之后，我们做同样的操作，但用r11。最后，我们在r10和r11寄存器中有两个整数值，现在我们可以通过add命令得到它的总和。现在我们必须将结果转换为字符串并打印。让我们看看如何做：
````s
mov rax, r10
;; number counter
xor r12, r12
;; convert to string
jmp int_to_str
````
这里我们将命令行参数的和加到rax寄存器中，将r12设置为零并跳转到`int_to_str`。好的，现在我们有程序的基础。我们已经知道如何打印字符串，我们有什么打印。我们来看看`str_to_int`和`int_to_str`的实现。
````s
str_to_int:
            xor rax, rax
            mov rcx,  10
next:
	    cmp [rsi], byte 0
	    je return_str
	    mov bl, [rsi]
        sub bl, 48
	    mul rcx
	    add rax, rbx
	    inc rsi
	    jmp next

return_str:
	    ret
````
在`str_to_int`的开始，我们将rax设置为0，将rcx设置为10.然后我们转到下一个标签。从上面的例子（第一次调用`str_to_int`之前的第一行）可以看出，我们把argv [1]放在rsi中。现在我们将rsi的第一个字节与0进行比较，因为每个字符串都以NULL符号结尾，如果我们返回。如果它不是0，我们将它的值复制到一个字节bl寄存器，并从其中减去48。为什么48？所有数字从0到9在asci表中有48到57个代码。所以如果我们从数字符号48（例如从57）减去，我们得到数字。然后我们将rax乘以rcx（其值为10）。之后，我们增加rsi以获得下一个字节并重新循环。例如，如果rsi指向'5''7''6''\ 000'序列，那么将遵循以下步骤：
````
    rax = 0
    get first byte - 5 and put it to rbx
    rax * 10 --> rax = 0 * 10
    rax = rax + rbx = 0 + 5
    Get second byte - 7 and put it to rbx
    rax * 10 --> rax = 5 * 10 = 50
    rax = rax + rbx = 50 + 7 = 57
    and loop it while rsi is not \000
````
`str_to_int`之后，rax保存了数字。现在看看`int_to_str`:
````s
int_to_str:
		mov rdx, 0
		mov rbx, 10
		div rbx
		add rdx, 48
		add rdx, 0x0
		push rdx
		inc r12
		cmp rax, 0x0
		jne int_to_str
		jmp print
````
这里我们把0放到rdx，把10放到rbx。之后我们执行`div rbx`。从上面的代码看到，在`int_to_str`函数调用之前，rax保存了一个整数数字-两个命令行参数之和。执行`div rbx`指令，我们用rbx除以rax，余数存到rdx，整除部分存到rax。接下来rax加48和0x0。在加这个48之后，我们能得到一个数字的asci码，还有所有字符串都要以0x0结尾。之后，保存这个字符到栈中，然后递增一下r12（第一次迭代的时候它是0，在_start时候设置为0），还有就是跟rax比较下是否等于0，如果是0，代表转换整形为字符串函数结束。算法的为代码如下：假设我们有一个数字为23
````
23 / 10. rax = 2; rdx = 3
rdx + 48 = "3"
push "3" to stack
compare rax with 0 if no go again
2 / 10. rax = 0; rdx = 2
rdx + 48 = "2"
push "2" to stack
compare rax with 0, if yes we can finish function execution and we will have "2" "3" ... in stack
````
我们实现了两个有用的功能int\_to\_str，str\_to\_int并将整数转换为字符串，反之亦然。现在我们有两个整数的和，它们被转换成字符串并保存在堆栈中。我们可以打印结果：
````s
print:
	;;;; calculate number length
	mov rax, 1
	mul r12
	mov r12, 8
	mul r12
	mov rdx, rax

	;;;; print sum
	mov rax, SYS_WRITE
	mov rdi, STD_IN
	mov rsi, rsp
	;; call sys_write
	syscall

    jmp exit
````
我们已经知道如何用sys\_writesyscall 打印字符串，但这里是一个有趣的部分。我们必须计算字符串的长度。如果你看看`int_to_str`，你会看到我们每次迭代增加r12寄存器，所以它包含我们要的数字的数量。我们必须还要将它乘与8（因为我们把每个符号推到堆栈），它将是我们需要打印的字符串的长度。之后，我们每次将1放到rax（sys_write number），1到rdi（stdin），字符串长度为rdx，指向堆栈顶部的指针为rsi（字符串开头）。然后结束程序：
````s
exit:
	mov rax, SYS_EXIT
	exit code
	mov rdi, EXIT_CODE
	syscall
````
这就是这篇文章的全部。

翻译 https://0xax.github.io/asm_3/
已加入我的repo https://github.com/ejunjsh/asm-code