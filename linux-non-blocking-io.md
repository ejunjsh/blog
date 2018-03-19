---
title: linux的阻塞和非阻塞socket的区别
date: 2018-03-12 23:44:11
tags: [nio,linux,非阻塞socket,阻塞socket]
categories: linux
---
> 在上一篇文章有提到非阻塞socket，所以这篇文章就看看这个是什么东东👿

# 读操作
对于阻塞的socket,当socket的接收缓冲区中没有数据时，read调用会一直阻塞住，直到有数据到来才返回。当socket缓冲区中的数据量小于期望读取的数据量时，返回实际读取的字节数。当sockt的接收缓冲区中的数据大于期望读取的字节数时，读取期望读取的字节数，返回实际读取的长度。

对于非阻塞socket而言，socket的接收缓冲区中有没有数据，read调用都会立刻返回。接收缓冲区中有数据时，与阻塞socket有数据的情况是一样的，如果接收缓冲区中没有数据，则返回错误号为EWOULDBLOCK,表示该操作本来应该阻塞的，但是由于本socket为非阻塞的socket，因此立刻返回，遇到这样的情况，可以在下次接着去尝试读取。如果返回值是其它负值，则表明读取错误。

因此，非阻塞的rea调用一般这样写:
````c
if ((nread = read(sock_fd, buffer, len)) < 0)
{
 if (errno == EWOULDBLOCK)
  {
   return 0; //表示没有读到数据
  }
  else 
   return -1; //表示读取失败
}
else 
   return nread;读到数据长度
````
<!-- more -->

# 写操作
对于写操作write,原理是类似的，非阻塞socket在发送缓冲区没有空间时会直接返回错误号EWOULDBLOCK,表示没有空间可写数据，如果错误号是别的值，则表明发送失败。如果发送缓冲区中有足够空间或者是不足以拷贝所有待发送数据的空间的话，则拷贝前面N个能够容纳的数据，返回实际拷贝的字节数。

而对于阻塞Socket而言，如果发送缓冲区没有空间或者空间不足的话，write操作会直接阻塞住，如果有足够空间，则拷贝所有数据到发送缓冲区，然后返回.

非阻塞的write操作一般写法是:
````c
int write_pos = 0;
int nLeft = nLen;
while (nLeft > 0)
{
 int nWrite = 0;
 if ((nWrite = write(sock_fd, data + write_pos, nLeft)) <= 0)
 {
  if (errno == EWOULDBLOCK)
  {
    nWrite = 0;
  }
  else 
    return -1; //表示写失败
 }
 nLeft -= nWrite;
 write_pos += nWrite;
}
return nLen;
````

# 建立连接
阻塞方式下，connect首先发送SYN请求道服务器，当客户端收到服务器返回的SYN的确认时，则connect返回.否则的话一直阻塞.

非阻塞方式，connect将启用TCP协议的三次握手，但是connect函数并不等待连接建立好才返回，而是立即返回。返回的错误码为EINPROGRESS,表示正在进行某种过程.

# 接收连接
对于阻塞方式的倾听socket,accept在连接队列中没有建立好的连接时将阻塞，直到有可用的连接，才返回。

非阻塞倾听socket,在有没有连接时都立即返回，没有连接时，返回的错误码为EWOULDBLOCK,表示本来应该阻塞。

# 无阻塞的设置方法
方法一:fcntl
````c
int flag;
if (flag = fcntl(fd, F_GETFL, 0) <0) perror("get flag");
flag |= O_NONBLOCK;
if (fcntl(fd, F_SETFL, flag) < 0)
perror("set flag");
````
方法二:ioctl
````
int b_on = 1;
ioctl (fd, FIONBIO, &b_on);
````

# 总结
非阻塞socket可以通过不断轮询来实现类似io复用的效果，但是不建议，因为会造成cpu的空转（如果一直没数据读写的话）。感觉跟java的nio上设置channel为非阻塞有点关系吧👿