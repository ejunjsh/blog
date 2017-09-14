---
title: nginx location 语法和优先级
date: 2014-10-07 20:53:44
tags: nginx
categories: ngix
---
> 验证下，记录下，以后不用到处找

<!-- more -->
语法:location [=|~|~*|^~] /uri/ { … }
* = 精确匹配，例如 location =/test 只匹配 /test
* ~ 后面接正则，区分大小写,匹配满足此正则的url
* ~* 后面接正则，不区分大小写,匹配满足此正则的url
* ^~ 匹配以什么开头的，例如  location ^~  /test 匹配 /test,/test/1,/test/2 ;
* 什么都不加的话，类似第四点，location /test 匹配 /test,/test/1,/test/2 ;
优先级
= 大于 ^~ 大于 ~ 或者~* 大于 什么都不加

假如遇到同级的话
* ~ 和~* （他们是同级的）比较的话，就是根据他们在文件的出现顺序。例如
location ~ /sb.txt
location ~* /sb.txt
如果访问 http://localhost/sb.txt,就会跳到第一个，访问 http://localhost/SB.txt 跳到第二个，
换个顺序
location ~* /sb.txt
location ~ /sb.txt
不管访问 http://localhost/sb.txt 还是 http://localhost/SB.txt ,都只会跳到第一个
* ^~ 的话，就根据他们的匹配度，例如
location ^~ /sb/
location ^~ /sb/sb2/
如果访问http://localhost/sb/sb2/,就会跳到第二个去处理，因为他更加匹配吧。
如果访问http://localhost/sb/，就会跳到第一个处理，如果访问http://localhost/sb/sb3,也会跳到第一个。
* 什么都不加的话，也是根据他们的匹配度，例如
location /sb/
location /sb/sb2/
如果访问http://localhost/sb/sb2/,就会跳到第二个去处理，因为他更加匹配吧。
如果访问http://localhost/sb/，就会跳到第一个处理，如果访问http://localhost/sb/sb3,也会跳到第一个。