---
title: postman的几种body的使用介绍
date: 2017-08-10 20:11:55
tags: postman
categories: http
---
> postman,用来模拟发送http请求的工具，里面涉及的请求body有以下几个类型，所以记下，而且也能理解http body的几种格式，分享之。。

<!-- more -->
# form-data
就是http请求中的multipart/form-data,它会将表单的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，也可以上传文件。当上传的字段是文件时，会有Content-Type来表名文件类型；content-disposition，用来说明字段的一些信息；
由于有boundary隔离，所以multipart/form-data既可以上传文件，也可以上传键值对，它采用了键值对的方式，所以可以上传多个文件。

[![](/images/postman-1.png)](/images/postman-1.png)
内容为
````
POST /login HTTP/1.1
Host: 10.170.56.67
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Cache-Control: no-cache
Postman-Token: 9843651a-5bf9-0544-03c1-fcc2a16f484b

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="username"

admin
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="password"

admin123
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="abc"; filename=""
Content-Type: 


------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="tttt"; filename=""
Content-Type: 


------WebKitFormBoundary7MA4YWxkTrZu0gW--
````

# x-www-form-urlencoded
就是application/x-www-from-urlencoded,会将表单内的数据转换为键值对，并以urlencode为格式
[![](/images/postman-2.png)](/images/postman-2.png)
内容为
````
POST /login HTTP/1.1
Host: 10.170.56.67
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache
Postman-Token: e6887900-a46e-2ff4-8232-de878b75f5fd

username=admin&password=admin123
````

# raw
可以上传任意格式的文本，可以上传text、json、xml、html等
[![](/images/postman-3.png)](/images/postman-3.png)
内容为
````
POST /login HTTP/1.1
Host: 10.170.56.67
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: 233df0e0-c6d9-98c7-4d7e-736329322683

{
  "abc":"cba",
  "cba":"abc"
}
````
从图片和内容对比，可以发现，基本，粘什么，就发什么，不会进行任何转意。

# binary
相当于Content-Type:application/octet-stream,从字面意思得知，只可以上传二进制数据，通常用来上传文件，由于没有键值，所以，一次只能上传一个文件。

# multipart/form-data与x-www-form-urlencoded区别
* multipart/form-data：既可以上传文件等二进制数据，也可以上传表单键值对，只是最后会转化为一条信息；
* x-www-form-urlencoded：只能上传键值对，并且键值对都是间隔分开的