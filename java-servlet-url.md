---
title: Servlet获取URL地址
date: 2014-02-20 17:09:11
tags: java
categories: java
---
> 老是不记得，果断mark.

<!-- more -->

这里来说说用Servlet获取URL地址。在HttpServletRequest类里，有以下六个取URL的函数： 

* getContextPath 取得项目名 
* getServletPath 取得Servlet名 
* getPathInfo 取得Servlet后的URL名，不包括URL参数 
* getRequestURL 取得不包括参数的URL 
* getRequestURI 取得不包括参数的URI，即去掉协议和服务器名的URL 

具体如下图：
[![](/images/java-servlet-url.png)](/images/java-servlet-url.png)
相对应的函数的值如下： 

* getContextPath：/ServletTest 
* getServletPath：/main 
* getPathInfo：/index/testpage/test 
* getRequestURL：http://localhost:8080/ServletTest/main/index/testpage/test 
* getRequestURI：/ServletTest/main/index/testpage/test 