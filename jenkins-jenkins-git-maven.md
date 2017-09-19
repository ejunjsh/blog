---
title: jenkins + git+maven做持续集成
date: 2014-02-20 21:02:27
tags: [jenkins,git,maven,java]
categories: java
---
1. 下个jenkins，官网去下 http://jenkins-ci.org/，里面提供war包下载，直接部署到tomcat什么上面吧。
<!-- more -->
2. 部署成功后打开网站例如：http://localhost/jenkin，默认是不带git的插件的，所以先去下一个先，点击主页的右侧“系统管理”=>"管理插件"=>“可选插件” 找到"git plungin" 然后点击直接安装。（这可能要花点时间）
3. 下完git插件后就要配环境了，还是点击右侧“系统管理”=>“系统设置” 主要配jdk和maven的环境。（把自动安装勾掉就可以输路径了），保存下就可以了。
[![](http://idiotsky.me/images/jenkins-git-maven-1.png)](http://idiotsky.me/images/jenkins-git-maven-1.png) 
[![](http://idiotsky.me/images/jenkins-git-maven-2.png)](http://idiotsky.me/images/jenkins-git-maven-2.png) 

4. 点击右侧“新建”=>“构建一个maven项目” 输入名字到下一步
如下图勾上“丢弃旧的构建”，按照自己的需要配置，否则很占硬盘。
[![](http://idiotsky.me/images/jenkins-git-maven-3.png)](http://idiotsky.me/images/jenkins-git-maven-3.png) 
配置git仓库（如果是私有库，必须添加一个Credentials，点击右侧Add，在弹出界面录入帐号密码）
[![](http://idiotsky.me/images/jenkins-git-maven-4.png)](http://idiotsky.me/images/jenkins-git-maven-4.png) 
接下来配置定时构建（勾上Build periodically,图中设置是每15分钟一次），配置要执行的maven命令 clean install (mvn不用输)
[![](http://idiotsky.me/images/jenkins-git-maven-5.png)](http://idiotsky.me/images/jenkins-git-maven-5.png) 
保存后，一个构建就可以了（可以立即构建试试，也可以定时执行）。jenkins提供了一堆的页面去展示构建的过程，很不错。
如果web程序想自动部署到本地的tomcat，可以试下cargo插件，加上下面代码到项目pom上。下面代码改下路径就可以了。当然也可以部署到远程，就不贴了。
````xml
<plugin>
				<groupId>org.codehaus.cargo</groupId>
				<artifactId>cargo-maven2-plugin</artifactId>
				<version>1.4.5</version>
				<configuration>
					<container>
						<containerId>tomcat7x</containerId>
						<home>/opt/apache-tomcat-7.0.47</home>
					</container>
					<configuration>
						<type>existing</type>
						<home>/opt/apache-tomcat-7.0.47</home>
					</configuration>
				</configuration>
				<executions>
        <execution>
            <id>tomcat-deploy</id>
            <phase>package</phase>
            <goals><goal>deploy</goal></goals>
        </execution>
    </executions>	
</plugin>
````
这样一个持续集成就配好了。想想那边提交代码，另一边就自动部署到tomcat上，爽歪歪了。

