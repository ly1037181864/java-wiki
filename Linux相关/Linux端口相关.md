##Linux端口相关

查看端口情况
- netstat命令

netstat命令各个参数说明如下
```text
-t : 指明显示TCP端口
-u : 指明显示UDP端口
-l : 仅显示监听套接字(所谓套接字就是使应用程序能够读写与收发通讯协议(protocol)与资料的程序)
-p : 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序。
-n : 不进行DNS轮询，显示IP(可以加速操作)
```

显示当前服务器上所有端口及进程服务，于grep结合可查看某个具体端口及服务情况
```text
netstat -ntlp              //查看当前所有tcp端口
netstat -ntulp |grep 80    //查看所有80端口使用情况
netstat -an | grep 3306    //查看所有3306端口使用情况

```

查看一台服务器上面哪些服务及端口
```text
netstat  -lanp
```

查看一个服务有几个端口。比如要查看mysqld
```text
ps -ef |grep mysqld
```

查看某一端口的连接数量,比如3306端口
```text
netstat -pnt |grep :3306 |wc
```

查看某一端口的连接客户端IP 比如3306端口
```textbcprov
netstat -anp |grep 3306
```

查看网络端口 
```text
netstat -an 
```

```text
netstat -nupl  (UDP类型的端口)
netstat -ntpl  (TCP类型的端口)
netstat -anp 显示系统端口使用情况
```

查看指定端口运行的程序，同时还有当前连接
```text
lsof -i :port
```

```text
nmap 端口扫描
```

- 查看端口是否可以访问
```text
telnet ip 端口号 （如本机的35465：telnet localhost 35465）
开放的端口位于/etc/sysconfig/iptables中
查看时通过 more /etc/sysconfig/iptables 命令查看
```
[注：iptables不一定存在，可以通过firewalld防火墙相关的命令去操作]

- 使用案例
```text
1.使用lsof
lsof -i:端口号                     查看某个端口是否被占用

2.使用netstat
使用netstat -anp|grep 80
netstat -tunlp |grep 端口号，用于查看指定的端口号的进程情况，如查看8000端口的情况，netstat -tunlp |grep 8000
```


