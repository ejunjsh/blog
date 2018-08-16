---
title: epoll的ET和LT举例
date: 2018-08-14 21:30:07
tags: [c,epoll]
categories: c
---

# ET

Edge Triggered (ET) 边缘触发只有数据到来,才触发,不管缓存区中是否还有数据。

LT(level triggered)是缺省的工作方式，并且同时支持block(阻塞)和no-block(非阻塞) socket.在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的fd进行IO操作。如果你不作任何操作，内核还是会继续通知你的，所以，这种模式编程出错误可能性要小一点。传统的select/poll都是这种模型的代表．

优点：当进行socket通信的时候，保证了数据的完整输出，进行IO操作的时候，如果还有数据，就会一直的通知你。

缺点：由于只要还有数据，内核就会不停的从内核空间转到用户空间，所有占用了大量内核资源，试想一下当有大量数据到来的时候，每次读取一个字节，这样就会不停的进行切换。内核资源的浪费严重。效率来讲也是很低的。

# LT

Level Triggered (LT) 水平触发只要有数据都会触发。

ET(edge-triggered)是高速工作方式，只支持no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知。请注意，如果一直不对这个fd作IO操作(从而导致它再次变成未就绪)，内核不会发送更多的通知(only once).

优点：每次内核只会通知一次，大大减少了内核资源的浪费，提高效率。

缺点：不能保证数据的完整。不能及时的取出所有的数据。

应用场景： 处理大数据。使用non-block(非阻塞)模式的socket。

<!-- more -->

# 举例

下面是一个客户端和服务端的代码，客户端每次输出10个字节，然后休眠5秒，服务器会分别使用LT和ET来展示他们的不同。

客户端代码：

````c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#include <fcntl.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define MAXLINE  10
#define SERV_PROT 8000

int main(void)
{
    struct sockaddr_in seraddr;
    char buf[MAXLINE];
    int connfd, i;
    char ch = 'a';
    connfd = socket(AF_INET,SOCK_STREAM,0);
    bzero(&seraddr,sizeof(seraddr));
    seraddr.sin_family = AF_INET;
    seraddr.sin_port = htons(SERV_PROT);
    struct in_addr s;
    inet_pton(AF_INET, "127.0.0.1", (void *)&s);
    seraddr.sin_addr=s;
    connect(connfd, (struct sockaddr *)&seraddr, sizeof(seraddr));
    while(1){
        for(i=0;i < MAXLINE/2;i++){
            buf[i] = ch;
        }
        buf[i-1] = '\n';
        ch++;
        for(;i<MAXLINE;i++)
            buf[i] = ch;
        buf[i-1] = '\n';
        ch++;
        write(connfd, buf, sizeof(buf));
        sleep(5);
    }

    close(connfd);
    return 0;
}
````

服务端代码（LT）：

````c
#include <stdio.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <signal.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/epoll.h>
#include <unistd.h>
#define MAXLINE 10
#define SERV_PORT 8000
int main(void)
{
 struct sockaddr_in servaddr, cliaddr;
 socklen_t cliaddr_len;
 int listenfd, connfd;
 char buf[MAXLINE];
 char str[INET_ADDRSTRLEN];
 int i, efd;
 listenfd = socket(AF_INET, SOCK_STREAM, 0);

 bzero(&servaddr, sizeof(servaddr));
 servaddr.sin_family = AF_INET;
 servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
 servaddr.sin_port = htons(SERV_PORT);
 bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));
 listen(listenfd, 20);
 struct epoll_event event;
 struct epoll_event resevent[10];
 int res, len;
 efd = epoll_create(10);
 event.events = EPOLLIN;
 printf("Accepting connections ...\n");
 cliaddr_len = sizeof(cliaddr);
 connfd = accept(listenfd, (struct sockaddr *)&cliaddr, &cliaddr_len);
 printf("received from %s at PORT %d\n",
 inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
 ntohs(cliaddr.sin_port));
 event.data.fd = connfd;
 epoll_ctl(efd, EPOLL_CTL_ADD, connfd, &event);
 while (1) {
  res = epoll_wait(efd, resevent, 10, -1);
  if (resevent[0].data.fd == connfd) {
  len = read(connfd, buf, MAXLINE/2);
  write(STDOUT_FILENO, buf, len);
 }
}
 return 0;
}
````

输出

```` 
$ ./server
Accepting connections ...
received from 127.0.0.1 at PORT 40580
aaaa
bbbb
# 等5秒
cccc
dddd
# 等5秒
# 以下输出省略
````

上面输出符合预期

接下里把服务器改成ET模式会怎么样呢，下面是要修改的地方

````diff
32c32
<  event.events = EPOLLIN;
---
> event.events = EPOLLIN|EPOLLET;
````

输出结果如下

```` 
$ ./server
Accepting connections ...
received from 127.0.0.1 at PORT 40580
aaaa
# 等5秒
bbbb
# 等5秒
cccc
# 等5秒
dddd
# 等5秒
# 以下输出省略
````

问题来了，输出的东西比ET的时候慢了半拍了，原因在于以下代码：

````c
while (1) {
  res = epoll_wait(efd, resevent, 10, -1);
  if (resevent[0].data.fd == connfd) {
  len = read(connfd, buf, MAXLINE/2);
  write(STDOUT_FILENO, buf, len);
 }
````

`read`函数只取5个字节，之后就没有输出了，等客户端下次来数据的时候再次触发下5个字节，这是ET模式的正常表现形式，而LT会在下次循环的时候，会再次触发可读，然后`epoll_wait`返回，接下来继续把剩下5个字节读入。

那接下来修改ET模式

````diff
9a10
> #include <fcntl.h>
38a40,42
> int flag=fcntl(connfd,F_GETFL);
> flag|=O_NONBLOCK;
> fcntl(connfd,F_SETFL,flag);
44,45c48,50
<   len = read(connfd, buf, MAXLINE/2);
<   write(STDOUT_FILENO, buf, len);
---
>   while ((len = read(connfd, buf, MAXLINE/2)) >0 )
>         write(STDOUT_FILENO, buf, len);
>    }
47d51
< }
````


输出

```` 
$ ./server
Accepting connections ...
received from 127.0.0.1 at PORT 40580
aaaa
bbbb
# 等5秒
cccc
dddd
# 等5秒
# 以下输出省略
````

正常了，上面修改的部分，主要有两点：
* 把套接字描述符改成非阻塞
* 循环读出所有数据

__为什么ET模式只能用在非阻塞的套接字呢，通过上面例子可以看到，如果是阻塞的话，在读出10个字节之后，循环还会继续，`read`函数会阻塞直到下次数据到来，这样的话，如果是多连接的情况下，所有连接的数据都会阻塞在这个连接的`read`函数里面了，所以ET模式只支持非阻塞是有道理的。__

# 实验代码

所有代码在这个[链接](https://github.com/ejunjsh/c-code/tree/master/epoll)
