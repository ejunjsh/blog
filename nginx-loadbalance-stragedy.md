---
title: nginx负载均衡的5种策略
date: 2014-10-17 23:41:29
tags: nginx
categories: nginx
---
nginx可以根据客户端IP进行负载均衡，在upstream里设置ip_hash，就可以针对同一个C类地址段中的客户端选择同一个后端服务器，除非那个后端服务器宕了才会换一个。

nginx的upstream目前支持的5种方式的分配

# 轮询（默认）
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。 
````
upstream backserver { 
server 192.168.0.14; 
server 192.168.0.15; 
} 
````
<!-- more -->

# 指定权重
指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
````
upstream backserver { 
server 192.168.0.14 weight=10; 
server 192.168.0.15 weight=10; 
} 
````

# IP绑定 ip_hash
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。 
````
upstream backserver { 
ip_hash; 
server 192.168.0.14:88; 
server 192.168.0.15:80; 
} 
````

# fair（第三方）
按后端服务器的响应时间来分配请求，响应时间短的优先分配。 
````
upstream backserver { 
server server1; 
server server2; 
fair; 
} 
````

# url_hash（第三方）
按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。 
````
upstream backserver { 
server squid1:3128; 
server squid2:3128; 
hash $request_uri; 
hash_method crc32; 
} 
````

# least_conn
这个功能可以让在有需要长时间才能完成的请求时让请求的分配更公平。简言之就是它可以控制不让单独的一台服务器负载太高，而是会根据其负载动态的分配请求。
````
upstream backserver { 
least_conn
server server1; 
server server2; 
} 
````

# down
标记某台应用服务器暂时不参与负载均衡。
````
upstream backserver { 
server server1 down;
server server2; 
} 
````

# backup
这个标记和`down`正好相反，它是在其他所有应用服务器全部繁忙或者处于`down`状态时才会被启用，所以这台机器的压力会最轻。
````
upstream backserver { 
server server1 backup;
server server2; 
} 
````

# 健康状态检查
Nginx会自动把返回报错信息的应用服务器标记为『故障』，然后就不再把新的请求分配给它了，这个相关的配置有:
1. `max_fails` 报错多少次会被标记为failed
2. `fail_timeout` 超时多少次会被标记成failed