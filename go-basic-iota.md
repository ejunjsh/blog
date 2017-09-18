---
title: go基础-iota
date: 2017-09-18 15:44:21
tags: go
categories: go
---
> iota这个关键字，用来实现枚举的功能，但是用起来很奇怪，其实最后表示的还是常量。

# 先上代码
````go
package main

import (
	"fmt"
)

const(
    a= 10+iota
    b
    c
    d
)

func main(){
	fmt.Println(a)
	fmt.Println(b)
	fmt.Println(c)
	fmt.Println(d)
}
````
结果：
````
10
11
12
13
````
其实上面代码等价于
````go
const(
    a= 10+0
    b= 10+1
    c= 10+2
    d= 10+3
)
````
看出规律了吧，只要iota出现一次，就累加一次，而且一旦出现一次，就算后面不使用这个iota关键字，接下来的变量都会套用前面的表达式来计算，所以b,c,d用的就是a的表达式

# 再上代码
````go
package main

import (
	"fmt"
)

const(
    a= 10+iota
    b
    c
    d
	  e=1+iota
	  f
)

func main(){
	fmt.Println(a)
	fmt.Println(b)
	fmt.Println(c)
	fmt.Println(d)
	fmt.Println(e)
	fmt.Println(f)
}

````
这个的输出，应该能猜出来了吧
````
10
11
12
13
5
6
````
f很明显是由于e的表达式变了，所以是套用了e的表达式，而不用a的表达式

# 最后的代码
````go
package main

import (
	"fmt"
)

const(
    a= 10+iota
    b
    c
)

const(
	d=1+iota
	e
	f
)

func main(){
	fmt.Println(a)
	fmt.Println(b)
	fmt.Println(c)
	fmt.Println(d)
	fmt.Println(e)
	fmt.Println(f)
}
````
结果：
````
10
11
12
1
2
3
````
很明显，iota只能在一个代码块累加，在另外的代码块就又重置了。

# 总结
其实iota在go就是一个常量，定义在builtin.go这个源文件
````go
// iota is a predeclared identifier representing the untyped integer ordinal
// number of the current const specification in a (usually parenthesized)
// const declaration. It is zero-indexed.
const iota = 0 // Untyped int.
````

本文代码在 https://github.com/ejunjsh/go-code/tree/master/iota


