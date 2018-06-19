---
title: HTTP 总结
date: 2018-05-22 22:51:18
tags: [http,protocol]
categories: protocol
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
|  |  |  | |
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

------------------------------------------------------------

## RFC 2616 状态码

| 状态码 | 类别 | 原因短语 |含义||
| :---: | :---: | :---: |:---: |:---:|
| 100 | Informational（信息性状态码） | Continue（继续） |收到了请求的起始部分，客户端应该继续请求。|❤|
| 101 | Informational（信息性状态码） | Switching Protocols（切换协议）|服务器正根据客户端的指示将协议切换成 Update 首部列出的协议。|❤|
|||||
| 200 | Success（成功状态码）| OK |服务器已成功处理请求|❤|
| 201 | Success（成功状态码）|Created（已创建）| 对那些要服务器创建对象的请求来说，资源已创建完毕|
| 202 | Success（成功状态码）|Accepted（已接受）| 请求已接受，但服务器尚未处理|
| 203 | Success（成功状态码）|Non-Authoritative Information（非权威信息）| 服务器已将事务成功处理，只是实体首部包含的信息不是来自原始服务器，而是来自资源的副本|
| 204 | Success（成功状态码）|No Content（没有内容）| 响应报文包含一些首部和一个状态行，**但不包含实体的主体内容**，**一般在只需要从客户端往服务器发送信息，而对客户端不需要发送新信息内容的情况下使用**|❤|
| 205 | Success（成功状态码）|Reset Content（重置内容）| 另一个主要用于浏览器的代码。意思是浏览器应该重置当前页面上所有的 HTML 表单 |
| 206 | Success（成功状态码） |Partial Content（部分内容）| 成功执行了一个部分或者 Range (范围)请求，客户端可以通过一些特殊的首部来获取部分或某个范围内的文档<br>**响应报文中包含由 Content-Range、Date、以及 ETag 或者 Content-Location 指定范围的实体内容**|❤|
|||||
|300| Redirection（重定向状态码） |Multiple Choices（多项选择）| 客户端请求了实际指向多个资源的 URL。这个代码是和一个选项列表一起返回的，然后用户就可以选择他希望使用的选项了。服务器可以在 Location 首部包含首选 URL| 
|301| Redirection（重定向状态码） | Moved Permanently（永久移除）| **永久性重定向**，请求的 URL 已移走。响应中应该包含一个 Location URL，说明资源现在所处的位置|❤|
|302| Redirection（重定向状态码）| Found（已找到）| **临时性重定向**，与状态码 301 类似， 但这里的移除是临时的。客户端应该用 Location 首部给出的 URL 对资源进行临时定位|❤|
|303| Redirection（重定向状态码）| See Other（参见其他）| 告诉客户端应该用另一个 URL 获取资源。这个新的 URL 位于响应报文的 Location 首部。303 状态码 和 302 状态码有相同的功能，**但是 303 明确表示客户端应采用 GET 方法获取资源**。|❤|
||||当 301、302、303 响应状态码返回时，几乎所有的浏览器都会把 POST 改成 GET，并删除请求报文内的主体，之后请求会自动再次发送。<br>301、302 标准是禁止将 POST 方法改变成 GET 方法的，但实际使用时大家都会这么做||
|304| Redirection（重定向状态码）| Not Modified（未修改）| 该状态码表示客户端发送附带条件的请求时，服务器允许请求访问资源，但因发生请求未满足条件的情况后，直接返回 304 Not Modified（服务器端资源未改变，可直接使用客户端未过期的缓存）304 状态码返回时，不包含任何响应的主体部分。**304 虽然被划分在 3XX 类别中，但是和重定向一点关系也没有**|❤|
||||（附带条件的请求是指采用 GET 方法的请求报文中包含 If-Match，If-Modified-Since,If-None-Match，If-Range，If-Unmodified-Since 中任一首部）||
|305| Redirection（重定向状态码）| Use Proxy（使用代理）| 必须通过代理访问 资源，代理的位置是在 Location 首部中给出的|
|306|（未使用）||这个状态码当前并未使用|
|307| Redirection（重定向状态码）| Temporary Redirect（临时重定向）| 和状态码 302 类似。但客户端应该用 Location 首部给出的 URL 对资源进行临时定位。<br>307 会遵守浏览器标准，不会从 POST 变成 GET|❤|
|||||
|400|Client Error（客户端错误状态码）| Bad request（坏请求）| 告诉客户端它发送了一条异常请求|❤|
|401|Client Error（客户端错误状态码）| Unauthorized（未授权）| 与适当的首部一起返回，在客户端获得资源访问权之前，请它进行身份认证|❤|
|402|Client Error（客户端错误状态码）| Payment Required（要求付款）| 当前此状态码并未使用，是为未来使用预留的 |
|403|Client Error（客户端错误状态码）| Forbidden（禁止）| 服务器拒绝了请求|❤| 
|404|Client Error（客户端错误状态码）| Not Found（未找到）| 服务器无法找到 所请求的 URL|❤|
|405|Client Error（客户端错误状态码）| Method Not Allowed（不允许使用的方法）|请求中有一个所请求的 URI 不支持的方法。响应中应该包含一个 Allow 首部，以告知客户端所请求的资源支持使用哪些方法| 
|406|Client Error（客户端错误状态码）| Not Acceptable（无法接受）| 客户端可以指定一些参数来说明希望接受哪些类型的实体。服务器没有资源与客户端可接受的 URL 相匹配时可使用此代码| 
|407|Client Error（客户端错误状态码）| Proxy Authentication Required（要求进行代理认证）|和状态码 401 类似，但用于需要进行资源认证的代理服务器|
|408|Client Error（客户端错误状态码）| Request Timeout（请求超时）| 如果客户端完成其请求时花费的时间太长，服务器可以回送这个状态码并关闭连接 |
|409|Client Error（客户端错误状态码）| Conflict（ 冲突）| 发出的请求在资源上造成了一些冲突| 
|410|Client Error（客户端错误状态码）| Gone（消失了）| 除了服务器曾持有这些资源之外，与状态码 404 类似 |
|411|Client Error（客户端错误状态码）| Length Required（要求长度指示）| 服务器要求在请求报文中包含 Content- Length 首部时会使用这个代码。发起的请求中若没有 Content-Length 首部，服务器 是不会接受此资源请求的| 
|412|Client Error（客户端错误状态码）|Precondition Failed（先决条件失败）| 如果客户端发起了一个条件请求， 如果服务器无法满足其中的某个条件，就返回这个响应码| 
|413|Client Error（客户端错误状态码）| Request Entity Too Large（请求实体太大）| 客户端发送的实体主体部分比 服务器能够或者希望处理的要大|
|414|Client Error（客户端错误状态码）| Request URI Too Long（请求 URI 太长）| 客户端发送的请求所携带的请求 URL 超过了服务器能够或者希望处理的长度|
|415 |Client Error（客户端错误状态码）|Unsupported Media Type（不支持的媒体类型）| 服务器无法理解或不支持客户端所发送的实体的内容类型| 
|416 |Client Error（客户端错误状态码）|Requested Range Not Satisfiable（所请求的范围未得到满足）| 请求报文请求的是某范围内的指定资源，但那个范围无效，或者未得到满足 |
|417|Client Error（客户端错误状态码）| Expectation Failed（无法满足期望）| 请求的 Expect 首部包含了一个预期内容，但服务器无法满足||
|||||
|500|Server Error（服务器错误状态码）| Internal Server Error（内部服务器错误）| 服务器遇到了一个错误，使其无法为请求提供服务|❤|
|501 |Server Error（服务器错误状态码）|Not Implemented（未实现）| 服务器无法满足客户端请求的某个功能 |
|502 |Server Error（服务器错误状态码）|Bad Gateway（网关故障）| 作为代理或网关使用的服务器遇到了来自响应链中上游的无效响应 |
|503|Server Error（服务器错误状态码）| Service Unavailable（未提供此服务）| 服务器目前无法为请求提供服务，但过一段时间就可以恢复服务|❤|
|504|Server Error（服务器错误状态码） |Gateway Timeout（网关超时）| 与状态码 408 类似，但是响应来自网关或代理，此网关或代理在等待另一台服务器的响应时出现了超时 |
|505|Server Error（服务器错误状态码）| HTTP Version Not Supported（不支持的 HTTP 版本）| 服务器收到的请求是以它不支持或不愿支持的协议版本表示的|

>在 RFC2616 中定义了 40 种 HTTP 状态码，webDAV ( Web-based Distributed Authoring and Versioning，基于万维网的分布式创作和版本控制)在 RFC4918 和 RFC5842 中，定义了一些特殊的状态码，在 RFC2518、RFC2817、RFC2295、RFC2774、RFC6585 中还额外定义了一些附加的 HTTP 状态码。总共有 60+ 种。具体链接可以见 [HTTP状态码 (wikipedia)](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)


webDAV 新增状态码

| 状态码 | 类别 | 原因短语 |含义||
| :---: | :---: | :---: |:---: |:---:|
| 102 | Informational（信息性状态码） | Processing（处理中）|可正常处理请求，但目前是处理中状态。WebDAV请求可能包含许多涉及文件操作的子请求，需要很长时间才能完成请求。该代码表示​​服务器已经收到并正在处理请求，但无响应可用。这样可以防止客户端超时，并假设请求丢失。||
| 207 | Success（成功状态码） | Multi-Status（多种状态）| 存在多种状态。代表之后的消息体将是一个 XML 消息，并且可能依照之前子请求数量的不同，包含一系列独立的响应代码。||
| 208 | Success（成功状态码） | Already Reported（已经响应）| DAV绑定的成员已经在（多状态）响应之前的部分被列举，且未被再次包含。||
|422|Client Error（客户端错误状态码）| Unprocessable Entity（不可处理的实体）| 格式正确，内容有误，无法处理响应||
|423|Client Error（客户端错误状态码）| Locked（被锁定）| 资源已被加锁||
|424|Client Error（客户端错误状态码）| Failed Dependency（失败的依赖）| 处理与某请求关联的请求失败，因为不再维持依赖关系。||
|507|Server Error（服务器错误状态码）| Insufficient Storage（存储空间不足）| 服务器无法存储完成请求所必须的内容。这个状况被认为是临时的。||
|508|Server Error（服务器错误状态码）| Loop Detected（检测到环）| 服务器在处理请求时陷入死循环。|

# MIME 媒体内容


HTTP 仔细地给每种要通过 Web 传输的对象都打上了名为 MIME 类型（MIME type）的数据格式标签。最初设计 MIME（Multipurpose Internet Mail Extension，多用途因特网邮件扩展）是为了解决在不同的电子邮件系统之间搬移报文时存在的问题。MIME 在电子邮件系统中工作得非常好，因此 HTTP 也采纳了它，用它来描述并标记多媒体内容。


RFC2045，“ MIME: Format of Internet Message Bodies”（“ MIME：因特网报文主体的格式”）


常见的主 MIME 类型

| 类型| 描述 | 
| :---: | :---: |
|application |应用程序特有的内容格式（离散类型）|
|audio| 音频格式（离散类型） |
|chemical |化学数据集（离散 IETF 扩展类型）| 
|image| 图片格式（离散类型） |
|message |报文格式（复合类型）| 
|model| 三维模型格式（离散 IETF 扩展类型）| 
|multipart |多部分对象集合（复合类型）|
| text |文本格式（离散类型）| 
|video |视频电影格式（离散类型）|




# HTTP 报文结构

[![](http://idiotsky.me/images3/http-summary-1.png)](http://idiotsky.me/images3/http-summary-1.png)


[![](http://idiotsky.me/images3/http-summary-2.png)](http://idiotsky.me/images3/http-summary-2.png)


[![](http://idiotsky.me/images3/http-summary-3.png)](http://idiotsky.me/images3/http-summary-3.png)

[![](http://idiotsky.me/images3/http-summary-4.png)](http://idiotsky.me/images3/http-summary-4.png)


Response Headers:

[![](http://idiotsky.me/images3/http-summary-5.png)](http://idiotsky.me/images3/http-summary-5.png)


Request Headers:

[![](http://idiotsky.me/images3/http-summary-6.png)](http://idiotsky.me/images3/http-summary-6.png)


请求报文是由请求方法，请求 URI，协议版本，可选请求首部字段和内容实体构成的。

响应报文基本上由协议版本，状态码（表示请求成功与失败的数字代码），用以解释状态码的原因短语，可选的响应首部字段以及实体主体构成。


# HTTP 缓存控制

## 缓存规则解析
为方便大家理解，我们认为浏览器存在一个缓存数据库,用于存储缓存信息。

在客户端第一次请求数据时，此时缓存数据库中没有对应的缓存数据，需要请求服务器，服务器返回后，将数据存储至缓存数据库中。

[![](http://idiotsky.me/images3/http-summary-21.png)](http://idiotsky.me/images3/http-summary-21.png)

HTTP缓存有多种规则，根据是否需要重新向服务器发起请求来分类，我将其分为两大类(**强制缓存，对比缓存**)

在详细介绍这两种规则之前，先通过时序图的方式，让大家对这两种规则有个简单了解。

已存在缓存数据时，仅基于强制缓存，请求数据的流程如下

[![](http://idiotsky.me/images3/http-summary-22.png)](http://idiotsky.me/images3/http-summary-22.png)

已存在缓存数据时，仅基于对比缓存，请求数据的流程如下

[![](http://idiotsky.me/images3/http-summary-23.png)](http://idiotsky.me/images3/http-summary-23.png)

对缓存机制不太了解的同学可能会问，基于对比缓存的流程下，不管是否使用缓存，都需要向服务器发送请求，那么还用缓存干什么？

这个问题，我们暂且放下，后文在详细介绍每种缓存规则的时候，会带给大家答案。

我们可以看到两类缓存规则的不同，强制缓存如果生效，不需要再和服务器发生交互，而对比缓存不管是否生效，都需要与服务端发生交互。

两类缓存规则可以同时存在，强制缓存优先级高于对比缓存，也就是说，当执行强制缓存的规则时，如果缓存生效，直接使用缓存，不再执行对比缓存规则。

## 强制缓存

从上文我们得知，强制缓存，在缓存数据未失效的情况下，可以直接使用缓存数据，那么浏览器是如何判断缓存数据是否失效呢？

我们知道，在没有缓存数据的时候，浏览器向服务器请求数据时，服务器会将数据和缓存规则一并返回，缓存规则信息包含在响应header中。

对于强制缓存来说，响应header中会有两个字段来标明失效规则（Expires/Cache-Control）

使用chrome的开发者工具，可以很明显的看到对于强制缓存生效时，网络请求的情况

[![](http://idiotsky.me/images3/http-summary-24.png)](http://idiotsky.me/images3/http-summary-24.png)

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

[![](http://idiotsky.me/images3/http-summary-25.png)](http://idiotsky.me/images3/http-summary-25.png)

图中Cache-Control仅指定了max-age，所以默认为private，缓存时间为31536000秒（365天）

也就是说，在365天内再次请求这条数据，都会直接获取缓存数据库中的数据，直接使用。

## 对比缓存

对比缓存，顾名思义，需要进行比较判断是否可以使用缓存。

浏览器第一次请求数据时，服务器会将缓存标识与数据一起返回给客户端，客户端将二者备份至缓存数据库中。

再次请求数据时，客户端将备份的缓存标识发送给服务器，服务器根据缓存标识进行判断，判断成功后，返回304状态码，通知客户端比较成功，可以使用缓存数据。

第一次访问：

[![](http://idiotsky.me/images3/http-summary-26.png)](http://idiotsky.me/images3/http-summary-26.png)

再次访问：

[![](http://idiotsky.me/images3/http-summary-27.png)](http://idiotsky.me/images3/http-summary-27.png)

通过两图的对比，我们可以很清楚的发现，在对比缓存生效时，状态码为304，并且报文大小和请求时间大大减少。

原因是，服务端在进行标识比较后，只返回header部分，通过状态码通知客户端使用缓存，不再需要将报文主体部分返回给客户端。

对于对比缓存来说，缓存标识的传递是我们着重需要理解的，它在请求header和响应header间进行传递，

一共分为两种标识传递，接下来，我们分开介绍。

### Last-Modified  /  If-Modified-Since

Last-Modified：

服务器在响应请求时，告诉浏览器资源的最后修改时间。

[![](http://idiotsky.me/images3/http-summary-28.png)](http://idiotsky.me/images3/http-summary-28.png)

If-Modified-Since：

再次请求服务器时，通过此字段通知服务器上次请求时，服务器返回的资源最后修改时间。

服务器收到请求后发现有头If-Modified-Since 则与被请求资源的最后修改时间进行比对。

若资源的最后修改时间大于If-Modified-Since，说明资源又被改动过，则响应整片资源内容，返回状态码200；

若资源的最后修改时间小于或等于If-Modified-Since，说明资源无新修改，则响应HTTP 304，告知浏览器继续使用所保存的cache。

[![](http://idiotsky.me/images3/http-summary-29.png)](http://idiotsky.me/images3/http-summary-29.png)

### Etag  /  If-None-Match

（优先级高于Last-Modified  /  If-Modified-Since）

Etag：

服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器决定）。

[![](http://idiotsky.me/images3/http-summary-30.png)](http://idiotsky.me/images3/http-summary-30.png)

If-None-Match：

再次请求服务器时，通过此字段通知服务器客户段缓存数据的唯一标识。

服务器收到请求后发现有头If-None-Match 则与被请求资源的唯一标识进行比对，

不同，说明资源又被改动过，则响应整片资源内容，返回状态码200；

相同，说明资源无新修改，则响应HTTP 304，告知浏览器继续使用所保存的cache。

[![](http://idiotsky.me/images3/http-summary-31.png)](http://idiotsky.me/images3/http-summary-31.png)

## 小结

对于强制缓存，服务器通知浏览器一个缓存时间，在缓存时间内，下次请求，直接用缓存，不在时间内，执行比较缓存策略。

对于比较缓存，将缓存信息中的Etag和Last-Modified通过请求发送给服务器，由服务器校验，返回304状态码时，浏览器直接使用缓存。

浏览器第一次请求：

[![](http://idiotsky.me/images3/http-summary-32.png)](http://idiotsky.me/images3/http-summary-32.png)

浏览器再次请求时：

[![](http://idiotsky.me/images3/http-summary-9.png)](http://idiotsky.me/images3/http-summary-9.png)

还有一张图总结下：

[![](http://idiotsky.me/images3/http-summary-33.jpg)](http://idiotsky.me/images3/http-summary-33.jpg)


# 请求首部

## 请求信息性首部

| 首部 | 描述 | 
| :---: | :---: |
|Client-IP4| 提供了运行客户端的机器的 IP 地址|
|From| 提供了客户端用户的 E-mail 地址 |
|Host |给出了接收请求的服务器的主机名和端口号 |
|Referer |提供了包含当前请求 URI 的文档的 URL（正确的拼写其实应该是Referrer ，大家一致沿用错误至今） |
|UA-Color |提供了与客户端显示器的显示颜色有关的信息 |
|UA-CPU |给出了客户端 CPU 的类型或制造商 |
|UA-Disp |提供了与客户端显示器（屏幕）能力有关的信息| 
|UA-OS |给出了运行在客户端机器上的操作系统名称及版本 |
|UA-Pixels |提供了客户端显示器的像素信息 |
|User-Agent |将发起请求的应用程序名称告知服务器|

## Accept 首部

| 首部 | 描述 | 
| :---: | :---: |
|Accept |告诉服务器能够发送哪些媒体类型 |
|Accept- Charset |告诉服务器能够发送哪些字符集 |
|Accept- Encoding |告诉服务器能够发送哪些编码方式 |
|Accept- Language |告诉服务器能够发送哪些语言 |
|TE | 告诉服务器可以使用哪些扩展传输的编码|


常见内容编码

常用的内容编码有以下几种：

- gzip（GNU zip）  
  由文件压缩程序 gzip（GNU zip）生成的编码格式（RFC1952），采用 Lempel-Ziv 算法（LZ77）及 32 位循环冗余校验（Cyclic Redundancy Check，统称 CRC）
- compress（UNIX 系统的标准压缩）  
  由 UNIX 文件压缩程序 compress 生成的编码格式，采用 Lempel-Ziv-Welch 算法 （LZW）
- deflate（zlib）  
  组合使用 zlib 格式（RFC1950）及由 deflate 压缩算法（RFC1951）生成的编码格式
- identity（不进行编码）  
  不执行压缩或不会变化的默认编码格式


## 条件请求首部

| 首部 | 描述 | 
| :---: | :---: |
|Expect |允许客户端列出某请求所要求的服务器行为 |
|If-Match| 如果实体标记与文档当前的实体标记相匹配，就获取这份文档 |
|If-Modified-Since |除非在某个指定的日期之后资源被修改过，否则就限制这个请求 |
|If-None-Match |如果提供的实体标记与当前文档的实体标记不相符，就获取文档 |
|If-Range |允许对文档的某个范围进行条件请求 |
|If-Unmodified-Since| 除非在某个指定日期之后资源没有被修改过，否则就限制这个请求 |
|Range |如果服务器支持范围请求，就请求资源的指定范围|

## 安全请求首部

| 首部 | 描述 | 
| :---: | :---: |
|Authorization |包含了客户端提供给服务器，以便对其自身进行认证的数据 |
|Cookie |客户端用它向服务器传送一个令牌 —— 它并不是真正的安全首部，但确实隐含了安全功能 |
| Cookie | 用来说明请求端支持的 cookie 版本|

 
## 代理请求首部

| 首部 | 描述 | 
| :---: | :---: |
|Max-Forward |在通往源端服务器的路径上，将请求转发给其他代理或网关的最大次数 —— 与 TRACE 方法一同使用 |
|Proxy-Authorization |与 Authorization 首部相同， 但这个首部是在与代理进行认证时使用的 |
|Proxy-Connection |与 Connection 首部相同， 但这个首部是在与代理建立连接时使用的|


# 响应首部

## 响应信息性首部

| 首部 | 描述 | 
| :---: | :---: |
|Age |（从最初创建开始）响应持续时间 |
|Public | 服务器为其资源支持的请求方法列表 |
|Retry-After |如果资源不可用的话，在此日期或时间重试 Server 服务器应用程序软件的名称和版本 |
|Title | 对 HTML 文档来说，就是 HTML 文档 的源端给出的标题 |
|Warning| 比原因短语中更详细一些的警告报文|


## 协商首部

| 首部 | 描述 | 
| :---: | :---: |
|Accept-Ranges |对此资源来说，服务器可接受的范围类型 |
|Vary |服务器查看的其他首部的列表，可能会使响应发生变化；也就是说，这是一个首部列表，服务器会根据这些首部的内容挑选出最适合的资源版本发送给客户端。首部字段 Vary 可对缓存进行控制。源服务器会向代理服务器传达关于本地缓存使用方法的命令。从代理服务器接收到源服务器返回包含 Vary 指定项的响应之后，若再进行缓存，仅对请求中含有相同 Vary 指定首部字段的请求返回缓存。即使对相同资源发起请求，但由于 Vary 指定的首部字段不相同，因此必须要从源服务器重新获取资源。|

## 安全响应首部

| 首部 | 描述 | 
| :---: | :---: |
|Proxy-Authenticate| 来自代理的对客户端的质询列表 |
|Set-Cookie |不是真正的安全首部，但隐含有安全功能；可以在客户端设置一个令牌，以便服务器对客户端进行标识 |
|Set-Cookie2 |与 Set-Cookie 类似，RFC 2965 Cookie 定义； |
|WWW-Authenticate| 来自服务器的对客户端的质询列表。它会告知客户端适用于访问请求 URI 所指定资源的认证方案（Basic 或是 Digest）和带参数提示的质询（challenge）|


Cookie 的 HttpOnly 属性是 Cookie 的扩展功能，它使 JavaScript 脚本无法获得 Cookie。其主要目的为了防止跨站脚本攻击（Cross-site scripting，XSS）对 Cookie 的信息窃取。

````http
Set-Cookie: name-value;HttpOnly

````

顺带一提，该扩展并非是为了防止 XSS 而开发的。

# 实体首部


## 实体信息性首部

| 首部 | 描述 | 
| :---: | :---: |
|Allow |列出了可以对此实体执行的请求方法 |
|Location |告知客户端实体实际上位于何处；用于将接收端定向到资源的（可能是新的）位置（URL）上去|


## 内容首部

| 首部 | 描述 | 
| :---: | :---: |
|Content-Base16 |解析主体中的相对 URL 时使用的基础 URL |
|Content-Encoding |对主体执行的任意编码方式 |
|Content-Language| 理解主体时最适宜使用的自然语言 |
|Content-Length |主体的长度或尺寸|
| Content-Location |资源实际所处的位置 |
|Content-MD5 |主体的 MD5 校验和|
| Content-Range |在整个资源中此实体表示的字节范围 |
|Content-Type |这个主体的对象类型|


由于 HTTP 首部无法记录二进制值，所以要通过 Base-64 编码处理。采用 Content-MD5 这种方法，对内容上的偶发性改变是无从查证的，也无法检测出恶意篡改。原因在于，内容如果被篡改了，那么同时意味着 Content-MD5 也可以被重新计算后更新，被篡改。所以处在接收阶段的客户端是无法意识到报文主体以及首部字段 Content-MD5 是已经被篡改过的。

## 实体缓存首部

| 首部 | 描述 | 
| :---: | :---: |
|ETag |与此实体相关的实体标记 |
|Expires |实体不再有效，要从原始的源端再次获取此实体的日期和时间 |
|Last-Modified |这个实体最后一次被修改的日期和时间|


Expires 是 Web 服务器响应消息头字段，在响应 http 请求时告诉浏览器在过期时间前浏览器可以直接从浏览器缓存取数据，而无需再次请求。

Expires 的缺点是：响应报文中 Expires 所定义的缓存时间是相对服务器上的时间而言的，其定义的是资源“失效时刻”，如果客户端上的时间跟服务器上的时间不一致，缓存将失效。  

另外，Expires 主要使用在 HTTP1.0 版本。




如果两者的 URI 是相同，所以仅凭 URI 指定缓存的资源是很困难的。若下载过程中出现连续中断、再连接的情况，都会依据 ETag 值指定资源。

ETag 也分为强 ETag 值和弱 ETag 值：

强 ETag 值：

强 ETag 值，不论实体发生多少细微的变化都会改变其值。

````http  
ETag: "usagi-1234"

````

弱 ETag 值：

弱 ETag 值只用于提示资源是否相同。只有资源发生了根本改变，产生差异时才会改变 ETag 值。这时，会在字段值最开始处附加 W/

````http  
ETag: W/"usagi-1234"

````




# 扩展首部


## X-Frame-Options

首部字段 X-Frame-Options 属于 HTTP 响应首部，用于控制网站内容在其他 Web 网站的 Frame 标签内的显示问题。其主要目的是为了防止点击劫持（clickjacking）攻击。

## X-XSS-Protection

首部字段 X-XSS-Protection 属于 HTTP 响应首部，它是针对跨站脚本攻击（XSS）的一种对策，用于控制浏览器 XSS 防护机制的开关。0：将 XSS 过滤设置成无效状态，1：将 XSS 过滤设置成有效状态。

## DNT

首部字段 DNT 属于 HTTP 请求首部，其中 DNT 是 Do Not Track 的简称，意为拒绝个人信息被收集，是表示拒绝被精准广告追踪的一种方法。0：同意被追踪，1：拒绝被追踪。

## P3P

首部字段 P3P 属于 HTTP 响应首部，通过利用 P3P（The Platform for Privacy Preferences，在线隐私偏好平台）技术，可以让 Web 网站上的个人隐私变成一种仅供程序可理解的形式，以达到保护用户隐私的目的。

>在 HTTP 等多种协议中，通过给非标准参数加上前缀 X- ，来区别于标准参数，并使那些非标准的参数作为扩展变成可能。但是这种简单粗暴的做法有百害而无一益，因此在 “RFC6648 - Deprecating the "X-" Prefix and Similar Constructs in Application Protocols ”中提议停止该做法。然而，对已经在使用中的 X- 前缀来说，不应该要求其变更。



HTTP 首部字段将定义成缓存代理和非缓存代理的行为，分为 端到端首部（End-to-end Header）、逐跳首部（Hop-by-hop Header）

- 端到端首部：分在此类别中的首部会转发给请求 / 响应对应的最终接收目标，且必须保存在由缓存生成的相应中，另外规定它必须被转发。
- 逐跳首部：分在此类别中的首部只对单次转发有效，会因通过缓存或代理而不再转发。HTTP/1.1 和之后版本中，如果要使用 hop-by-hop 首部，需提供 Connection 首部字段。（Connection、Keep-Alive、Proxy-Authenticate、Proxy-Authorization、Trailer、TE、Transfer-Encoding、Upgrade 这 8 个首部字段属于逐跳首部，除此以外的字段都属于端到端首部）


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


# HTTP 各版本比较

## HTTP/1.0 与 HTTP/1.1 的区别

1. HTTP/1.1 默认是持久连接
2. HTTP/1.1 支持管线化处理
3. HTTP/1.1 支持虚拟主机
4. HTTP/1.1 新增状态码 100
5. HTTP/1.1 支持分块传输编码
6. HTTP/1.1 新增缓存处理指令 max-age

具体内容见上文

## HTTP/1.1 与 HTTP/2.0 的区别

> [HTTP/2 简介](https://developers.google.com/web/fundamentals/performance/http2/?hl=zh-cn)

### 多路复用

HTTP/2.0 使用多路复用技术，同一个 TCP 连接可以处理多个请求。

###  首部压缩

HTTP/1.1 的首部带有大量信息，而且每次都要重复发送。HTTP/2.0 要求通讯双方各自缓存一份首部字段表，从而避免了重复传输。

###  服务端推送

HTTP/2.0 在客户端请求一个资源时，会把相关的资源一起发送给客户端，客户端就不需要再次发起请求了。例如客户端请求 index.html 页面，服务端就把 index.js 一起发给客户端。

###  二进制格式

HTTP/1.1 的解析是基于文本的，而 HTTP/2.0 采用二进制格式。


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
片段标识符（fragment identifier）指的是，URL的#号后面的部分，比如http://example.com/x.html#fragment的#fragment。如果只是改变片段标识符，页面不会重新刷新。

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



ref
http://www.ruanyifeng.com/blog/2016/04/cors.html

http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html

https://github.com/halfrost/Halfrost-Field/blob/master/contents/Protocol/HTTP.md

https://www.cnblogs.com/chenqf/p/6386163.html