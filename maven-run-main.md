---
title: 用Maven跑Java main的3种方法
date: 2016-02-06 23:34:28
tags: [java,maven]
categories: java
---
# 概述
Maven exec plugin可以使我们运行自己工程的Java类的main方法，并在classpath里自动包含工程的dependencies。本文用示例代码展示了使用maven exec plugin来运行java main方法的3种方法。

<!-- more -->

# 在命令行（Command line）运行
用这种方式运行的话，并没有在某个maven phase中，所以你首先需要compile（编译）一下代码。 
请记住exec:java不会自动编译代码，需要先编译才行。
````
mvn compile
````
## 不带参数跑：
````
mvn exec:java -Dexec.mainClass="com.vineetmanohar.module.Main"
````
## 带参数跑：
````
mvn exec:java -Dexec.mainClass="com.vineetmanohar.module.Main" -Dexec.args="arg0 arg1 arg2"
````
## 在classpath里用runtime依赖
````
mvn exec:java -Dexec.mainClass="com.vineetmanohar.module.Main" -Dexec.classpathScope=runtime
````

# 在pom.xml文件的某个phase里运行
也可以在maven的某个phase里运行main方法。比如，作为test phase的一部分运行CodeGenerator.main()方法
````xml
<build>  
 <plugins>  
  <plugin>  
   <groupId>org.codehaus.mojo</groupId>  
   <artifactId>exec-maven-plugin</artifactId>  
   <version>1.1.1</version>  
   <executions>  
    <execution>  
     <phase>test</phase>  
     <goals>  
      <goal>java</goal>  
     </goals>  
     <configuration>  
      <mainClass>com.vineetmanohar.module.CodeGenerator</mainClass>  
      <arguments>  
       <argument>arg0</argument>  
       <argument>arg1</argument>  
      </arguments>  
     </configuration>  
    </execution>  
   </executions>  
  </plugin>  
 </plugins>  
</build>
````
用以上配置运行exec plugin，运行相应的phase就行了。
````
mvn test
````
# 在pom.xml文件的某个profile运行
也可以用不同的profile运行main方法。只要用`<profile>`标签包裹住以上配置就行。
````xml
<profiles>  
 <profile>  
  <id>code-generator</id>  
  <build>  
   <plugins>  
    <plugin>  
     <groupId>org.codehaus.mojo</groupId>  
     <artifactId>exec-maven-plugin</artifactId>  
     <version>1.1.1</version>  
     <executions>  
      <execution>  
       <phase>test</phase>  
       <goals>  
        <goal>java</goal>  
       </goals>  
       <configuration>  
        <mainClass>com.vineetmanohar.module.CodeGenerator</mainClass>  
        <arguments>  
         <argument>arg0</argument>  
         <argument>arg1</argument>  
        </arguments>  
       </configuration>  
      </execution>  
     </executions>  
    </plugin>  
   </plugins>  
  </build>  
 </profile>  
</profiles>
````
调用以上profile，运行以下命令就行：
````
mvn test -Pcode-generator
````