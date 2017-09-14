---
title: go的变量和字面值的类型
date: 2017-09-09 15:44:21
tags: go
categories: go
---
>前几天逛v2ex，无聊看到一个关于这个的话题 [golang 的字面值与类型转换，来猜猜结果](https://www.v2ex.com/t/389157)，所以现在总结下，免得以后进坑。

# 论述
先上代码
````go
package main

import "fmt"

func main() {
 a := 1
 b := 3
 
 fmt.Printf("%T\n", a / b)
 fmt.Printf("%T\n", a / 3)
 fmt.Printf("%T\n", a / 3.0)
 fmt.Printf("%T\n", 1 / 3)
 fmt.Printf("%T\n", 1 / 3.0)
}
````
运行代码：
````
int
int
int
int
float64
````
<!-- more -->
其实这里有奇怪的第三行和最后一行的输出结果，第三行里面a=1跟最后一行是一样的，为什么结果类型不一样呢，很明显这里有类型转换了，究竟谁类型转换了呢，继续上代码
````go
package main

import "fmt"

func main() {
 a := 1
 
 fmt.Println(a / 3)
 fmt.Println(a / 3.0)
 //fmt.Println(a / 3.1) //类型错误
}
````
结果
````
0
0
# literal/test2/main.go:10: constant 3.1 truncated to integer
````
很明显是字面值转换了类型，最后一行的3.1是转换不了整形的，所以就报错了，而3.0是没问题的，那为什么变量不会转换呢，继续上代码
````go
package main

import "fmt"

func main() {
	a := 1
	b := 3.0

	fmt.Printf("%T\n", a)
	fmt.Printf("%T\n", b)
    //fmt.Printf("%T\n", a/b) //编译错误
}
````
结果：
````
int
float64
#literal/test3/main.go:11: invalid operation: a / b (mismatched types int and float64)
````
很明显变量类型在初始化赋值的时候就确定，在运算的时候变量不会去类型转换。

# 总结
go里面的变量运算是保证运算变量一定是相同类型才行，否则会编译错误，而且是初始赋值后就确定类型，不会在运算时自动帮你转换。但是字面值不同，在不同的场景会转换成不同的类型，当然前提是可以转换，否则就跟上面的例子3.1一样，没办法转换成整形而报错。
所有代码在 https://github.com/ejunjsh/go-code/tree/master/literal