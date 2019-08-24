##Spring Boot启动时动态切换每个环境的配置文件
开发项目一般是开发环境，测试环境，和生产环境，例如：SpringBoot的application.properties配置如下
开发环境：application-dev.properties
测试环境：application-test.properties
生产环境：application-prod.properties

当你启动SpringBoot时，切换每个环境的application.properties。由于有多种设置方法，这里介绍四种:
- 将配置文件设置为启动参数 
```` 
有以下两种设置启动参数的方法
1.命令行参数
java -jar spring-boot-application-properties-sample-1.0.0.jar --spring.profiles.active=dev1

2.java系统参数
java -jar -Dspring.profiles.active=dev1 spring-boot-application-properties-sample-1.0.0.jar

命令行参数的优先级大于java系统参数
java -jar -Dspring.profiles.active = dev1 spring-boot-application-properties-sample.jar --spring.profiles.active = dev2
````

- 用OS环境变量进行配置文件设置
````
windows
系统环境变量  SPRING_PROFILES_ACTIVE
linux
export SPRING_PROFILES_ACTIVE=dev1
````

- 使用tomcat的JNDI进行配置文件设置
````
把Spring Boot打成war包放在Tomcat中执行。
<Context>
<Environment type = “java.lang.String”  name = “spring.profiles.active”  value = “dev2” />
</ Context>
````

- 将配置文件设置为tomcat的startup.bat（sh）中的环境变量
````
如startup.bat（sh），catalina.bat（sh）等。
windows
set "SPRING_PROFILES_ACTIVE=dev2"

linux
export SPRING_PROFILES_ACTIVE=dev2

如果在context中设置了，这里也设置了，context优先级高。
````

如果你这些都不设置，可以在application.properties 中设置spring.profiles.active=dev1，打包的时候回加载dev1，但是这个的优先级在上面说的4种之下。

linux的jar启动脚本
文件名:restart_user
````
#!/bin/bash
echo -e "\033[32m===start run user-center===\033[0m"
TT=$(ps -ef | grep user-center-1.0.0.jar| grep -v "grep" |awk '{print $2}')
if [ ! ${TT} ];then
        echo -e "\033[33m===user-center??𼰿杩𹰿?===\033[0m"
else
        echo -e "\033[31m===?抽𻠀user-center===\033[0m"
        kill -9 ${TT}
        sleep 2
fi
echo -e "\033[32m===??𸐿user-center===\033[0m"
nohup java  -Xmx512m -Xms512m -jar /projects/xa-estimate/jar/user-center-1.0.0.jar --spring.profiles.active=test &
echo -e "\033[32m===end===\033[0m"
````