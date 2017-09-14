---
title: maven resource 记录
date: 2014-02-20 23:15:58
tags: [maven,java]
categories: java
---
> 今天遇到maven打包的时候发现在main/java里面的xml没有打包进jar上。

<!-- more -->
上网搜了下，maven默认打包main/resource的资源，要想打包main/java的要像下面这样配。
````xml
<build>
  <resources> 
            <resource> 
                <directory>src/main/java</directory> 
                <includes> 
                    <include>**/*.properties</include> 
                    <include>**/*.xml</include> 
                    <include>**/*.tld</include> 
                </includes> 
                <filtering>true</filtering> 
            </resource> 
  </resources> 
</build>
````
搞定收工。。。不知道有没有更好到方法呢？