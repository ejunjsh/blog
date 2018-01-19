---
title: 深入go接口
date: 2018-01-15 15:44:21
tags: go
categories: go
---
# 接口是什么
接口就是一个抽象类型，与之对应的就是具体类型，同时接口也是抽象方法接口。
````go
type human interface{
	walk()
	run()
	eat()
}
````
上面的代码定义了接口，接口里定义了几个抽象方法，一般其他语言例如Java，都会定义一个具体的类型来实现这个接口，像这样`class woman implements human` 声明`woman`实现了`human`。但是go上使用了一种`duck typing`来定义具体类型。
<!-- more -->

> When I see a bird that walks like a duck and swins like a duck and quacks like a duck, I call that bird a duck. – James Whitcomb Riley

结合维基百科的[定义](https://en.wikipedia.org/wiki/Duck_typing)，duck typing是面向对象编程语言的一种类型定义方法。我们判断一个对象是什么不是通过它的类型定义来判断，而是判断它是否满足某些特定的方法和属性定义。
````go
type man struct{}

func (m *man) walk(){
	fmt.Println("man walk")
}

func (m *man) run(){
	fmt.Println("man run")
}

func (m *man) eat(){
	fmt.Println("man eat")
}
````
上面定义的`man` 并没有声明它实现了哪些接口，但是，它确切是`human`的具体类型
````go
func main(){
	var h human=&man{}
	h.eat()
	h.run()
	h.walk()
}
````
运行结果
````
man eat
man run
man walk
````
上面结果说明了，不用声明某具体类型实现哪个接口，只要它有某接口的所有方法，那它就是某接口的具体类型。

# 接口实现多态

# 接口实现泛型

# 深入源码

https://zhuanlan.zhihu.com/p/27652856
