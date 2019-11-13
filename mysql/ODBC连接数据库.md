###ODBC连接数据库

ODBC介绍      
ODBC是Open Database Connect 即开发数据库互连的简称，它是一个用于访问数据库的统一界面标准。ODBC引入一个公共接口以解决不同
数据库潜在的不一致性，从而很好的保证了基于数据库系统的应用程序的相对独立性。很多程序员都已经体会到了在Windows平台下通过
ODBC进行数据库编程开发的益处，其实在Linux/Unix下现在也有了自己的ODBC，可以使我们的数据库编程就像在Windows平台下一样简单。

直白的说就是数据库与数据库之间的直连想通

安装ODBC驱动包
在Linux上安装MariaDB Connector / ODBC       
下载二进制软件包
```text
安装过程非常简单。首先，您需要从二进制压缩包中提取文件。
然后，你需要在你的系统驱动程序的共享库安装到合适的位置。
驱动程序的共享库被调用libmaodbc.so，它位于lib目录还是lib64目录中，具体取决于您下载的是32位还是64位程序包。
该驱动程序的共享库可以安装在任何地方，但为简单起见，下面的说明将假定您将其安装到/usr/lib64，这是许多Linux发行版上64位共享
库的公用目录。
```
例如，以下命令将在RHEL或CentOS 7上下载并安装MariaDB Connector / ODBC 3.0.8：
```text
mkdir odbc_package
cd odbc_package
wget https://downloads.mariadb.com/Connectors/odbc/connector-odbc-3.0.8/mariadb-connector-odbc-3.0.8-ga-rhel7-x86_64.tar.gz
tar -xvzf mariadb-connector-odbc-3.0.8-ga-rhel7-x86_64.tar.gz
sudo install lib64/libmaodbc.so /usr/lib64/
```
对于其他Linux发行版，这些命令将是相似的。但是，程序包的URL将不同。

在Linux上安装UnixODBC¶
```text
为了在Linux上使用MariaDB Connector / ODBC，您还需要安装受支持的驱动程序管理器。我们当前在Linux上支持的唯一驱动程序管理器
是UnixODBC。在大多数Linux发行版中，可以使用Linux发行版的程序包管理器来安装UnixODBC。
```
例如，以下命令会将unixODBC软件包安装在RHEL，CentOS和类似的Linux发行版上：
```text
sudo yum install unixODBC
```

测试
```text
odbcinst -j
```
```text
[root@571345ddf744 /]# odbcinst -j
unixODBC 2.3.6
DRIVERS............: /etc/odbcinst.ini
SYSTEM DATA SOURCES: /etc/odbc.ini
FILE DATA SOURCES..: /etc/ODBCDataSources
USER DATA SOURCES..: /root/.odbc.ini
SQLULEN Size.......: 8
SQLLEN Size........: 8
SQLSETPOSIROW Size.: 8 //表明安装成功
```

在Linux上使用MariaDB Connector / ODBC创建数据源
```text
第一步是配置UnixODBC以将MariaDB Connector / ODBC识别为Driver。要配置Driver，您可以使用该odbcinst工具，该工具可以将
MariaDB Connector / ODBC的配置条目添加到系统的全局/etc/odbcinst.ini文件中。
```
例如，创建一个类似于以下内容的模板文件，其名称类似于MariaDB_odbc_driver_template.ini：
```text
[MariaDB ODBC 3.0 Driver]
Description = MariaDB Connector/ODBC v.3.0
Driver = /usr/lib64/libmaodbc.so
```
然后/etc/odbcinst.ini使用以下命令将其安装到系统的全局文件中：
```text
sudo odbcinst -i -d -f MariaDB_odbc_driver_template.ini
```


在Linux上使用UnixODBC配置DSN
```text
第二步是为MariaDB服务器使用数据源名称（DSN）配置UnixODBC。A DSN允许您集中配置服务器的所有连接参数，以便可以轻松配置如何
在环境中连接服务器。要配置DSN，您可以使用该odbcinst工具，该工具可以将给定数据源的配置条目添加到系统的全局/etc/odbc.ini文
件或用户的本地~/.odbc.ini文件中。
```
例如，创建一个类似于以下内容的模板文件，其名称类似于MariaDB_odbc_data_source_template.ini：
```text
[MariaDB-server]
Description=MariaDB server
Driver=MariaDB ODBC 3.0 Driver
SERVER=<your server>
USER=<your user>
PASSWORD=<your password>
DATABASE=<your database>
PORT=<your port>
```
然后您可以/etc/odbc.ini使用以下命令将其安装到系统的全局文件中：
```text
sudo odbcinst -i -s -l -f MariaDB_odbc_data_source_template.ini
```
或者，您可以~/.odbc.ini使用以下命令将其安装到用户的本地文件中：
```text
odbcinst -i -s -h -f MariaDB_odbc_data_source_template.ini
```

参考：
https://blog.csdn.net/mei777387/article/details/75331428
https://mariadb.com/kb/en/library/about-mariadb-connector-odbc/
https://mariadb.com/kb/en/library/creating-a-data-source-with-mariadb-connectorodbc/