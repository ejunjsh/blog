---
title: 启用Nginx状态监控
date: 2014-10-12 20:48:39
tags: nginx
categories: ngnix
---
# 编译Nginx添加http_stub_status_module
编译Nginx的时候添加参数：--with-http_stub_status_module
````bash
cd nginx-{version}/

./configure  --prefix=/opt/nginx --with-http_stub_status_module
 --with-http_ssl_module

make && make install
````
<!-- more -->
# 启用nginx status配置
修改Nginx配置文件nginx.conf，在HTTP段中添加
````bash
vi /opt/nginx/conf/nginx.conf
````
````bash
server

{

  listen 80;
  server_name localhost;

  location /nginx_status {                  #主要是这里代表根目录显示信息
  stub_status on;
  access_log  off;
  }
}
````
# 打开status页面
浏览器访问监控页面地址http://{your IP}/nginx-status,显示如下
````
Active connections: 2 server accepts handled requests 8 8 33 Reading: 0 Writing: 1 Waiting: 1 
````
解析： 
Active connections    //当前 Nginx 正处理的活动连接数。 
server accepts handledrequests //总共处理了8 个连接 , 成功创建 8 次握手,总共处理了33个请求。 
Reading //nginx 读取到客户端的 Header 信息数。 
Writing //nginx 返回给客户端的 Header 信息数。 
Waiting //开启 keep-alive 的情况下，这个值等于 active – (reading + writing)，意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接