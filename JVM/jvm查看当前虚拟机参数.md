###查看当前虚拟机参数

- jps命令
 命令格式 jps [options] [hostid]
 ```text
-l : 输出主类全名或jar路径
-q : 只输出LVMID
-m : 输出JVM启动时传递给main()的参数
-v : 输出JVM启动时显示指定的JVM参数
```
```text
[root@iZ94dupy6gtZ build-2020-06-05]# jps -l
924101 org.apache.catalina.startup.Bootstrap
405792 org.apache.catalina.startup.Bootstrap
519074 org.apache.catalina.startup.Bootstrap
510418 org.apache.catalina.startup.Bootstrap
510819 sun.tools.jps.Jps
923267 org.apache.zookeeper.server.quorum.QuorumPeerMain
737769 org.apache.catalina.startup.Bootstrap
923713 org.apache.catalina.startup.Bootstrap
923954 org.apache.catalina.startup.Bootstrap
```  
```text
[root@iZ94dupy6gtZ build-2020-06-05]# clear
[root@iZ94dupy6gtZ build-2020-06-05]# jps -m
510887 Jps -m
924101 Bootstrap start
405792 Bootstrap start
519074 Bootstrap start
510418 Bootstrap start
923267 QuorumPeerMain /home/zookeeper/zookeeper-app/bin/../conf/zoo.cfg
737769 Bootstrap start
923713 Bootstrap start
923954 Bootstrap start
```
```text
[root@iZ94dupy6gtZ build-2020-06-05]# jps -q
924101
405792
519074
510934
510418
923267
737769
923713
923954
```
```
[root@iZ94dupy6gtZ build-2020-06-05]# jps -v
924101 Bootstrap -Djava.util.logging.config.file=/home/xagdadmin/tomcats/tomcat-libra/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms128m -Xmx512m -XX:PermSize=128M -XX:MaxPermSize=256m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dcatalina.base=/home/xagdadmin/tomcats/tomcat-libra -Dcatalina.home=/usr/local/tomcat/apache-tomcat-8.5.23 -Djava.io.tmpdir=/home/xagdadmin/tomcats/tomcat-libra/temp
405792 Bootstrap -Djava.util.logging.config.file=/home/xagdadmin/tomcats/tomcat-virgo/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms128m -Xmx512m -XX:PermSize=128M -XX:MaxPermSize=256m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dcatalina.base=/home/xagdadmin/tomcats/tomcat-virgo -Dcatalina.home=/usr/local/tomcat/apache-tomcat-8.5.23 -Djava.io.tmpdir=/home/xagdadmin/tomcats/tomcat-virgo/temp
519074 Bootstrap -Djava.util.logging.config.file=/home/xagdadmin/tomcats/tomcat-aries/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms512m -Xmx1024m -XX:PermSize=512m -XX:MaxPermSize=104m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dcatalina.base=/home/xagdadmin/tomcats/tomcat-aries -Dcatalina.home=/usr/local/tomcat/apache-tomcat-8.5.23 -Djava.io.tmpdir=/home/xagdadmin/tomcats/tomcat-aries/temp
510418 Bootstrap -Djava.util.logging.config.file=/home/xagdadmin/tomcats/tomcat-scorpius/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Dhttps.protocols=TLSv1.1,TLSv1.2 -Xms512m -Xmx2024m -Xss1024K -XX:PermSize=512m -XX:MaxPermSize=2024m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dcatalina.base=/home/xagdadmin/tomcats/tomcat-scorpius -Dcatalina.home=/usr/local/tomcat/apache-tomcat-8.5.23 -Djava.io.tmpdir=/home/xagdadmin/tomcats/tomcat-scorpius/temp
923267 QuorumPeerMain -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false
737769 Bootstrap -Djava.util.logging.config.file=/home/xagdadmin/tomcats/tomcat-porface/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms128m -Xmx512m -XX:PermSize=128M -XX:MaxPermSize=256m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dcatalina.base=/home/xagdadmin/tomcats/tomcat-porface -Dcatalina.home=/usr/local/tomcat/apache-tomcat-8.5.23 -Djava.io.tmpdir=/home/xagdadmin/tomcats/tomcat-porface/temp
923713 Bootstrap -Djava.util.logging.config.file=/home/xagdadmin/tomcats/tomcat-ruleEngine/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms128m -Xmx2048m -XX:PermSize=128M -XX:MaxPermSize=256m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dcatalina.base=/home/xagdadmin/tomcats/tomcat-ruleEngine -Dcatalina.home=/usr/local/tomcat/apache-tomcat-8.5.23 -Djava.io.tmpdir=/home/xagdadmin/tomcats/tomcat-ruleEngine/temp
923954 Bootstrap -Djava.util.logging.config.file=/home/xagdadmin/tomcats/tomcat-sales/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms128m -Xmx512m -XX:PermSize=128M -XX:MaxPermSize=256m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dcatalina.base=/home/xagdadmin/tomcats/tomcat-sales -Dcatalina.home=/usr/local/tomcat/apache-tomcat-8.5.23 -Djava.io.tmpdir=/home/xagdadmin/tomcats/tomcat-sales/temp
510973 Jps -Denv.class.path=.:/usr/java/jdk1.7.0_79/jre/lib/rt.jar:/usr/java/jdk1.7.0_79/lib/dt.jar:/usr/java/jdk1.7.0_79/lib/tools.jar -Dapplication.home=/usr/java/jdk1.7.0_79 -Xms8m
```
其中jps -v命令可以查看启动时输入的jvm参数

- jinfo命令
命令格式    jinfo [option] [args] LVMID
```text
-flag : 输出指定args参数的值
-flags : 不需要args参数，输出所有JVM参数的值
-sysprops : 输出系统属性，等同于System.getProperties()
```
```text
[root@iZ94dupy6gtZ build-2020-06-05]# jinfo -flags 924101
Attaching to process ID 924101, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.79-b02

-Djava.util.logging.config.file=/home/xagdadmin/tomcats/tomcat-libra/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager 
-Xms128m -Xmx512m -XX:PermSize=128M -XX:MaxPermSize=256m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources 
-Dcatalina.base=/home/xagdadmin/tomcats/tomcat-libra -Dcatalina.home=/usr/local/tomcat/apache-tomcat-8.5.23 -Djava.io.tmpdir=/home/xagdadmin/tomcats/tomcat-libra/temp

```