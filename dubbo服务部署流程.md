###SpringBoot+dubbo+zookeeper服务部署流程
####jar包导入
- dubbo整合SpringBoot和zookeeper时注意zookeeper的客户端curator的版本，这里选择dubbo2.7.3和curator4.0.1的版本，同时zookeeper的版本是3.5.5
这里需要指出的是如果curator和zookeeper的版本过低而dubbo的版本过高，从而导致触发curator不停的注册监听事件导致内存溢出
````
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>2.7.3</version>
</dependency>
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.7.3</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.0.1</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.0.1</version>
</dependency>
````

###dubbo-admin监控中心的部署
- 从dubbo官网下载dubbo-admin2.6.*的版本，新的管理中心还不成熟，暂时用老的版本
- maven配置编译完成后，配置好相应的配置，如注册中心地址，日志等配置，打成jar放到服务器上
-配置相应的启动脚本，启动服务即可

###zookeeper的安装部署
- 下载最新的zookeeper包，注意高版本的dubbo和curator必须是最新的3.5.5的版本，因为curator从4.*的版本开始就只支持zookeeper3.5.*的版本，否则会报错
