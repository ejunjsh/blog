---
title: linux命令-gcc
date: 2016-12-27 11:31:23
tags: [linux,gcc,ldd,静态库,动态库]
categories: linux命令
---
> 备忘，mark 👿

gcc命令使用GNU推出的基于C/C++的编译器，是开放源代码领域应用最广泛的编译器，具有功能强大，编译代码支持性能优化等特点。现在很多程序员都应用GCC，怎样才能更好的应用GCC。目前，GCC可以用来编译C/C++、FORTRAN、JAVA、OBJC、ADA等语言的程序，可根据需要选择安装支持的语言。

# 语法
````shell
gcc(选项)(参数)
````
## 选项
````shell
-o：指定生成的输出文件；
-E：仅执行编译预处理；
-S：将C代码转换为汇编代码；
-wall：显示警告信息；
-c：仅执行编译操作，不进行连接操作。
````
## 参数
C源文件：指定C语言源代码文件。
<!-- more -->

# 常用编译命令选项
假设源程序文件名为test.c

## 无选项编译链接
````shell
gcc test.c
````
将test.c预处理、汇编、编译并链接形成可执行文件。这里未指定输出文件，默认输出为a.out。

## 选项 -o
````shell
gcc test.c -o test
````
将test.c预处理、汇编、编译并链接形成可执行文件test。-o选项用来指定输出文件的文件名。

## 选项 -E
````shell
gcc -E test.c -o test.i
````
将test.c预处理输出test.i文件。

## 选项 -S
````shell
gcc -S test.i 
````
将预处理输出文件test.i汇编成test.s文件。

## 选项 -c
````shell
gcc -c test.s
````
将汇编输出文件test.s编译输出test.o文件。

## 无选项链接
````shell
gcc test.o -o test
````
将编译输出文件test.o链接成最终可执行文件test。

## 选项 -O
````shell
gcc -O1 test.c -o test
````
使用编译优化级别1编译程序。级别为1~3，级别越大优化效果越好，但编译时间越长。

# 多源文件的编译方法
如果有多个源文件，基本上有两种编译方法：

假设有两个源文件为test.c和testfun.c

## 多个文件一起编译
````shell
gcc testfun.c test.c -o test
````
将testfun.c和test.c分别编译后链接成test可执行文件。

## 分别编译各个源文件，之后对编译后输出的目标文件链接。
````shell
gcc -c testfun.c    #将testfun.c编译成testfun.o
gcc -c test.c       #将test.c编译成test.o
gcc -o testfun.o test.o -o test    #将testfun.o和test.o链接成test
````

以上两种方法相比较，第一中方法编译时需要所有文件重新编译，而第二种方法可以只重新编译修改的文件，未修改的文件不用重新编译。

# 其他选项

## -l参数和-L参数

-l参数 就是用来指定程序要链接的库，-l参数紧接着就是库名，那么库名跟真正的库文件名有什么关系呢？就拿数学库来说，他的库名是m，他的库文件名是libm.so，很容易看出，把库文件名的头lib和尾.so去掉就是库名了。

好了现在我们知道怎么得到库名了，比如我们自已要用到一个第三方提供的库名字叫libtest.so，那么我们只要把libtest.so拷贝到/usr/lib里，编译时加上-ltest参数，我们就能用上libtest.so库了（当然要用libtest.so库里的函数，我们还需要与libtest.so配套的头文件）。

放在/lib和/usr/lib和/usr/local/lib里的库直接用-l参数就能链接了，但如果库文件没放在这三个目录里，而是放在其他目录里，这时我们只用-l参数的话，链接还是会出错，出错信息大概是：“/usr/bin/ld: cannot find -lxxx”，也就是链接程序ld在那3个目录里找不到libxxx.so，这时另外一个参数-L就派上用场了，比如常用的X11的库，它放在/usr/X11R6/lib目录下，我们编译时就要用-L/usr/X11R6/lib -lX11参数，-L参数跟着的是库文件所在的目录名。再比如我们把libtest.so放在/aaa/bbb/ccc目录下，那链接参数就是-L/aaa/bbb/ccc -ltest

另外，大部分libxxxx.so只是一个链接，以RH9为例，比如libm.so它链接到/lib/libm.so.x，/lib/libm.so.6又链接到/lib/libm-2.3.2.so，如果没有这样的链接，还是会出错，因为ld只会找libxxxx.so，所以如果你要用到xxxx库，而只有libxxxx.so.x或者libxxxx-x.x.x.so，做一个链接就可以了ln -s libxxxx-x.x.x.so libxxxx.so

## -include和-I参数

-include用来包含头文件，但一般情况下包含头文件都在源码里用#include xxxxxx实现，-include参数很少用。-I参数是用来指定头文件目录，/usr/include目录一般是不用指定的，gcc知道去那里找，但是如果头文件不在/usr/include里我们就要用-I参数指定了，比如头文件放在/myinclude目录里，那编译命令行就要加上-I/myinclude参数了，如果不加你会得到一个”xxxx.h: No such file or directory”的错误。-I参数可以用相对路径，比如头文件在当前目录，可以用-I.来指定。

## 其他选项简介
-Dname=definition... 在命令行上定义宏,有两种方式,-Dname或者-Dname=definition.在命令行上设置宏定义的目的主要是为了在调试的时候设定一些开关, 而在发布的时候再关闭或者打开这些开关即可,当然宏定义也用来对代码进行有选择地编译.另外也还有其他的一些作用.

-shared参数 编译动态库时要用到，比如gcc -shared test.c -o libtest.so

-ansi 只支持 ANSI 标准的 C 语法。这一选项将禁止 GNU C 的某些特色，例如 asm 或 typeof 关键词。

-g 生成调试信息。GNU 调试器可利用该信息。

-static 禁止使用共享连接。

-w 不生成任何警告信息。

-Wall 生成所有警告信息。

-pthread  通过pthreads库加入对多线程的支持,这为预处理和连接设置了标志.pthread是POSIX指定的标准线程库.

# 相关环境变量

CFLAGS 表示用于 C 编译器的选项，
CXXFLAGS 表示用于 C++ 编译器的选项。
这两个变量实际上涵盖了编译和汇编两个步骤。

CFLAGS： 指定头文件（.h文件）的路径，如：CFLAGS=-I/usr/include -I/path/include。同样地，安装一个包时会在安装路径下建立一个include目录，当安装过程中出现问题时，试着把以前安装的包的include目录加入到该变量中来。

LDFLAGS：gcc 等编译器会用到的一些优化参数，也可以在里面指定库文件的位置。用法：LDFLAGS=-L/usr/lib -L/path/to/your/lib。每安装一个包都几乎一定的会在安装目录里建立一个lib目录。如果明明安装了某个包，而安装另一个包时，它愣是说找不到，可以抒那个包的lib路径加入的LDFALGS中试一下。

LIBS：告诉链接器要链接哪些库文件，如LIBS = -lpthread -liconv

简单地说，LDFLAGS是告诉链接器从哪里寻找库文件，而LIBS是告诉链接器要链接哪些库文件。不过使用时链接阶段这两个参数都会加上，所以你即使将这两个的值互换，也没有问题。

有时候LDFLAGS指定-L虽然能让链接器找到库进行链接，但是运行时链接器却找不到这个库，如果要让软件运行时库文件的路径也得到扩展，那么我们需要增加这两个库给"-Wl,R"：

````
LDFLAGS = -L/var/xxx/lib -L/opt/mysql/lib -Wl,R/var/xxx/lib -Wl,R/opt/mysql/lib
````

如果在执行./configure以前设置环境变量export LDFLAGS="-L/var/xxx/lib -L/opt/mysql/lib -Wl,R/var/xxx/lib -Wl,R/opt/mysql/lib" ，注意设置环境变量等号两边不可以有空格，而且要加上引号（shell的用法）。那么执行configure以后，Makefile将会设置这个选项，链接时会有这个参数，编译出来的可执行程序的库文件搜索路径就得到扩展了。

LIBRARY_PATH环境变量用于在程序编译期间查找动态链接库时指定查找共享库的路径，例如，指定gcc编译需要用到的动态链接库的目录。设置方法如下（其中，LIBDIR1和LIBDIR2为两个库目录）：

````
export LIBRARY_PATH=LIBDIR1:LIBDIR2:$LIBRARY_PATH
````

LD_LIBRARY_PATH环境变量用于在程序加载运行期间查找动态链接库时指定除了系统默认路径之外的其他路径，注意，LD_LIBRARY_PATH中指定的路径会在系统默认路径之前进行查找。设置方法如下（其中，LIBDIR1和LIBDIR2为两个库目录）：
````
export LD_LIBRARY_PATH=LIBDIR1:LIBDIR2:$LD_LIBRARY_PATH
````

上面两条环境变量的区别和使用

开发时，设置LIBRARY_PATH，以便gcc能够找到编译时需要的动态链接库。

发布时，设置LD_LIBRARY_PATH，以便程序加载运行时能够自动找到需要的动态链接库。

# ldd命令

> 👿 说到动态库，顺便说说这个命令吧

`ldd(选项)(可执行文件)` 用于打印程序或者库文件所依赖的共享库列表。

````
--version：打印指令版本号；
-v：详细信息模式，打印所有相关信息；
-u：打印未使用的直接依赖；
-d：执行重定位和报告任何丢失的对象；
-r：执行数据对象和函数的重定位，并且报告任何丢失的对象和函数；
--help：显示帮助信息。
````

首先ldd不是一个可执行程序，而只是一个shell脚本

ldd能够显示可执行模块的dependency，其原理是通过设置一系列的环境变量，如下：LD_TRACE_LOADED_OBJECTS、LD_WARN、LD_BIND_NOW、LD_LIBRARY_VERSION、LD_VERBOSE等。当LD_TRACE_LOADED_OBJECTS环境变量不为空时，任何可执行程序在运行时，它都会只显示模块的dependency，而程序并不真正执行。要不你可以在shell终端测试一下，如下：

````
export LD_TRACE_LOADED_OBJECTS=1
````

再执行任何的程序，如ls等，看看程序的运行结果。

ldd显示可执行模块的dependency的工作原理，其实质是通过ld-linux.so（elf动态库的装载器）来实现的。我们知道，ld-linux.so模块会先于executable模块程序工作，并获得控制权，因此当上述的那些环境变量被设置时，ld-linux.so选择了显示可执行模块的dependency。

实际上可以直接执行ld-linux.so模块，如：`/lib/ld-linux.so.2 --list program`（这相当于ldd program）
