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
上面的代码定义了接口，接口里定义了几个抽象方法，一般其他语言例如Java，都会定义一个具体的类型来实现这个接口，像这样`class man implements human` 声明`man`实现了`human`。但是go上使用了一种`duck typing`来定义具体类型。
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
很明显，再加个`woman`类型，实现所有方法就可以实现个多态（方法重载）
````go
type woman struct{}

func (m *woman) walk(){
	fmt.Println("woman walk")
}

func (m *woman) run(){
	fmt.Println("woman run")
}

func (m *woman) eat(){
	fmt.Println("woman eat")
}

func main(){
	var h human=&man{}
	h.eat()
	h.run()
    h.walk()
    h=&woman{}
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
woman eat
woman run
woman walk
````

# 类型断言
所谓类型断言，就是一个接口类型，转化成具体类型时候使用。还有go里面`interface{}`是一个万能的类型，有点像java的`Object`类型
````go
func do(v interface{}) {
    n := v.(int)    // might panic
}

func do(v interface{}) {
    n, ok := v.(int)
    if !ok {
        // 断言失败处理
    }
}
````
上面的方法可以用所有的不同类型的参数调用，所以如果不是`int`类型参数的话就是panic了，所以第二个`do`可以进行断言失败处理，避免错误发生。

# 深入源码
 
## interface 底层结构
根据 interface 是否包含有 method，底层实现上用两种 struct 来表示：iface 和 eface。eface表示不含 method 的 interface 结构，或者叫 empty interface。对于 Golang 中的大部分数据类型都可以抽象出来 _type 结构，同时针对不同的类型还会有一些其他信息。
````go
type eface struct {
    _type *_type
    data  unsafe.Pointer
}

type _type struct {
    size       uintptr // type size
    ptrdata    uintptr // size of memory prefix holding all pointers
    hash       uint32  // hash of type; avoids computation in hash tables
    tflag      tflag   // extra type information flags
    align      uint8   // alignment of variable with this type
    fieldalign uint8   // alignment of struct field with this type
    kind       uint8   // enumeration for C
    alg        *typeAlg  // algorithm table
    gcdata    *byte    // garbage collection data
    str       nameOff  // string form
    ptrToThis typeOff  // type for pointer to this type, may be zero
}
````
iface 表示 non-empty interface 的底层实现。相比于 empty interface，non-empty 要包含一些 method。method 的具体实现存放在 itab.fun 变量里。如果 interface 包含多个 method，这里只有一个 fun 变量怎么存呢？这个下面再细说。
````go
type iface struct {
    tab  *itab
    data unsafe.Pointer
}

// layout of Itab known to compilers
// allocated in non-garbage-collected memory
// Needs to be in sync with
// ../cmd/compile/internal/gc/reflect.go:/^func.dumptypestructs.
type itab struct {
    inter  *interfacetype
    _type  *_type
    link   *itab
    bad    int32
    inhash int32      // has this itab been added to hash?
    fun    [1]uintptr // variable sized
}
````
我们使用实际程序来看一下。
````go
package main

import (
    "fmt"
)

type MyInterface interface {
    Print()
}

type MyStruct struct{}
func (ms MyStruct) Print() {}

func main() {
    x := 1
    var y interface{} = x
    var s MyStruct
    var t MyInterface = s
    fmt.Println(y, z)
}
````
查看汇编代码。
````
$ go build -gcflags '-l' -o interface11 interface11.go
$ go tool objdump -s "main\.main" interface11
TEXT main.main(SB) /Users/kltao/code/go/examples/interface11.go
    interface11.go:15   0x10870f0   65488b0c25a0080000  GS MOVQ GS:0x8a0, CX
    interface11.go:15   0x10870f9   483b6110        CMPQ 0x10(CX), SP
    interface11.go:15   0x10870fd   0f86de000000        JBE 0x10871e1
    interface11.go:15   0x1087103   4883ec70        SUBQ $0x70, SP
    interface11.go:15   0x1087107   48896c2468      MOVQ BP, 0x68(SP)
    interface11.go:15   0x108710c   488d6c2468      LEAQ 0x68(SP), BP
    interface11.go:17   0x1087111   48c744243001000000  MOVQ $0x1, 0x30(SP)
    interface11.go:17   0x108711a   488d057fde0000      LEAQ 0xde7f(IP), AX
    interface11.go:17   0x1087121   48890424        MOVQ AX, 0(SP)
    interface11.go:17   0x1087125   488d442430      LEAQ 0x30(SP), AX
    interface11.go:17   0x108712a   4889442408      MOVQ AX, 0x8(SP)
    interface11.go:17   0x108712f   e87c45f8ff      CALL runtime.convT2E(SB)
    interface11.go:17   0x1087134   488b442410      MOVQ 0x10(SP), AX
    interface11.go:17   0x1087139   4889442438      MOVQ AX, 0x38(SP)
    interface11.go:17   0x108713e   488b4c2418      MOVQ 0x18(SP), CX
    interface11.go:17   0x1087143   48894c2440      MOVQ CX, 0x40(SP)
    interface11.go:19   0x1087148   488d15b1000800      LEAQ 0x800b1(IP), DX
    interface11.go:19   0x108714f   48891424        MOVQ DX, 0(SP)
    interface11.go:19   0x1087153   488d542430      LEAQ 0x30(SP), DX
    interface11.go:19   0x1087158   4889542408      MOVQ DX, 0x8(SP)
    interface11.go:19   0x108715d   e8fe45f8ff      CALL runtime.convT2I(SB)
````
代码 17 行 var y interface{} = x 调用了函数 runtime.convT2E，将 int 类型的 x 转换成 empty interface。代码 19 行 var t MyInterface = s 将 MyStruct 类型转换成 non-empty interface: MyInterface。
````go
func convT2E(t *_type, elem unsafe.Pointer) (e eface) {
    ...
  
    x := newobject(t)
    typedmemmove(t, x, elem)
    e._type = t
    e.data = x
    return
}

func convT2I(tab *itab, elem unsafe.Pointer) (i iface) {
    t := tab._type
    
    ...
  
    x := newobject(t)
    typedmemmove(t, x, elem)
    i.tab = tab
    i.data = x
    return
}
````
看上面的函数原型，可以看出中间过程编译器将根据我们的转换目标类型的 empty interface 还是 non-empty interface，来对原数据类型进行转换（转换成 <*_type, unsafe.Pointer> 或者 <*itab, unsafe.Pointer>）。这里对于 struct 满不满足 interface 的类型要求（也就是 struct 是否实现了 interface 的所有 method），是由编译器来检测的。

## itab
iface 结构中最重要的是 itab 结构。itab 可以理解为 `pair<interface type, concrete type>` 。当然 itab 里面还包含一些其他信息，比如 interface 里面包含的 method 的具体实现。下面细说。itab 的结构如下。

````go
type itab struct {
    inter  *interfacetype
    _type  *_type
    link   *itab
    bad    int32
    inhash int32      // has this itab been added to hash?
    fun    [1]uintptr // variable sized
}
````
其中 interfacetype 包含了一些关于 interface 本身的信息，比如 package path，包含的 method。上面提到的 iface 和 eface 是数据类型（built-in 和 type-define）转换成 interface 之后的实体的 struct 结构，而这里的 interfacetype 是我们定义 interface 时候的一种抽象表示。
````go
type interfacetype struct {
    typ     _type
    pkgpath name
    mhdr    []imethod
}

type imethod struct {   //这里的 method 只是一种函数声明的抽象，比如  func Print() error
    name nameOff
    ityp typeOff
}
````
_type 表示 concrete type。fun 表示的 interface 里面的 method 的具体实现。比如 interface type 包含了 method A, B，则通过 fun 就可以找到这两个 method 的具体实现。这里有个问题 fun 是长度为 1 的 uintptr 数组，那么怎么表示多个 method 呢？看一下测试程序。
````go
package main

type MyInterface interface {
    Print()
    Hello()
    World()
    AWK()
}

func Foo(me MyInterface) {
    me.Print()
    me.Hello()
    me.World()
    me.AWK()
}

type MyStruct struct {}

func (me MyStruct) Print() {}
func (me MyStruct) Hello() {}
func (me MyStruct) World() {}
func (me MyStruct) AWK() {}

func main() {
    var me MyStruct
    Foo(me)
}
````
看一下函数调用对应的汇编代码。
````
$ go build -gcflags '-l' -o interface8 interface8.go
$ go tool objdump -s "main\.Foo" interface8
TEXT main.Foo(SB) /Users/kltao/code/go/examples/interface8.go
    interface8.go:10    0x104c060   65488b0c25a0080000  GS MOVQ GS:0x8a0, CX
    interface8.go:10    0x104c069   483b6110        CMPQ 0x10(CX), SP
    interface8.go:10    0x104c06d   7668            JBE 0x104c0d7
    interface8.go:10    0x104c06f   4883ec10        SUBQ $0x10, SP
    interface8.go:10    0x104c073   48896c2408      MOVQ BP, 0x8(SP)
    interface8.go:10    0x104c078   488d6c2408      LEAQ 0x8(SP), BP
    interface8.go:11    0x104c07d   488b442418      MOVQ 0x18(SP), AX
    interface8.go:11    0x104c082   488b4830        MOVQ 0x30(AX), CX //取得 Print 函数地址
    interface8.go:11    0x104c086   488b542420      MOVQ 0x20(SP), DX
    interface8.go:11    0x104c08b   48891424        MOVQ DX, 0(SP)
    interface8.go:11    0x104c08f   ffd1            CALL CX     // 调用 Print()
    interface8.go:12    0x104c091   488b442418      MOVQ 0x18(SP), AX
    interface8.go:12    0x104c096   488b4828        MOVQ 0x28(AX), CX //取得 Hello 函数地址
    interface8.go:12    0x104c09a   488b542420      MOVQ 0x20(SP), DX
    interface8.go:12    0x104c09f   48891424        MOVQ DX, 0(SP)
    interface8.go:12    0x104c0a3   ffd1            CALL CX           //调用 Hello()
    interface8.go:13    0x104c0a5   488b442418      MOVQ 0x18(SP), AX
    interface8.go:13    0x104c0aa   488b4838        MOVQ 0x38(AX), CX //取得 World 函数地址
    interface8.go:13    0x104c0ae   488b542420      MOVQ 0x20(SP), DX 
    interface8.go:13    0x104c0b3   48891424        MOVQ DX, 0(SP)
    interface8.go:13    0x104c0b7   ffd1            CALL CX           //调用 World()
    interface8.go:14    0x104c0b9   488b442418      MOVQ 0x18(SP), AX
    interface8.go:14    0x104c0be   488b4020        MOVQ 0x20(AX), AX //取得 AWK 函数地址
    interface8.go:14    0x104c0c2   488b4c2420      MOVQ 0x20(SP), CX
    interface8.go:14    0x104c0c7   48890c24        MOVQ CX, 0(SP)
    interface8.go:14    0x104c0cb   ffd0            CALL AX           //调用 AWK()
    interface8.go:15    0x104c0cd   488b6c2408      MOVQ 0x8(SP), BP
    interface8.go:15    0x104c0d2   4883c410        ADDQ $0x10, SP
    interface8.go:15    0x104c0d6   c3          RET
    interface8.go:10    0x104c0d7   e8f48bffff      CALL runtime.morestack_noctxt(SB)
    interface8.go:10    0x104c0dc   eb82            JMP main.Foo(SB)
````
其中 0x18(SP) 对应的 itab 的地址。fun 在 x86-64 机器上对应 itab 内的地址偏移为 8+8+8+4+4 = 32 = 0x20，也就是 0x20(AX) 对应的 fun 的值，此时存放的 AWK 函数地址。然后 0x28(AX) = &Hello，0x30(AX) = &Print，0x38(AX) = &World。对的，每次函数是按字典序排序存放的。

我们再来看一下函数地址究竟是怎么写入的？首先 Golang 中的 uintptr 一般用来存放指针的值，这里对应的就是函数指针的值（也就是函数的调用地址）。但是这里的 fun 是一个长度为 1 的 uintptr 数组。我们看一下 runtime 包的 additab 函数。
````go
func additab(m *itab, locked, canfail bool) {
    ...
    *(*unsafe.Pointer)(add(unsafe.Pointer(&m.fun[0]), uintptr(k)*sys.PtrSize)) = ifn
    ...
}
````
上面的代码的意思是在 fun[0] 的地址后面依次写入其他 method 对应的函数指针。熟悉 C++ 的同学可以类比 C++ 的虚函数表指针来看。

剩下的还有 bad，link，inhash。其中 bad 是一个表征 itab 状态的变量。而这里的 link 是 *itab 类型，是不是表示 interface 的嵌套呢？并不是，interface 的嵌套也是把 method 平铺而已。link 要和 inhash 一起来说。在 runtime 包里面有一个 hash 表，通过 hash[hashitab(interface\_type, concrete\_type)] 可以取得 itab，这是出于性能方面的考虑。主要代码如下，这里就不再赘述了。
````go
const (
    hashSize = 1009
)

var (
    ifaceLock mutex // lock for accessing hash
    hash      [hashSize]*itab
)

func itabhash(inter *interfacetype, typ *_type) uint32 {
    // compiler has provided some good hash codes for us.
    h := inter.typ.hash
    h += 17 * typ.hash
    // TODO(rsc): h += 23 * x.mhash ?
    return h % hashSize
}

func additab(...) {
    ...
    h := itabhash(inter, typ)
    m.link = hash[h]
    m.inhash = 1
    atomicstorep(unsafe.Pointer(&hash[h]), unsafe.Pointer(m))
}
````

## 类型断言
上面有说到的断言，它的实现源码如下几个函数：
````go
// The assertXXX functions may fail (either panicking or returning false,
// depending on whether they are 1-result or 2-result).
func assertI2I(inter *interfacetype, i iface) (r iface) {
    tab := i.tab
    if tab == nil {
        // explicit conversions require non-nil interface value.
        panic(&TypeAssertionError{"", "", inter.typ.string(), ""})
    }
    if tab.inter == inter {
        r.tab = tab
        r.data = i.data
        return
    }
    r.tab = getitab(inter, tab._type, false)
    r.data = i.data
    return
}
func assertI2I2(inter *interfacetype, i iface) (r iface, b bool) {
    tab := i.tab
    if tab == nil {
        return
    }
    if tab.inter != inter {
        tab = getitab(inter, tab._type, true)
        if tab == nil {
            return
        }
    }
    r.tab = tab
    r.data = i.data
    b = true
    return
}

// 类似
func assertE2I(inter *interfacetype, e eface) (r iface)
func assertE2I2(inter *interfacetype, e eface) (r iface, b bool)
````

参考 https://zhuanlan.zhihu.com/p/27652856
