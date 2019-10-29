###Linux系统下Yum安装MySQL5.7数据库

- 下载yum源：wget 'https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm'   
- 安装yum源：rpm -Uvh mysql57-community-release-el7-11.noarch.rpm   
- 查看有哪些版本的mysql：yum repolist all | grep mysql   
- 安装：yum install -y mysql-community-server
- 启动mysql：systemctl/service start mysqld
- 查看mysql状态：systemctl/service status mysqld
- 查看mysql的初始密码：grep 'temporary password' /var/log/mysqld.log
- 登录mysql：mysql -uroot -p
- 修改密码：SET PASSWORD = PASSWORD('Admin123!');
- 开放端口：GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Admin123!' WITH GRANT OPTION;
- 刷新设置：flush privileges;
- 设置开启启动：
systemctl enable mysqld      
systemctl daemon-reload


参考：https://www.jianshu.com/p/531cc35b15e7
