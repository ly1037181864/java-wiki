###Linux防火墙相关

linux防火墙查看状态firewall、iptable
- iptables防火墙
```text
1、查看防火墙状态
service iptables status

2、停止防火墙
service iptables stop

3、启动防火墙
service iptables start

4、重启防火墙
service iptables restart 

5、永久关闭防火墙
chkconfig iptables off 

6、永久关闭后重启
chkconfig iptables on

7、开放端口 以80端口为例
vim /etc/sysconfig/iptables
# 加入如下代码
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
保存退出后重启防火墙

```

- firewall防火墙
```text
1、查看firewall服务状态
systemctl status firewalld
出现Active: active (running)切高亮显示则表示是启动状态。
出现 Active: inactive (dead)灰色表示停止，看单词也行。

2、查看firewall的状态
firewall-cmd --state
出现running:启动状态
其他：表示停止

3、开启、重启、关闭、firewalld.service服务
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop


4、查看防火墙规则(所有端口的开放情况)
firewall-cmd --list-all

5、查询、开放、关闭端口
# 查询端口是否开放
firewall-cmd --query-port=8080/tcp
# 开放80端口
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp
修改配置后要重启防火墙

6、重启防火墙
firewall-cmd --reload
```

