---
title: jQuery的.bind()、.live()和.delegate()之间区别
date: 2014-01-04 01:56:51
tags: [jQuery]
categories: jQuery
---
`摘要: jQuery的.bind()、.live()和.delegate()之间的区别并非总是那么明显的，然而，如果我们对所有的不同之处都有清晰的理解的话，那么这将会有助于我们编写出更加简洁的代码，以及防止在交互应用中弹出错误。`

[![](/images/jquery-1.jpg)](/images/jquery-1.jpg) 
<!-- more -->

DOM树

首先，可视化一个HMTL文档的DOM树是很有帮助的。一个简单的HTML页面看起来就像是这个样子：
[![](/images/jquery-2.png)](/images/jquery-2.png) 

事件冒泡(又称事件传播)

当我们点击一个链接时，其触发了链接元素的单击事件，该事件则引发任何我们已绑定到该元素的单击事件上的函数的执行。

 
````javascript  
 $('a').bind('click', function() { alert("That tickles!") });  
````
  
因此一个单击操作会触发alert函数的执行。
[![](/images/jquery-3.png)](/images/jquery-3.png) 

click事件接着会向树的根方向传播，广播到父元素，然后接着是每个祖先元素，只要是它的某个后代元素上的单击事件被触发，事件就会传给它。

[![](/images/jquery-4.png)](/images/jquery-4.png) 

在操纵DOM的语境中，document是根节点。

现在我们可以较容易地说明.bind()、.live()和.delegate()的不同之处了。

.bind()

````javascript  
 $('a').bind('click', function() { alert("That tickles!") });  
 ````
  
这是最简单的绑定方法了。JQuery扫描文档找出所有的$(‘a’)元素，并把alert函数绑定到每个元素的click事件上。

.live()

````javascript    
 $('a').live('click', function() { alert("That tickles!") });  
````
  
JQuery把alert函数绑定到$(document)元素上，并使用’click’和’a’作为参数。任何时候只要有事件冒泡到document节点上，它就查看该事件是否是一个click事件，以及该事件的目标元素与’a’这一CSS选择器是否匹配，如果都是的话，则执行函数。

live方法还可以被绑定到具体的元素(或“context”)而不是document上，像这样：

 ````javascript    
 $('a', $('#container')[0]).live(...);  
 ````
  
.delegate()

````javascript  
 $('#container').delegate('a', 'click', function() { alert("That tickles!") });  
````
  
JQuery扫描文档查找$(‘#container’)，并使用click事件和’a’这一CSS选择器作为参数把alert函数绑定到$(‘#container’)上。任何时候只要有事件冒泡到$(‘#container’)上，它就查看该事件是否是click事件，以及该事件的目标元素是否与CCS选择器相匹配。如果两种检查的结果都为真的话，它就执行函数。

可以注意到，这一过程与.live()类似，但是其把处理程序绑定到具体的元素而非document这一根上。精明的JS’er们可能会做出这样的结论，即$('a').live() == $(document).delegate('a')，是这样吗?嗯，不，不完全是。

为什么.delegate()要比.live()好用

基于几个原因，人们通常更愿意选用jQuery的delegate方法而不是live方法。考虑下面的例子：

````javascript     
 $('a').live('click', function() { blah() });     
 // 或者   
 $(document).delegate('a', 'click', function() { blah() });  
````
  
速度

后者实际上要快过前者，因为前者首先要扫描整个的文档查找所有的$(‘a’)元素，把它们存成jQuery对象。尽管live函数仅需要把’a’作为串参数传递以用做之后的判断，但是$()函数并未“知道”被链接的方法将会是.live()。

而另一方面，delegate方法仅需要查找并存储$(document)元素。

一种寻求避开这一问题的方法是调用在$(document).ready()之外绑定的live，这样它就会立即执行。在这种方式下，其会在DOM获得填充之前运行，因此就不会查找元素或是创建jQuery对象了。

灵活性和链能力

live函数也挺令人费解的。想想看，它被链到$(‘a’)对象集上，但其实际上是在$(document)对象上发生作用。由于这个原因，它能够试图以一种吓死人的方式来把方法链到自身上。实际上，我想说的是，以$.live(‘a’,…)这一形式作为一种全局性的jQuery方法，live方法会更具意义一些。

仅支持CSS选择器

最后一点，live方法有一个非常大的缺点，那就是它仅能针对直接的CSS选择器做操作，这使得它变得非常的不灵活。

欲了解更多关于CSS选择器的缺点，请参阅Exploring jQuery .live() and .die()一文。

更新：感谢Hacker News上的pedalpete和后面评论中的Ellsass提醒我加入接下来的这一节内容。

为什么选择.live()或.delegate()而不是.bind()

毕竟，bind看起来似乎更加的明确和直接，难道不是吗?嗯，有两个原因让我们更愿意选择delegate或live而不是bind：

为了把处理程序附加到可能还未存在于DOM中的DOM元素之上。因为bind是直接把处理程序绑定到各个元素上，它不能把处理程序绑定到还未存在于页面中的元素之上。

如果你运行了$(‘a’).bind(…)，而后新的链接经由AJAX加入到了页面中，则你的bind处理程序对于这些新加入的链接来说是无效的。而另一方面live和delegate则是被绑定到另一个祖先节点上，因此其对于任何目前或是将来存在于该祖先元素之内的元素都是有效的。

或者为了把处理程序附加到单个元素上或是一小组元素之上，监听后代元素上的事件而不是循环遍历并把同一个函数逐个附加到DOM中的100个元素上。把处理程序附加到一个(或是一小组)祖先元素上而不是直接把处理程序附加到页面中的所有元素上，这种做法带来了性能上的好处。

停止传播

最后一个我想做的提醒与事件传播有关。通常情况下，我们可以通过使用这样的事件方法来终止处理函数的执行：

````javascript    
 $('a').bind('click', function(e) {      
 e.preventDefault();   
 // 或者   
 e.stopPropagation();   
 });   
 ````
  
不过，当我们使用live或是delegate方法的时候，处理函数实际上并没有在运行，需要等到事件冒泡到处理程序实际绑定的元素上时函数才会运行。而到此时为止，我们的其他的来自.bind()的处理函数早已运行了。

原文地址：http://developer.51cto.com/art/201103/249694.htm