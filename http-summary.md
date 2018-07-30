---
title: HTTP 总结
date: 2018-05-22 22:51:18
tags: [http,protocols]
categories: protocols
---

> 👿 mark ，很长 😄

# HTTP 概述

Web 使用一种名为 HTTP (HyperText Transfer Protocol，超文本传输协议) 的协议作为规范的。
>HTTP 更加严谨的译名应该是 超文本转移协议。


HTTP 于 1990 年问世。那时的 HTTP 并没有作为正式的标准，因为被称为 HTTP/0.9   
HTTP 正式作为标准被公布是 1996 年 5 月，版本命名为 HTTP/1.0，记载于 RFC1945   
HTTP 在 1997 年 1 月公布了当前最主流的版本，版本命名为 HTTP/1.1，记载于 RFC2616  
HTTP/2 于 2015 年 5 月 14 日发布，引入了服务器推送等多种功能，是目前最新的版本。记载于 RFC7540
(它不叫 HTTP/2.0，是因为标准委员会不打算再发布子版本了，下一个新版本将是 HTTP/3)

<!-- more -->


# HTTP 支持的方法

HTTP 是一种不保存状态，即 无状态（ stateless ）协议。HTTP 协议自身不对请求和响应之间的通信状态进行保存。也就是说在 HTTP 这个级别，协议对于发送过的请求或响应都不做持久化处理。这也是为了更快的处理大量事务，确保协议的可伸缩性。

HTTP/1.1 虽然是无状态协议，但是为了实现期望的保持状态的功能，特意引入了 Cookie 技术。

| 方法名 | 说明 | 支持的 HTTP 协议版本 | 详细说明|
| :---: | :---: | :---: |:---: |
| GET | 获取资源 | 1.0、1.1 | GET 方法用来请求访问已被 URI 识别的资源。指定的资源经服务器端解析后返回响应内容。（我想访问你的某个资源）|
| POST | 传输实体主体 | 1.0、1.1 | POST 方法用来传输实体的主体。虽然 GET 也可以传输实体的主体，但一般不用 GET 而用 POST，POST 的主要目的并不是获取响应的主体内容。（我想把这条信息告诉你）|
| PUT | 传输文件 | 1.0、1.1 | 要求在请求报文的主体中包含文件内容，然后保存到请求 URI 指定位置。（我想要把这份文件传给你）|
| HEAD | 获取报文首部 | 1.0、1.1 | HEAD 方法和 GET 方法一样，只是不返回报文主体部分。用于确认 URI 的有效性及资源更新的日期时间等等（我想要那个相关信息）|
| DELETE | 删除文件 | 1.0、1.1 | 与 PUT 相反的方法，DELETE 方法按请求 URI 删除指定资源（把这份文件删掉吧）|
| OPTIONS | 询问支持的方法 | 1.1 | OPTIONS 用来查询针对请求 URI 指定的资源支持的方法（你支持哪些方法？）|
| TRACE | 追踪路径 | 1.1 | TRACE 方法是让 Web 服务器将之前的请求通信返回给客户端的方法，TRACE 方法不常用，并且容易引发 XST ( Cross-Site-Tracing ，跨站追踪)攻击，所以通常更不会用到了|
| CONNECT | 要求用隧道协议连接代理 | 1.1 | CONNECT 方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行 TCP 通信，主要使用 SSL （ Secure Sockets Layers ，安全套接层）和 TLS （ Transport Layer Security ，传输层安全）协议把通信内容加密后经网络隧道传输|
| PATCH | 更新部分文件内容| 1.1| **当资源存在的时候**，PATCH 用于资源的部分内容的更新，例如更新某一个字段。具体比如说只更新用户信息的电话号码字段，而 PUT 用于更新某个资源较完整的内容，比如说用户要重填完整表单更新所有信息，后台处理更新时可能只是保留内部记录 ID 不变。<br>**当资源不存在的时候**，PATCH 是修改原来的内容，也可能会产生一个新的版本。比如当资源不存在的时候，PATCH 可能会去创建一个新的资源，这个意义上像是 saveOrUpdate 操作。而 PUT 只对已有资源进行更新操作，所以是 update 操作|
| LINK | 建立和资源之间的联系 | 1.0 | ✖︎最新版中已经废弃✖︎|
| UNLINK | 断开连接关系 | 1.0 | ✖︎最新版中已经废弃✖︎|
| PROPFIND | 获取属性 | 1.1 | WebDAV 获取属性|
| PROPPATCH | 修改属性 | 1.1 | WebDAV 修改属性|
| MKCOL | 创建属性 | 1.1 | WebDAV 创建属性|
| COPY | 复制资源及属性 | 1.1 | WebDAV 复制资源及属性|
| MOVE | 移动资源 | 1.1 | WebDAV 移动资源|
| LOCK | 资源加锁 | 1.1 | WebDAV 资源加锁|
| UNLOCK | 资源解锁 | 1.1 | WebDAV 资源解锁|


在HTTP/1.1规范中幂等性的定义是：

> Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.

从定义上看，HTTP 方法的幂等性是指一次和多次请求某一个资源应该具有同样的副作用。幂等性属于语义范畴，正如编译器只能帮助检查语法错误一样，HTTP 规范也没有办法通过消息格式等语法手段来定义它，这可能是它不太受到重视的原因之一。但实际上，幂等性是分布式系统设计中十分重要的概念，而 HTTP 的分布式本质也决定了它在 HTTP 中具有重要地位。


HTTP 方法的安全性指的是不会改变服务器状态，也就是说它只是可读的。所以只有 OPTIONS、GET、HEAD 是安全的，其他都是不安全的。

| HTTP 方法 | 幂等性 | 安全性 |
| :---: | :---: | :---: |
|OPTIONS|	yes	|yes|
|GET	|yes	|yes|
|HEAD	|yes	|yes|
|PUT	|yes	|no|
|DELETE	|yes	|no|
|POST	|no	|no|
|PATCH	|no	|no|

**POST 和 PATCH 这两个不是幂等性的**。  
两次相同的POST请求会在服务器端创建两份资源，它们具有不同的URI。  
对同一URI进行多次PUT的副作用和一次PUT是相同的。  


# HTTP 状态码


服务器返回的  **响应报文**  中第一行为状态行，包含了状态码以及原因短语，用来告知客户端请求的结果。

| 状态码 | 类别 | 原因短语 |
| :---: | :---: | :---: |
| 1XX | Informational（信息性状态码） | 接收的请求正在处理 |
| 2XX | Success（成功状态码） | 请求正常处理完毕 |
| 3XX | Redirection（重定向状态码） | 需要进行附加操作以完成请求 |
| 4XX | Client Error（客户端错误状态码） | 服务器无法处理请求 |
| 5XX | Server Error（服务器错误状态码） | 服务器处理请求出错 |




## 1XX 信息

-  **100 Continue** ：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应。

## 2XX 成功

-  **200 OK** 

-  **204 No Content** ：请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用。

-  **206 Partial Content** ：表示客户端进行了范围请求。响应报文包含由 Content-Range 指定范围的实体内容。

## 3XX 重定向

-  **301 Moved Permanently** ：永久性重定向

-  **302 Found** ：临时性重定向

-  **303 See Other** ：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源。

- 注：虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法。

-  **304 Not Modified** ：如果请求报文首部包含一些条件，例如：If-Match，If-ModifiedSince，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。

-  **307 Temporary Redirect** ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。

## 4XX 客户端错误

-  **400 Bad Request** ：请求报文中存在语法错误。

-  **401 Unauthorized** ：该状态码表示发送的请求需要有认证信息（BASIC 认证、DIGEST 认证）。如果之前已进行过一次请求，则表示用户认证失败。

-  **403 Forbidden** ：请求被拒绝，服务器端没有必要给出拒绝的详细理由。

-  **404 Not Found** 

## 5XX 服务器错误

-  **500 Internal Server Error** ：服务器正在执行请求时发生错误。

-  **503 Service Unavilable** ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

# HTTP 报文结构

[![](http://idiotsky.top/images3/http-summary-1.png)](http://idiotsky.top/images3/http-summary-1.png)


[![](http://idiotsky.top/images3/http-summary-2.png)](http://idiotsky.top/images3/http-summary-2.png)


[![](http://idiotsky.top/images3/http-summary-3.png)](http://idiotsky.top/images3/http-summary-3.png)

[![](http://idiotsky.top/images3/http-summary-4.png)](http://idiotsky.top/images3/http-summary-4.png)


Response Headers:

[![](http://idiotsky.top/images3/http-summary-5.png)](http://idiotsky.top/images3/http-summary-5.png)


Request Headers:

[![](http://idiotsky.top/images3/http-summary-6.png)](http://idiotsky.top/images3/http-summary-6.png)


请求报文是由请求方法，请求 URI，协议版本，可选请求首部字段和内容实体构成的。

响应报文基本上由协议版本，状态码（表示请求成功与失败的数字代码），用以解释状态码的原因短语，可选的响应首部字段以及实体主体构成。


# HTTP 缓存控制

## 缓存规则解析
为方便大家理解，我们认为浏览器存在一个缓存数据库,用于存储缓存信息。

在客户端第一次请求数据时，此时缓存数据库中没有对应的缓存数据，需要请求服务器，服务器返回后，将数据存储至缓存数据库中。

[![](http://idiotsky.top/images3/http-summary-21.png)](http://idiotsky.top/images3/http-summary-21.png)

HTTP缓存有多种规则，根据是否需要重新向服务器发起请求来分类，我将其分为两大类(**强制缓存，对比缓存**)

在详细介绍这两种规则之前，先通过时序图的方式，让大家对这两种规则有个简单了解。

已存在缓存数据时，仅基于强制缓存，请求数据的流程如下

[![](http://idiotsky.top/images3/http-summary-22.png)](http://idiotsky.top/images3/http-summary-22.png)

已存在缓存数据时，仅基于对比缓存，请求数据的流程如下

[![](http://idiotsky.top/images3/http-summary-23.png)](http://idiotsky.top/images3/http-summary-23.png)

对缓存机制不太了解的同学可能会问，基于对比缓存的流程下，不管是否使用缓存，都需要向服务器发送请求，那么还用缓存干什么？

这个问题，我们暂且放下，后文在详细介绍每种缓存规则的时候，会带给大家答案。

我们可以看到两类缓存规则的不同，强制缓存如果生效，不需要再和服务器发生交互，而对比缓存不管是否生效，都需要与服务端发生交互。

两类缓存规则可以同时存在，强制缓存优先级高于对比缓存，也就是说，当执行强制缓存的规则时，如果缓存生效，直接使用缓存，不再执行对比缓存规则。

## 强制缓存

从上文我们得知，强制缓存，在缓存数据未失效的情况下，可以直接使用缓存数据，那么浏览器是如何判断缓存数据是否失效呢？

我们知道，在没有缓存数据的时候，浏览器向服务器请求数据时，服务器会将数据和缓存规则一并返回，缓存规则信息包含在响应header中。

对于强制缓存来说，响应header中会有两个字段来标明失效规则（Expires/Cache-Control）

使用chrome的开发者工具，可以很明显的看到对于强制缓存生效时，网络请求的情况

[![](http://idiotsky.top/images3/http-summary-24.png)](http://idiotsky.top/images3/http-summary-24.png)

### Expires
Expires的值为服务端返回的到期时间，即下一次请求时，请求时间小于服务端返回的到期时间，直接使用缓存数据。

不过Expires 是HTTP 1.0的东西，现在默认浏览器均默认使用HTTP 1.1，所以它的作用基本忽略。

另一个问题是，到期时间是由服务端生成的，但是客户端时间可能跟服务端时间有误差，这就会导致缓存命中的误差。

所以HTTP 1.1 的版本，使用Cache-Control替代。

### Cache-Control
Cache-Control 是最重要的规则。常见的取值有private、public、no-cache、max-age，no-store，默认为private。
* private:客户端可以缓存
* public:客户端和代理服务器都可缓存（前端的同学，可以认为public和private是一样的）
* max-age=xxx:缓存的内容将在 xxx 秒后失效
* no-cache:需要使用对比缓存来验证缓存数据（后面介绍）
* no-store:所有内容都不会缓存，强制缓存，对比缓存都不会触发（对于前端开发来说，缓存越多越好，so...基本上和它说886）

举个板栗

[![](http://idiotsky.top/images3/http-summary-25.png)](http://idiotsky.top/images3/http-summary-25.png)

图中Cache-Control仅指定了max-age，所以默认为private，缓存时间为31536000秒（365天）

也就是说，在365天内再次请求这条数据，都会直接获取缓存数据库中的数据，直接使用。

## 对比缓存

对比缓存，顾名思义，需要进行比较判断是否可以使用缓存。

浏览器第一次请求数据时，服务器会将缓存标识与数据一起返回给客户端，客户端将二者备份至缓存数据库中。

再次请求数据时，客户端将备份的缓存标识发送给服务器，服务器根据缓存标识进行判断，判断成功后，返回304状态码，通知客户端比较成功，可以使用缓存数据。

第一次访问：

[![](http://idiotsky.top/images3/http-summary-26.png)](http://idiotsky.top/images3/http-summary-26.png)

再次访问：

[![](http://idiotsky.top/images3/http-summary-27.png)](http://idiotsky.top/images3/http-summary-27.png)

通过两图的对比，我们可以很清楚的发现，在对比缓存生效时，状态码为304，并且报文大小和请求时间大大减少。

原因是，服务端在进行标识比较后，只返回header部分，通过状态码通知客户端使用缓存，不再需要将报文主体部分返回给客户端。

对于对比缓存来说，缓存标识的传递是我们着重需要理解的，它在请求header和响应header间进行传递，

一共分为两种标识传递，接下来，我们分开介绍。

### Last-Modified  /  If-Modified-Since

Last-Modified：

服务器在响应请求时，告诉浏览器资源的最后修改时间。

[![](http://idiotsky.top/images3/http-summary-28.png)](http://idiotsky.top/images3/http-summary-28.png)

If-Modified-Since：

再次请求服务器时，通过此字段通知服务器上次请求时，服务器返回的资源最后修改时间。

服务器收到请求后发现有头If-Modified-Since 则与被请求资源的最后修改时间进行比对。

若资源的最后修改时间大于If-Modified-Since，说明资源又被改动过，则响应整片资源内容，返回状态码200；

若资源的最后修改时间小于或等于If-Modified-Since，说明资源无新修改，则响应HTTP 304，告知浏览器继续使用所保存的cache。

[![](http://idiotsky.top/images3/http-summary-29.png)](http://idiotsky.top/images3/http-summary-29.png)

### Etag  /  If-None-Match

（优先级高于Last-Modified  /  If-Modified-Since）

Etag：

服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器决定）。

[![](http://idiotsky.top/images3/http-summary-30.png)](http://idiotsky.top/images3/http-summary-30.png)

If-None-Match：

再次请求服务器时，通过此字段通知服务器客户段缓存数据的唯一标识。

服务器收到请求后发现有头If-None-Match 则与被请求资源的唯一标识进行比对，

不同，说明资源又被改动过，则响应整片资源内容，返回状态码200；

相同，说明资源无新修改，则响应HTTP 304，告知浏览器继续使用所保存的cache。

[![](http://idiotsky.top/images3/http-summary-31.png)](http://idiotsky.top/images3/http-summary-31.png)

## 小结

对于强制缓存，服务器通知浏览器一个缓存时间，在缓存时间内，下次请求，直接用缓存，不在时间内，执行比较缓存策略。

对于比较缓存，将缓存信息中的Etag和Last-Modified通过请求发送给服务器，由服务器校验，返回304状态码时，浏览器直接使用缓存。

浏览器第一次请求：

[![](http://idiotsky.top/images3/http-summary-32.png)](http://idiotsky.top/images3/http-summary-32.png)

浏览器再次请求时：

[![](http://idiotsky.top/images3/http-summary-9.png)](http://idiotsky.top/images3/http-summary-9.png)

还有一张图总结下：

[![](http://idiotsky.top/images3/http-summary-33.jpg)](http://idiotsky.top/images3/http-summary-33.jpg)


# 提高 HTTP 性能

## 并行连接

通过多条 TCP 连接发起并发的 HTTP 请求。

## 持久连接 

重用 TCP 连接，以消除连接及关闭的时延。 持久连接（HTTP Persistent Connections），也称为 HTTP keep-alive 或者 HTTP connection reuse 。

在 HTTP/1.1 中，所有的连接默认都是持久连接。但是服务器端不一定都能够支持持久连接，所以除了服务端，客户端也需要支持持久连接。


## 管道化连接 

通过共享的 TCP 连接发起并发的 HTTP 请求。

持久连接使得多数请求以管线化（pipelining）方式发送成为可能。以前发送请求后需要等待并收到响应，才能发送下一个请求。管线化技术出现后，不用等待响应，直接发送下一个请求。

比如当请求一个包含 10 张图片的 HTML Web 页面，与挨个连接相比，用持久连接可以让请求更快结束。而管线化技术则比持久连接还要快。请求数越多，时间差就越明显。


## 复用的连接

交替传送请求和响应报文（实验阶段）。


# GET 和 POST 的区别

## 参数

GET 和 POST 的请求都能使用额外的参数，但是 GET 的参数是以查询字符串出现在 URL 中，而 POST 的参数存储在内容实体中(依旧是明文传输，只是和 GET 存放的位置不同罢了)。

GET 的传参方式相比于 POST 安全性较差，因为 GET 传的参数在 URL 中是可见的，可能会泄露私密信息。并且 GET 只支持 ASCII 字符，如果参数为中文则可能会出现乱码，而 POST 支持标准字符集。

````http
GET /test/demo_form.asp?name1=value1&name2=value2 HTTP/1.1
````

````http
POST /test/demo_form.asp HTTP/1.1
Host: w3schools.com
name1=value1&name2=value2
````

## 安全

安全的 HTTP 方法不会改变服务器状态，也就是说它只是可读的。

GET 方法是安全的，而 POST 却不是，因为 POST 的目的是传送实体主体内容，这个内容可能是用户上传的表单数据，上传成功之后，服务器可能把这个数据存储到数据库中，因此状态也就发生了改变。

安全的方法除了 GET 之外还有：HEAD、OPTIONS。

不安全的方法除了 POST 之外还有 PUT、DELETE。

## 幂等性

幂等的 HTTP 方法，同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用（统计用途除外）。在正确实现的条件下，GET，HEAD，PUT 和 DELETE 等方法都是幂等的，而 POST 方法不是。所有的安全方法也都是幂等的。

GET /pageX HTTP/1.1 是幂等的。连续调用多次，客户端接收到的结果都是一样的：

````http
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
````

POST /add_row HTTP/1.1 不是幂等的。如果调用多次，就会增加多行记录：

````http
POST /add_row HTTP/1.1
POST /add_row HTTP/1.1   -> Adds a 2nd row
POST /add_row HTTP/1.1   -> Adds a 3rd row
````

DELETE /idX/delete HTTP/1.1 是幂等的，即便是不同请求之间接收到的状态码不一样：

````http
DELETE /idX/delete HTTP/1.1   -> Returns 200 if idX exists
DELETE /idX/delete HTTP/1.1   -> Returns 404 as it just got deleted
DELETE /idX/delete HTTP/1.1   -> Returns 404
````

## 可缓存

如果要对响应进行缓存，需要满足以下条件：

1. 请求报文的 HTTP 方法本身是可缓存的，包括 GET 和 HEAD，但是 PUT 和 DELETE 不可缓存，POST 在多数情况下不可缓存的。
2. 响应报文的状态码是可缓存的，包括：200, 203, 204, 206, 300, 301, 404, 405, 410, 414, and 501。
3. 响应报文的 Cache-Control 首部字段没有指定不进行缓存。

## XMLHttpRequest

为了阐述 POST 和 GET 的另一个区别，需要先了解 XMLHttpRequest：

> XMLHttpRequest 是一个 API，它为客户端提供了在客户端和服务器之间传输数据的功能。它提供了一个通过 URL 来获取数据的简单方式，并且不会使整个页面刷新。这使得网页只更新一部分页面而不会打扰到用户。XMLHttpRequest 在 AJAX 中被大量使用。

在使用 XMLHttpRequest 的 POST 方法时，浏览器会先发送 Header 再发送 Data。但并不是所有浏览器会这么做，例如火狐就不会。

# HTTP/2.0

## HTTP/1.x 缺陷

HTTP/1.x 实现简单是以牺牲应用性能为代价的：

* 客户端需要使用多个连接才能实现并发和缩短延迟；
* 不会压缩请求和响应首部，从而导致不必要的网络流量；
* 不支持有效的资源优先级，致使底层 TCP 连接的利用率低下。

## 二进制分帧层

HTTP/2.0 将报文分成 HEADERS 帧和 DATA 帧，它们都是二进制格式的。

[![](http://idiotsky.top/images3/http-summary-52.png)](http://idiotsky.top/images3/http-summary-52.png)

在通信过程中，只会有一个 TCP 连接存在，它承载了任意数量的双向数据流（Stream）。一个数据流都有一个唯一标识符和可选的优先级信息，用于承载双向信息。消息（Message）是与逻辑请求或响应消息对应的完整的一系列帧。帧（Fram）是最小的通信单位，来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装。

[![](http://idiotsky.top/images3/http-summary-53.png)](http://idiotsky.top/images3/http-summary-53.png)

## 服务端推送

HTTP/2.0 在客户端请求一个资源时，会把相关的资源一起发送给客户端，客户端就不需要再次发起请求了。例如客户端请求 page.html 页面，服务端就把 script.js 和 style.css 等与之相关的资源一起发给客户端。

[![](http://idiotsky.top/images3/http-summary-54.png)](http://idiotsky.top/images3/http-summary-54.png)

## 首部压缩

HTTP/1.1 的首部带有大量信息，而且每次都要重复发送。HTTP/2.0 要求客户端和服务器同时维护和更新一个包含之前见过的首部字段表，从而避免了重复传输。不仅如此，HTTP/2.0 也使用 Huffman 编码对首部字段进行压缩。

[![](http://idiotsky.top/images3/http-summary-55.png)](http://idiotsky.top/images3/http-summary-55.png)

# HTTPS

HTTP 有以下安全性问题：

* 使用明文进行通信，内容可能会被窃听；
* 不验证通信方的身份，通信方的身份有可能遭遇伪装；
* 无法证明报文的完整性，报文有可能遭篡改。

HTTPs 并不是新协议，而是让 HTTP 先和 SSL（Secure Sockets Layer）通信，再由 SSL 和 TCP 通信。也就是说 HTTPs 使用了隧道进行通信。

通过使用 SSL，HTTPs 具有了加密（防窃听）、认证（防伪装）和完整性保护（防篡改）。

[![](http://idiotsky.top/images3/http-summary-56.jpg)](http://idiotsky.top/images3/http-summary-56.jpg)

## 加密

### 1. 对称密钥加密

对称密钥加密（Symmetric-Key Encryption），加密和解密使用同一密钥。

- 优点：运算速度快；
- 缺点：无法安全地将密钥传输给通信方。

[![](http://idiotsky.top/images3/http-summary-57.png)](http://idiotsky.top/images3/http-summary-57.png)

### 2.非对称密钥加密

非对称密钥加密，又称公开密钥加密（Public-Key Encryption），加密和解密使用不同的密钥。

公开密钥所有人都可以获得，通信发送方获得接收方的公开密钥之后，就可以使用公开密钥进行加密，接收方收到通信内容后使用私有密钥解密。

非对称密钥除了用来加密，还可以用来进行签名。因为私有密钥无法被其他人获取，因此通信发送方使用其私有密钥进行签名，通信接收方使用发送方的公开密钥对签名进行解密，就能判断这个签名是否正确。

- 优点：可以更安全地将公开密钥传输给通信发送方；
- 缺点：运算速度慢。

[![](http://idiotsky.top/images3/http-summary-58.png)](http://idiotsky.top/images3/http-summary-58.png)

### 3. HTTPs 采用的加密方式

HTTPs 采用混合的加密机制，使用非对称密钥加密用于传输对称密钥来保证安全性，之后使用对称密钥加密进行通信来保证效率。（下图中的 Session Key 就是对称密钥）

[![](http://idiotsky.top/images3/http-summary-59.png)](http://idiotsky.top/images3/http-summary-59.png)

## 认证

通过使用  **证书**  来对通信方进行认证。

数字证书认证机构（CA，Certificate Authority）是客户端与服务器双方都可信赖的第三方机构。

服务器的运营人员向 CA 提出公开密钥的申请，CA 在判明提出申请者的身份之后，会对已申请的公开密钥做数字签名，然后分配这个已签名的公开密钥，并将该公开密钥放入公开密钥证书后绑定在一起。

进行 HTTPs 通信时，服务器会把证书发送给客户端。客户端取得其中的公开密钥之后，先使用数字签名进行验证，如果验证通过，就可以开始通信了。

通信开始时，客户端需要使用服务器的公开密钥将自己的私有密钥传输给服务器，之后再进行对称密钥加密。

[![](http://idiotsky.top/images3/http-summary-60.png)](http://idiotsky.top/images3/http-summary-60.png)

## 完整性保护

SSL 提供报文摘要功能来进行完整性保护。

HTTP 也提供了 MD5 报文摘要功能，但不是安全的。例如报文内容被篡改之后，同时重新计算 MD5 的值，通信接收方是无法意识到发生了篡改。

HTTPs 的报文摘要功能之所以安全，是因为它结合了加密和认证这两个操作。试想一下，加密之后的报文，遭到篡改之后，也很难重新计算报文摘要，因为无法轻易获取明文。

## HTTPs 的缺点

- 因为需要进行加密解密等过程，因此速度会更慢；
- 需要支付证书授权的高费用。


# 浏览器同源政策及其规避方法

1995年，同源政策由 Netscape 公司引入浏览器。目前，所有浏览器都实行这个政策。

最初，它的含义是指，A网页设置的 Cookie，B网页不能打开，除非这两个网页"同源"。所谓"同源"指的是"三个相同"。

* 协议相同
* 域名相同
* 端口相同

举例来说，http://www.example.com/dir/page.html  这个网址，协议是http://，域名是www.example.com，端口是80（默认端口可以省略）。它的同源情况如下。

* http://www.example.com/dir2/other.html  同源
* http://example.com/dir/other.html 不同源（域名不同）
* http://v2.www.example.com/dir/other.html 不同源（域名不同）
* http://www.example.com:81/dir/other.html 不同源（端口不同）

同源政策的目的，是为了保证用户信息的安全，防止恶意的网站窃取数据。
设想这样一种情况：A网站是一家银行，用户登录以后，又去浏览其他网站。如果其他网站可以读取A网站的 Cookie，会发生什么？
很显然，如果 Cookie 包含隐私（比如存款总额），这些信息就会泄漏。更可怕的是，Cookie 往往用来保存用户的登录状态，如果用户没有退出登录，其他网站就可以冒充用户，为所欲为。因为浏览器同时还规定，提交表单不受同源政策的限制。
由此可见，"同源政策"是必需的，否则 Cookie 可以共享，互联网就毫无安全可言了。

随着互联网的发展，"同源政策"越来越严格。目前，如果非同源，共有三种行为受到限制。
1. Cookie、LocalStorage 和 IndexDB 无法读取。
2. DOM 无法获得。
3. AJAX 请求不能发送。

虽然这些限制是必要的，但是有时很不方便，合理的用途也受到影响。下面，我将详细介绍，如何规避上面三种限制。

## Cookie

Cookie 是服务器写入浏览器的一小段信息，只有同源的网页才能共享。但是，两个网页一级域名相同，只是二级域名不同，浏览器允许通过设置document.domain共享 Cookie。
举例来说，A网页是http://w1.example.com/a.html，B网页是http://w2.example.com/b.html，那么只要设置相同的document.domain，两个网页就可以共享Cookie。

````
document.domain = 'example.com';
````

现在，A网页通过脚本设置一个 Cookie。

````
document.cookie = "test1=hello";
````

B网页就可以读到这个 Cookie。

````
var allCookie = document.cookie;
````

注意，这种方法只适用于 Cookie 和 iframe 窗口，LocalStorage 和 IndexDB 无法通过这种方法，规避同源政策，而要使用下文介绍的PostMessage API。
另外，服务器也可以在设置Cookie的时候，指定Cookie的所属域名为一级域名，比如.example.com。

````
Set-Cookie: key=value; domain=.example.com; path=/
````

这样的话，二级域名和三级域名不用做任何设置，都可以读取这个Cookie。

## iframe

如果两个网页不同源，就无法拿到对方的DOM。典型的例子是iframe窗口和window.open方法打开的窗口，它们与父窗口无法通信。
比如，父窗口运行下面的命令，如果iframe窗口不是同源，就会报错。

````
document.getElementById("myIFrame").contentWindow.document
// Uncaught DOMException: Blocked a frame from accessing a cross-origin frame.
````

上面命令中，父窗口想获取子窗口的DOM，因为跨源导致报错。

反之亦然，子窗口获取主窗口的DOM也会报错。

````
window.parent.document.body
// 报错
````

如果两个窗口一级域名相同，只是二级域名不同，那么设置上一节介绍的document.domain属性，就可以规避同源政策，拿到DOM。
对于完全不同源的网站，目前有三种方法，可以解决跨域窗口的通信问题。

* 片段识别符（fragment identifier）
* window.name
* 跨文档通信API（Cross-document messaging）

### 片段识别符
片段标识符（fragment identifier）指的是，URL的#号后面的部分，比如 http://example.com/x.html#fragment 的 `#fragment`。如果只是改变片段标识符，页面不会重新刷新。

父窗口可以把信息，写入子窗口的片段标识符。

````
var src = originURL + '#' + data;
document.getElementById('myIFrame').src = src;
````

子窗口通过监听hashchange事件得到通知。

````
window.onhashchange = checkMessage;

function checkMessage() {
  var message = window.location.hash;
  // ...
}
````

同样的，子窗口也可以改变父窗口的片段标识符。

````
parent.location.href= target + "#" + hash;
````

### window.name

浏览器窗口有window.name属性。这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页设置了这个属性，后一个网页可以读取它。

父窗口先打开一个子窗口，载入一个不同源的网页，该网页将信息写入window.name属性。

````
window.name = data;
````

接着，子窗口跳回一个与主窗口同域的网址。

````
location = 'http://parent.url.com/xxx.html';
````

然后，主窗口就可以读取子窗口的window.name了。

````
var data = document.getElementById('myFrame').contentWindow.name;
````

这种方法的优点是，window.name容量很大，可以放置非常长的字符串；缺点是必须监听子窗口window.name属性的变化，影响网页性能。

### window.postMessage

上面两种方法都属于破解，HTML5为了解决这个问题，引入了一个全新的API：跨文档通信 API（Cross-document messaging）。
这个API为window对象新增了一个window.postMessage方法，允许跨窗口通信，不论这两个窗口是否同源。
举例来说，父窗口`http://aaa.com`向子窗口`http://bbb.com`发消息，调用postMessage方法就可以了。

````
var popup = window.open('http://bbb.com', 'title');
popup.postMessage('Hello World!', 'http://bbb.com');
````

postMessage方法的第一个参数是具体的信息内容，第二个参数是接收消息的窗口的源（origin），即"协议 + 域名 + 端口"。也可以设为*，表示不限制域名，向所有窗口发送。

子窗口向父窗口发送消息的写法类似。

````
window.opener.postMessage('Nice to see you', 'http://aaa.com');
````

父窗口和子窗口都可以通过message事件，监听对方的消息。

````
window.addEventListener('message', function(e) {
  console.log(e.data);
},false);
````

message事件的事件对象event，提供以下三个属性。

* event.source：发送消息的窗口
* event.origin: 消息发向的网址
* event.data: 消息内容

下面的例子是，子窗口通过event.source属性引用父窗口，然后发送消息。

````
window.addEventListener('message', receiveMessage);
function receiveMessage(event) {
  event.source.postMessage('Nice to see you!', '*');
}
````

event.origin属性可以过滤不是发给本窗口的消息。

````
window.addEventListener('message', receiveMessage);
function receiveMessage(event) {
  if (event.origin !== 'http://aaa.com') return;
  if (event.data === 'Hello World') {
      event.source.postMessage('Hello', event.origin);
  } else {
    console.log(event.data);
  }
}
````

### LocalStorage

通过window.postMessage，读写其他窗口的 LocalStorage 也成为了可能。

下面是一个例子，主窗口写入iframe子窗口的localStorage。

````
window.onmessage = function(e) {
  if (e.origin !== 'http://bbb.com') {
    return;
  }
  var payload = JSON.parse(e.data);
  localStorage.setItem(payload.key, JSON.stringify(payload.data));
};
````

上面代码中，子窗口将父窗口发来的消息，写入自己的LocalStorage。

父窗口发送消息的代码如下。

````
var win = document.getElementsByTagName('iframe')[0].contentWindow;
var obj = { name: 'Jack' };
win.postMessage(JSON.stringify({key: 'storage', data: obj}), 'http://bbb.com');
````

加强版的子窗口接收消息的代码如下。

````
window.onmessage = function(e) {
  if (e.origin !== 'http://bbb.com') return;
  var payload = JSON.parse(e.data);
  switch (payload.method) {
    case 'set':
      localStorage.setItem(payload.key, JSON.stringify(payload.data));
      break;
    case 'get':
      var parent = window.parent;
      var data = localStorage.getItem(payload.key);
      parent.postMessage(data, 'http://aaa.com');
      break;
    case 'remove':
      localStorage.removeItem(payload.key);
      break;
  }
};
````

加强版的父窗口发送消息代码如下。

````
var win = document.getElementsByTagName('iframe')[0].contentWindow;
var obj = { name: 'Jack' };
// 存入对象
win.postMessage(JSON.stringify({key: 'storage', method: 'set', data: obj}), 'http://bbb.com');
// 读取对象
win.postMessage(JSON.stringify({key: 'storage', method: "get"}), "*");
window.onmessage = function(e) {
  if (e.origin != 'http://aaa.com') return;
  // "Jack"
  console.log(JSON.parse(e.data).name);
};
````

## AJAX

同源政策规定，AJAX请求只能发给同源的网址，否则就报错。

除了架设服务器代理（浏览器请求同源服务器，再由后者请求外部服务），有三种方法规避这个限制。

* JSONP
* WebSocket
* CORS

### JSONP
JSONP是服务器与客户端跨源通信的常用方法。最大特点就是简单适用，老式浏览器全部支持，服务器改造非常小。

它的基本思想是，网页通过添加一个`<script>`元素，向服务器请求JSON数据，这种做法不受同源政策限制；服务器收到请求后，将数据放在一个指定名字的回调函数里传回来。

首先，网页动态插入`<script>`元素，由它向跨源网址发出请求。

````
function addScriptTag(src) {
  var script = document.createElement('script');
  script.setAttribute("type","text/javascript");
  script.src = src;
  document.body.appendChild(script);
}

window.onload = function () {
  addScriptTag('http://example.com/ip?callback=foo');
}

function foo(data) {
  console.log('Your public IP address is: ' + data.ip);
};
````

上面代码通过动态添加`<script>`元素，向服务器example.com发出请求。注意，该请求的查询字符串有一个callback参数，用来指定回调函数的名字，这对于JSONP是必需的。

服务器收到这个请求以后，会将数据放在回调函数的参数位置返回。

````
foo({
  "ip": "8.8.8.8"
});
````

由于`<script>`元素请求的脚本，直接作为代码运行。这时，只要浏览器定义了foo函数，该函数就会立即调用。作为参数的JSON数据被视为JavaScript对象，而不是字符串，因此避免了使用JSON.parse的步骤。

### WebSocket

WebSocket是一种通信协议，使用ws://（非加密）和wss://（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信。
下面是一个例子，浏览器发出的WebSocket请求的头信息.

````
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
````

上面代码中，有一个字段是Origin，表示该请求的请求源（origin），即发自哪个域名。

正是因为有了Origin这个字段，所以WebSocket才没有实行同源政策。因为服务器可以根据这个字段，判断是否许可本次通信。如果该域名在白名单内，服务器就会做出如下回应。

````
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
````

## CORS

CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。

本文详细介绍CORS的内部机制。

### 简介

CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。

整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

### 两种请求

浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

只要同时满足以下两大条件，就属于简单请求。

1. 请求方法是以下三种方法之一：
HEAD
GET
POST

2. HTTP的头信息不超出以下几种字段：
Accept
Accept-Language
Content-Language
Last-Event-ID
Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

凡是不同时满足上面两个条件，就属于非简单请求。
浏览器对这两种请求的处理，是不一样的。

### 简单请求

#### 基本流程

对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。

下面是一个例子，浏览器发现这次跨源AJAX请求是简单请求，就自动在头信息之中，添加一个Origin字段。

````
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
````

上面的头信息中，Origin字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。

如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段（详见下文），就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。

````
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
````

上面的头信息之中，有三个与CORS请求相关的字段，都以`Access-Control-`开头。

1. Access-Control-Allow-Origin
该字段是必须的。它的值要么是请求时Origin字段的值，要么是一个*，表示接受任意域名的请求。
2. Access-Control-Allow-Credentials
该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。
3. Access-Control-Expose-Headers
该字段可选。CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。上面的例子指定，getResponseHeader('FooBar')可以返回FooBar字段的值。

#### withCredentials 属性

上面说到，CORS请求默认不发送Cookie和HTTP认证信息。如果要把Cookie发到服务器，一方面要服务器同意，指定Access-Control-Allow-Credentials字段。

````
Access-Control-Allow-Credentials: true
````

另一方面，开发者必须在AJAX请求中打开withCredentials属性。

````
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;
````

否则，即使服务器同意发送Cookie，浏览器也不会发送。或者，服务器要求设置Cookie，浏览器也不会处理。
但是，如果省略withCredentials设置，有的浏览器还是会一起发送Cookie。这时，可以显式关闭withCredentials。

````
xhr.withCredentials = false;
````
需要注意的是，如果要发送Cookie，Access-Control-Allow-Origin就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的document.cookie也无法读取服务器域名下的Cookie。

### 非简单请求

#### 预检请求

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

下面是一段浏览器的JavaScript脚本。

````
var url = 'http://api.alice.com/cors';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
````

上面代码中，HTTP请求的方法是PUT，并且发送一个自定义头信息X-Custom-Header。

浏览器发现，这是一个非简单请求，就自动发出一个"预检"请求，要求服务器确认可以这样请求。下面是这个"预检"请求的HTTP头信息。

````
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
````

"预检"请求用的请求方法是OPTIONS，表示这个请求是用来询问的。头信息里面，关键字段是Origin，表示请求来自哪个源。

除了Origin字段，"预检"请求的头信息包括两个特殊字段。

1. Access-Control-Request-Method
该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是PUT。
2. Access-Control-Request-Headers
该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段，上例是X-Custom-Header。

#### 预检请求的回应

服务器收到"预检"请求以后，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段以后，确认允许跨源请求，就可以做出回应。

````
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
````

上面的HTTP回应中，关键的是Access-Control-Allow-Origin字段，表示http://api.bob.com可以请求数据。该字段也可以设为星号，表示同意任意跨源请求。

````
Access-Control-Allow-Origin: *
````

如果浏览器否定了"预检"请求，会返回一个正常的HTTP回应，但是没有任何CORS相关的头信息字段。这时，浏览器就会认定，服务器不同意预检请求，因此触发一个错误，被XMLHttpRequest对象的onerror回调函数捕获。控制台会打印出如下的报错信息。

````
XMLHttpRequest cannot load http://api.alice.com.
Origin http://api.bob.com is not allowed by Access-Control-Allow-Origin.
````

服务器回应的其他CORS相关字段如下。

````
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
````

1. Access-Control-Allow-Methods
该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。
2. Access-Control-Allow-Headers
如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。
3. Access-Control-Allow-Credentials
该字段与简单请求时的含义相同。
4. Access-Control-Max-Age
该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求。

#### 浏览器的正常请求和回应

一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个Origin头信息字段。服务器的回应，也都会有一个Access-Control-Allow-Origin头信息字段。

下面是"预检"请求之后，浏览器的正常CORS请求。

````
PUT /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
X-Custom-Header: value
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
````

上面头信息的Origin字段是浏览器自动添加的。

下面是服务器正常的回应。

````
Access-Control-Allow-Origin: http://api.bob.com
Content-Type: text/html; charset=utf-8
````

上面头信息中，Access-Control-Allow-Origin字段是每次回应都必定包含的。

### 与JSONP的比较

CORS与JSONP的使用目的相同，但是比JSONP更强大。

JSONP只支持GET请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。

# 理解HTTP幂等性

基于HTTP协议的Web API是时下最为流行的一种分布式服务提供方式。无论是在大型互联网应用还是企业级架构中，我们都见到了越来越多的SOA或RESTful的Web API。为什么Web API如此流行呢？我认为很大程度上应归功于简单有效的HTTP协议。HTTP协议是一种分布式的面向资源的网络应用层协议，无论是服务器端提供Web服务，还是客户端消费Web服务都非常简单。再加上浏览器、Javascript、AJAX、JSON以及HTML5等技术和工具的发展，互联网应用架构设计表现出了从传统的PHP、JSP、ASP.NET等服务器端动态网页向Web API + RIA（富互联网应用）过渡的趋势。Web API专注于提供业务服务，RIA专注于用户界面和交互设计，从此两个领域的分工更加明晰。在这种趋势下，Web API设计将成为服务器端程序员的必修课。然而，正如简单的Java语言并不意味着高质量的Java程序，简单的HTTP协议也不意味着高质量的Web API。要想设计出高质量的Web API，还需要深入理解分布式系统及HTTP协议的特性。

## 幂等性定义

本文所要探讨的正是HTTP协议涉及到的一种重要性质：幂等性(Idempotence)。在HTTP/1.1规范中幂等性的定义是：

> Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.

从定义上看，HTTP方法的幂等性是指一次和多次请求某一个资源应该具有同样的副作用。幂等性属于语义范畴，正如编译器只能帮助检查语法错误一样，HTTP规范也没有办法通过消息格式等语法手段来定义它，这可能是它不太受到重视的原因之一。但实际上，幂等性是分布式系统设计中十分重要的概念，而HTTP的分布式本质也决定了它在HTTP中具有重要地位。

## 分布式事务 vs 幂等设计

为什么需要幂等性呢？我们先从一个例子说起，假设有一个从账户取钱的远程API（可以是HTTP的，也可以不是），我们暂时用类函数的方式记为：

````
bool withdraw(account_id, amount)
````

withdraw的语义是从account_id对应的账户中扣除amount数额的钱；如果扣除成功则返回true，账户余额减少amount；如果扣除失败则返回false，账户余额不变。值得注意的是：和本地环境相比，我们不能轻易假设分布式环境的可靠性。一种典型的情况是withdraw请求已经被服务器端正确处理，但服务器端的返回结果由于网络等原因被掉丢了，导致客户端无法得知处理结果。如果是在网页上，一些不恰当的设计可能会使用户认为上一次操作失败了，然后刷新页面，这就导致了withdraw被调用两次，账户也被多扣了一次钱。如下图所示：

[![](http://idiotsky.top/images3/http-summary-50.png)](http://idiotsky.top/images3/http-summary-50.png)

这个问题的解决方案一是采用分布式事务，通过引入支持分布式事务的中间件来保证withdraw功能的事务性。分布式事务的优点是对于调用者很简单，复杂性都交给了中间件来管理。缺点则是一方面架构太重量级，容易被绑在特定的中间件上，不利于异构系统的集成；另一方面分布式事务虽然能保证事务的ACID性质，而但却无法提供性能和可用性的保证。

另一种更轻量级的解决方案是幂等设计。我们可以通过一些技巧把withdraw变成幂等的，比如：

int create_ticket() 
bool idempotent_withdraw(ticket_id, account_id, amount)
create_ticket的语义是获取一个服务器端生成的唯一的处理号ticket_id，它将用于标识后续的操作。idempotent_withdraw和withdraw的区别在于关联了一个ticket_id，一个ticket_id表示的操作至多只会被处理一次，每次调用都将返回第一次调用时的处理结果。这样，idempotent_withdraw就符合幂等性了，客户端就可以放心地多次调用。

基于幂等性的解决方案中一个完整的取钱流程被分解成了两个步骤：1.调用create_ticket()获取ticket_id；2.调用idempotent_withdraw(ticket_id, account_id, amount)。虽然create_ticket不是幂等的，但在这种设计下，它对系统状态的影响可以忽略，加上idempotent_withdraw是幂等的，所以任何一步由于网络等原因失败或超时，客户端都可以重试，直到获得结果。如下图所示：

[![](http://idiotsky.top/images3/http-summary-51.png)](http://idiotsky.top/images3/http-summary-51.png)

## HTTP的幂等性

HTTP协议本身是一种面向资源的应用层协议，但对HTTP协议的使用实际上存在着两种不同的方式：一种是RESTful的，它把HTTP当成应用层协议，比较忠实地遵守了HTTP协议的各种规定；另一种是SOA的，它并没有完全把HTTP当成应用层协议，而是把HTTP协议作为了传输层协议，然后在HTTP之上建立了自己的应用层协议。本文所讨论的HTTP幂等性主要针对RESTful风格的，不过正如上一节所看到的那样，幂等性并不属于特定的协议，它是分布式系统的一种特性；所以，不论是SOA还是RESTful的Web API设计都应该考虑幂等性。下面将介绍HTTP GET、DELETE、PUT、POST四种主要方法的语义和幂等性。

HTTP GET方法用于获取资源，不应有副作用，所以是幂等的。比如：GET http://www.bank.com/account/123456，不会改变资源的状态，不论调用一次还是N次都没有副作用。请注意，这里强调的是一次和N次具有相同的副作用，而不是每次GET的结果相同。GET http://www.news.com/latest-news这个HTTP请求可能会每次得到不同的结果，但它本身并没有产生任何副作用，因而是满足幂等性的。

HTTP DELETE方法用于删除资源，有副作用，但它应该满足幂等性。比如：DELETE http://www.forum.com/article/4231，调用一次和N次对系统产生的副作用是相同的，即删掉id为4231的帖子；因此，调用者可以多次调用或刷新页面而不必担心引起错误。

比较容易混淆的是HTTP POST和PUT。POST和PUT的区别容易被简单地误认为“POST表示创建资源，PUT表示更新资源”；而实际上，二者均可用于创建资源，更为本质的差别是在幂等性方面。在HTTP规范中对POST和PUT是这样定义的：

> The POST method is used to request that the origin server accept the entity enclosed in the request as a new subordinate of the resource identified by the Request-URI in the Request-Line ...... If a resource has been created on the origin server, the response SHOULD be 201 (Created) and contain an entity which describes the status of the request and refers to the new resource, and a Location header.

>The PUT method requests that the enclosed entity be stored under the supplied Request-URI. If the Request-URI refers to an already existing resource, the enclosed entity SHOULD be considered as a modified version of the one residing on the origin server. If the Request-URI does not point to an existing resource, and that URI is capable of being defined as a new resource by the requesting user agent, the origin server can create the resource with that URI.

POST所对应的URI并非创建的资源本身，而是资源的接收者。比如：POST http://www.forum.com/articles 的语义是在http://www.forum.com/articles 下创建一篇帖子，HTTP响应中应包含帖子的创建状态以及帖子的URI。两次相同的POST请求会在服务器端创建两份资源，它们具有不同的URI；所以，POST方法不具备幂等性。而PUT所对应的URI是要创建或更新的资源本身。比如：PUT http://www.forum/articles/4231 的语义是创建或更新ID为4231的帖子。对同一URI进行多次PUT的副作用和一次PUT是相同的；因此，PUT方法具有幂等性。

在介绍了几种操作的语义和幂等性之后，我们来看看如何通过Web API的形式实现前面所提到的取款功能。很简单，用POST /tickets来实现create_ticket；用PUT /accounts/account_id/ticket_id?amount=xxx来实现idempotent_withdraw。值得注意的是严格来讲amount参数不应该作为URI的一部分，真正的URI应该是/accounts/account_id/ticket_id，而amount应该放在请求的body中。这种模式可以应用于很多场合，比如：论坛网站中防止意外的重复发帖。

## 小结

上面简单介绍了幂等性的概念，用幂等设计取代分布式事务的方法，以及HTTP主要方法的语义和幂等性特征。其实，如果要追根溯源，幂等性是数学中的一个概念，表达的是N次变换与1次变换的结果相同，有兴趣的读者可以从Wikipedia上进一步了解。



ref

http://www.ruanyifeng.com/blog/2016/04/cors.html

http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html

https://github.com/halfrost/Halfrost-Field/blob/master/contents/Protocol/HTTP.md

https://www.cnblogs.com/chenqf/p/6386163.html

http://www.cnblogs.com/weidagang2046/archive/2011/06/04/2063696.html