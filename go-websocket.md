---
title: 用go小试websocket
date: 2018-01-10 15:44:21
tags: [go,websocket]
categories: go
---
# 什么是websocket

初次接触 WebSocket 的人，都会问同样的问题：我们已经有了 HTTP 协议，为什么还需要另一个协议？它能带来什么好处？
答案很简单，因为 HTTP 协议有一个缺陷：通信只能由客户端发起。

举例来说，我们想了解今天的天气，只能是客户端向服务器发出请求，服务器返回查询结果。HTTP 协议做不到服务器主动向客户端推送信息。
这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用"轮询"：每隔一段时候，就发出一个询问，了解服务器有没有新的信息。最典型的场景就是聊天室。

轮询的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的。

WebSocket 协议在2008年诞生，2011年成为国际标准。所有浏览器都已经支持了。

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于[服务器推送](https://en.wikipedia.org/wiki/Push_technology)技术的一种。
[![](http://idiotsky.me/images2/go-websocket-1.png)](http://idiotsky.me/images2/go-websocket-1.png)

<!-- more -->
其他特点包括：
1. 建立在 TCP 协议之上，服务器端的实现比较容易。
2. 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
3. 数据格式比较轻量，性能开销小，通信高效。
4. 可以发送文本，也可以发送二进制数据。
5. 没有同源限制，客户端可以与任意服务器通信。
6. 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。
````
ws://example.com:80/some/path
````

[![](http://idiotsky.me/images2/go-websocket-2.jpg)](http://idiotsky.me/images2/go-websocket-2.jpg)

# 小试
用一个echo的例子来试一下websocket

## 客户端实现
````html
<!DOCTYPE HTML>
<html>
<head>
    <script type="text/javascript">
        var ws;
        function connect() {
            ws= new WebSocket("ws://localhost:8080/echo");
            ws.onopen = function()
            {
                alert("connection is successful");
            };
            ws.onmessage = function (e)
            {
                var msg = e.data;
                var li=document.createElement("li");
                li.innerText=msg;
                document.getElementsByTagName("ul")[0].appendChild(li);
            };
            ws.onclose = function()
            {
                // websocket is closed.
                alert("Connection is closed...");
            };
        }

        function send() {
            if(ws){
                var msg=document.getElementById("txt").value;
                ws.send(msg);
                return;
            }
            alert("connect first!!!");
        }
    </script>
</head>
<body>
<div>
    <a href="javascript:connect()">connect</a>
    <textarea id="txt" style="width: 100%"></textarea>
    <button id="sendBtn" onclick="send()">send</button>
    <ul>

    </ul>
</div>
</body>
</html>

````
代码很简单，就是用`new WebSocket("ws://localhost:8080/echo")`初始化websocket，然后注册相关事件，就可以完成一个简单websocket客户端了。

## 服务端实现
go的标准库没有实现websocket的功能，所以要用`github.com/gorilla/websocket`这个库来实现服务端
````go
package main

import (
	"log"
	"net/http"

	"github.com/gorilla/websocket"

)

var upgrader = websocket.Upgrader{
	ReadBufferSize:  1024,
	WriteBufferSize: 1024,
	CheckOrigin: func(r *http.Request) bool {
		return true
	},
}



func main() {
	static := http.FileServer(http.Dir("./static"))
	http.Handle("/", static)

	http.HandleFunc("/echo", echo)

	log.Printf("Service started on %d \n", 8080)
	log.Fatal(http.ListenAndServe(":8080", nil))
}

func echo(w http.ResponseWriter, r *http.Request) {
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println("upgrade error:", err.Error())
		return
	}
	log.Println("Connected...")
	defer conn.Close()
	for {
		messageType, p, err := conn.ReadMessage()
		if err != nil {
			log.Println("read message error:", err.Error())
			break
		}

		if err := conn.WriteMessage(messageType, p); err != nil {
			log.Println("write message error:", err.Error())
			break
		}
	}
	log.Println("Disconnect.")
}
````
首先创建一个http服务，然后把上面的html作为静态资源，同时定义一个`echo`的处理函数。函数里面对请求进行`upgrade`，表示从普通的请求变成websocket（前提是请求头里面要包含`upgrade`标记），之后就获取了connection，接下来就跟平常tcp的连接那样读写数据了。

# 总结
用go实现的websocket简直是简单到爆了，基本第三方库已经封装好了👿

所有代码在 https://github.com/ejunjsh/go-code/tree/master/websocket