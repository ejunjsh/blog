---
title: linux命令-ar
date: 2016-12-29 11:31:23
tags: [linux,ar,gcc,静态库,动态库]
categories: linux命令
---
> 备忘，mark 👿

# 用途说明
创建静态库.a文件。用C/C++开发程序时经常用到，但我很少单独在命令行中使用ar命令，一般写在makefile中，有时也会在shell脚 本中用到。

# 常用参数
格式：ar rcs  libxxx.a xx1.o xx2.o
参数r：在库中插入模块(替换)。当插入的模块名已经在库中存在，则替换同名的模块。如果若干模块中有一个模块在库中不存在，ar显示一个错误消息，并不替换其他同名模块。默认的情况下，新的成员增加在库的结尾处，可以使用其他任选项来改变增加的位置。【1】
参数c：创建一个库。不管库是否存在，都将创建。
参数s：创建目标文件索引，这在创建较大的库时能加快时间。（补充：如果不需要创建索引，可改成大写S参数；如果.a文件缺少索引，可以使用ranlib命令添加）
 
格式：ar t libxxx.a
显示库文件中有哪些目标文件，只显示名称。
 
格式：ar tv libxxx.a
显示库文件中有哪些目标文件，显示文件名、时间、大小等详细信息。
 
格式：nm -s libxxx.a
显示库文件中的索引表。
 
格式：ranlib libxxx.a
为库文件创建索引表。

# 创建并使用静态库

第一步：编辑源文件，test.h test.c main.c。其中main.c文件中包含main函数，作为程序入口；test.c中包含main函数中需要用到的函数。
vi test.h test.c main.c
第二步：将test.c编译成目标文件。
gcc -c test.c
如果test.c无误，就会得到test.o这个目标文件。
第三步：由.o文件创建静态库。
ar rcs libtest.a test.o
第四步：在程序中使用静态库。
gcc -o main main.c -L. -ltest
因为是静态编译，生成的执行文件可以独立于.a文件运行。
第五步：执行。
./main

# 创建并使用动态库

第一步：编辑源文件，test.h test.c main.c。其中main.c文件中包含main函数，作为程序入口；test.c中包含main函数中需要用到的函数。
vi test.h test.c main.c
第二步：将test.c编译成目标文件。
gcc -c test.c
前面两步与创建静态库一致。
第三步：由.o文件创建动态库文件。
gcc -shared -fPIC -o libtest.so test.o
第四步：在程序中使用动态库。
gcc -o main main.c -L. -ltest
当静态库和动态库同名时， gcc命令将优先使用动态库。
第五步：执行。
LD_LIBRARY_PATH=. ./main