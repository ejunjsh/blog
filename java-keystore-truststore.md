https://www.jianshu.com/p/569193b41e61
https://blog.csdn.net/shfqbluestone/article/details/21242323
https://blog.csdn.net/chenzanlong123/article/details/11784143

	导入证书到JRE环境
keytool -import -trustcacerts -alias tomcat -file server.cer -keystore  "%JAVA_HOME%\jre\lib\security\cacerts "  -storepass changeit
