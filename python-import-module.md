---
title: python在不同层级目录import模块的方法
date: 2015-02-18 22:32:02
tags: python
categories: python基础
---
> 使用python进行程序编写时，经常会使用第三方模块包。这种包我们可以通过python setup install 进行安装后，通过import XXX或from XXX import yyy 进行导入。不过如果是自己遍写的依赖包，又不想安装到python的相应目录，可以放到本目录里进行import进行调用；为了更清晰的理清程序之间的关系，例如我们会把这种包放到lib目录再调用。本篇就针对常见的模块调用方法汇总下。

<!-- more -->
# 同级目录下的调有
程序结构如下：
````
-- src
    |-- mod1.py
    |-- test1.py
````

若在程序test1.py中导入模块mod1, 则直接使用
````python
import mod1
或
from mod1 import *;
````
# 调用子目录下的模块
程序结构如下：
````
-- src
    |-- mod1.py
    |-- lib
    |    |-- mod2.py
    |-- test1.py
````

这时看到test1.py和lib目录（即mod2.py的父级目录），如果想在程序test1.py中导入模块mod2.py ，可以在lib件夹中建立空文件__init__.py文件(也可以在该文件中自定义输出模块接口)，然后使用：
````python
from lib.mod2 import *
或
import lib.mod2
````

# 调用上级目录下的文件

程序结构如下：
````
-- src
    |-- mod1.py
    |-- lib
    |    |-- mod2.py
    |-- sub
    |    |-- test2.py
````
这里想要实现test2.py调用mod1.py和mod2.py ，做法是我们先跳到src目录下面，直接可以调用mod1，然后在lib上当下建一个空文件__init__.py ，就可以像第二步调用子目录下的模块一样，通过import  lib.mod2进行调用了。具体代码如下：
````python
import sys
sys.path.append("..")
import mod1
import lib.mod2
````