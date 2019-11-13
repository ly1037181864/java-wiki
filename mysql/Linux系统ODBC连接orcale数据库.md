##Linux系统ODBC连接orcale数据库

- 安装unixODBC
```text
yum install unixODBC
```
测试`odbcinst -j`
```text
unixODBC 2.3.1
DRIVERS............: /etc/odbcinst.ini
SYSTEM DATA SOURCES: /etc/odbc.ini
FILE DATA SOURCES..: /etc/ODBCDataSources
USER DATA SOURCES..: /root/.odbc.ini
SQLULEN Size.......: 8
SQLLEN Size........: 8
SQLSETPOSIROW Size.: 8
```
出现上述信息表明安装成功

- 安装oracle的ODBC驱动
```text
操作不同的数据，需要相应的数据库提供的odbc驱动。而第一步安装的unixODBC会默认安装mqsql、PostqreSQL的驱动。查看odbc配置路径：
/etc/odbcinst.ini
/etc/odbc.ini
oracle需要手动安装驱动。
```
下载orcale驱动包并安装
```text
下载安装包
oracle-instantclient19.3-basic-19.3.0.0.0-1.x86_64.rpm
oracle-instantclient19.3-devel-19.3.0.0.0-1.x86_64.rpm
oracle-instantclient19.3-odbc-19.3.0.0.0-1.x86_64.rpm
oracle-instantclient19.3-sqlplus-19.3.0.0.0-1.x86_64.rpm   //便于oracle连接、测试使用等

安装驱动包
rpm -ivh *.rpm[安装包]
```
下载地址：http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html 

- 拷贝目录
拷贝/usr/lib/oracle/19.3/client64/lib/*到/usr/lib/目录下

- 执行ldconfig :[root@localhost /]# ldconfig (首字母是小写L,不是大写i)
```text
说明：ldconfig 命令的用途,主要是在默认搜寻目录(/lib和/usr/lib)以及动态库配置文件/etc/ld.so.conf内所列的目录下,搜索出可共
享的动态链接库(格式如前介绍,lib*.so*),进而创建出动态装入程序(ld.so)所需的连接和缓存文件.缓存文件默认为 /etc/ld.so.cache,
此文件保存已排好序的动态链接库名字列表.
```

- 配置配置文件
1、配置odbc.ini
```text
[xiaoan_oracle]
Driver       = /usr/lib/libsqora.so.19.1
Description  = Data Source to Oracle
ServerName = ip:端口/orcl
UserID      = 用户名
password    = 密码


[xiaoan_main]
Driver       = /usr/lib/libsqora.so.19.1
Description  = Data Source to Oracle
#ServerName   = ip:端口/orcl
ServerName = ip:端口/orcl
UserID      = 用户名
password    = 密码
```
2、配置odbcinst.ini
```text
[PostgreSQL]
Description=ODBC for PostgreSQL
Driver=/usr/lib/psqlodbcw.so
Setup=/usr/lib/libodbcpsqlS.so
Driver64=/usr/lib64/psqlodbcw.so
Setup64=/usr/lib64/libodbcpsqlS.so
FileUsage=1
UsageCount=2

[MySQL]
Description=ODBC for MySQL
Driver=/usr/lib/libmyodbc5.so
Setup=/usr/lib/libodbcmyS.so
Driver64=/usr/lib64/libmyodbc5.so
Setup64=/usr/lib64/libodbcmyS.so
FileUsage=1
UsageCount=2
```
[注：默认实际上已经配置好了，因为安装时unixODBC会默认安装mqsql、PostqreSQL的驱动，因而会自动配置]


- 测试
```text
[root@server lib]# isql xiaoan_main SCORPIUS PvnOqeX5k0yppdD0crOw -v //testODBC为数据源名称 如odbc.ini中配置的xiaoan_main等
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL> quit
```
出现上述则表明配置成功