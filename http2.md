# HTTP/2

HTTP协议从1991年的HTTP 0.9 到 HTTP 1.0、HTTP 1.1 再到 2015年开始至今的HTTP 2.0，每个版本都带来了惊人的升级体验。HTTP/2的惊人之处可以在[这里](https://http2.akamai.com/demo)体验一下，可以感受到相比HTTP 1.1在性能上的大幅提升。

## HTTP/2的新特性：

1. 新的二进制格式：HTTP 1.x的解析是基于文本，HTTP 2.0的解析是基于二进制，更加适用于服务器间信息传输了。
2. 多路复用（MultiPlexing）：也就是连接共享，每个请求都有一个ID，这样在一个连接上可以发送多个请求，并且它们在传输过程中是混杂在一起的，接收方可以根据请求的ID将请求再归属到不同的服务端请求里面。
3. 请求优先级（request prioritization）：HTTP/2中，一个源只有一个连接来实现多路复用，所有资源通过一个连接传输，为了避免线头堵塞（Head Of Line Block），这时资源传输的顺序就更重要了。优先加载重要资源，可以尽快渲染页面，提升用户体验。
4. header压缩：通信上方各自缓存一份Header fields表，只传输压缩之后的相应编码，避免了重复的header传输。
5. 服务端推送（server push）：这个就很好理解了，HTTP 1.x都是只能从client拉取资源，HTTP/2支持从server端推送资源至client端。

## HTPP 2.0的多路复用和HTTP 1.x的长连接的区别

* HTTP 1.0中一次请求响应就建立一次连接，用完关闭，每个请求都要建立一个连接。
* HTTP 1.1 Pipeling解决方式，也是我们常说的Keep-Alive模式，建立一次连接之后，若干个请求排队串行化单线程处理，后面的请求等待前面的请求返回了获得执行机会，一旦有请求超时等待，后续的请求只能被阻塞，毫无办法，也就是人们常说的线头阻塞（Head-of-Line Blocking）。
* HTTP 2.0多路复用，多个请求同时在一个连接上并行执行，若某个请求任务耗时严重，不会影响到其它请求的正常执行。
[![](https://upload-images.jianshu.io/upload_images/2250588-0942ff5daff5e5db.png)](https://upload-images.jianshu.io/upload_images/2250588-0942ff5daff5e5db.png)

## HTTP/2 & HTTPS
HTTPS是建构在SSL/TLS之上的HTTP协议，一种保证信息传输安全的协议，HTTP/2的定义本身是和HTTPS无关的。但是，在开放的互联网上HTTP/2将只用于`https://网址`，而 `http://网址`将继续使用HTTP/1.x，目的是在开放互联网上增加使用加密技术，以提供强有力的保护去遏制主动攻击。所以，在进行HTTP/2推广的同时也在强制推广HTTPS。
目前大部分HTTP/2的实现都是要强制使用HTTPS的，很少能见到有直接在TCP层之上实现的HTTP/2。
因此，可能会有人说建立在HTTPS之上会额外消耗很多性能，这个答案是肯定的。HTTP使用TCP三次握手建立连接，客户端和服务器需要交换3个包，HTTPS除了TCP的三个包，还要加上SSL/TLS握手需要的9个包，所以一共是12个包，增加了建立连接的时间。但是HTTP/2的多路复用技术，建立一次连接可以一直发送请求，因此HTTPS带来的性能影响可以忽略不计。

ref
https://www.jianshu.com/p/569193b41e61
