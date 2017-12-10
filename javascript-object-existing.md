---
title: 如何判断Javascript对象是否存在
date: 2014-03-26 01:17:54
tags: javascript
categories: javascript
---
> javascript 设计不严谨，就判断一个Javascript对象是否存在，有超过50种写法。 这里列举些常用的，并总结下。。

# 第一种
````javascript
　　if (!myObj) {
　　　　myObj = { };
　　}
````
但是，运行这段代码，浏览器会直接抛出ReferenceError错误，导致运行中断。请问错在哪里？
对了，if语句判断myObj是否为空时，这个变量还不存在，所以才会报错。改成下面这样，就能正确运行了。
<!-- more -->
````javascript
　　if (!myObj) {
　　　　var myObj = { };
　　}
````
为什么加了一个var以后，就不报错了？难道这种情况下，if语句做判断时，myObj就已经存在了吗？
要回答这个问题，就必须知道Javascript解释器的工作方式。Javascript语言是"先解析，后运行"，解析时就已经完成了变量声明，所以上面的代码实际等同于：
````javascript
　　var myObj;
　　if (!myObj) {
　　　　var myObj = { };
　　}
````
因此，if语句做判断时，myObj确实已经存在了，所以就不报错了。这就是var命令的"代码提升"（hoisting）作用。Javascript解释器，只"提升"var命令定义的变量，对不使用var命令、直接赋值的变量不起作用，这就是为什么不加var会报错的原因。

# 第二种
除了var命令，还可以有另一种改写，也能得到正确的结果：
````javascript
if (!window.myObj) {
　　　　myObj = { };
　　}
````
window是javascript的顶层对象，所有的全局变量都是它的属性。所以，判断myobj是否为空，等同于判断window对象是否有myobj属性，这样就可以避免因为myObj没有定义而出现ReferenceError错误。不过，从代码的规范性考虑，最好还是对第二行加上var：
````javascript
if (!window.myObj) {
　　　　var myObj = { };
　　}
````
或者写成这样：
````javascript
if (!window.myObj) {
　　　　window.myObj = { };
　　}
````

# 第三种
上面这种写法的缺点在于，在某些运行环境中（比如V8、Rhino），window未必是顶层对象。所以，考虑改写成：
````javascript
　　if (!this.myObj) {
　　　　this.myObj = { };
　　}
````
在全局变量的层面中，this关键字总是指向顶层变量，所以就可以独立于不同的运行环境。

# 第四种
但是，上面这样写可读性较差，而且this的指向是可变的，容易出错，所以进一步改写：
````javascript
　　var global = this;
　　if (!global.myObj) {
　　　　global.myObj = { };
　　}
````
用自定义变量global表示顶层对象，就清楚多了。

# 第五种
还可以使用typeof运算符，判断myObj是否有定义。
````javascript
if (typeof myObj == "undefined") {
　　　　var myObj = { };
　　}
````
这是目前使用最广泛的判断javascript对象是否存在的方法。

# 第六种
由于在已定义、但未赋值的情况下，myObj的值直接等于undefined，所以上面的写法可以简化：
````javascript
if (myObj == undefined) {
　　　　var myObj = { };
　　}
````
这里有两个地方需要注意，首先第二行的var关键字不能少，否则会出现ReferenceError错误，其次undefined不能加单引号或双引号，因为这里比较的是undefined这种数据类型，而不是"undefined"这个字符串。

# 第七种
上面的写法在"精确比较"（===）的情况下，依然成立：
````javascript
if (myObj === undefined) {
　　　　var myObj = { };
　　}
````

# 第八种
根据javascript的语言设计，undefined == null，所以比较myObj是否等于null，也能得到正确结果：
````javascript
if (myObj == null) {
　　　　var myObj = { };
　　}
````
不过，虽然运行结果正确，但是从语义上看，这种判断方法是错的，应该避免。因为null指的是已经赋值为null的空对象，即这个对象实际上是有值的，而undefined指的是不存在或没有赋值的对象。因此，这里只能使用"比较运算符"（==），如果这里使用"精确比较运算符"（===），就会出错。

# 第九种
还可以使用in运算符，判断myObj是否为顶层对象的一个属性：
````javascript
if (!('myObj' in window)) {
　　　　window.myObj = { };
　　}
````

# 第十种
最后，使用hasOwnProperty方法，判断myObj是否为顶层对象的一个属性：
````javascript
if (!this.hasOwnProperty('myObj')) {
　　　　this.myObj = { };
　　}
````

# 总结
1. 如果只判断对象是否存在，推荐使用第五种写法。
2. 如果除了对象是否存在，还要判断对象是否有null值，推荐使用第一种写法。
3. 除非特殊情况，所有变量都应该使用var命令声明。
4. 为了跨平台，建议避免使用window表示顶层对象。
5. 在Javascript语言中，null和undefined容易产生混淆。在可能同时涉及两者的情况下，建议使用"精确比较"运算符（===）。

参考 http://www.ruanyifeng.com/blog/2011/05/how_to_judge_the_existence_of_a_global_object_in_javascript.html