##大型sql文件导入mysql方案

- navicat远程导入
```text
操作简单，但是缺点很明显：导入效率低，严重占用本地的IO，影响机器的正常工作
```

- source导入
```text
1.在测试环境的ECS上安装一个mysql-client
2.修改mysql中的max_allowed_packet参数为10G大小，net_buffer_length参数也根据需求适度调大。
3.多个文件，可以利用一个sql脚本聚合实现，所以all.sql 的内容可以如下
source /mydata/sql/a.sql;
source /mydata/sql/b.sql;
4.为避免ssh连接掉线而导致执行关闭，需要写一个shell脚本，通过nohup后台执行。myshell.sh脚本如下
mysql -h host  -uxxx -pxxx --database=user_database</mydata/sql/all.sql
5. 后台执行指令
nohup ./myshell.sh &
```

- DMBS导入
```text
压缩单表SQL文件为单独zip文件，需要注意单个文件的大小不能超过1G
通过云服务器上传
```
[注：此方案只适合数据库为云服务器]

参考：https://www.cnblogs.com/jpfss/p/11392591.html