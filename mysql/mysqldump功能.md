###mysql导出数据结构、函数、存储结构

- 导出整个数据库
```text
mysqldump -h 主机ip -u 用户名 -p 密码 数据库名 > 导出的路径  //全量导出包含数据接口、数据、存储过程等
```
如果root用户没用密码可以不写-p，当然导出的sql文件你可以制定一个路径，未指定则存放在mysql的bin目录下

- 导出某张表
```text
mysqldump -h 主机ip -u 用户名 -p 密码 数据库名 表名 > 导出的路径  
```

- 导出一个数据库结构
```text
mysqldump -h 主机ip -u 用户名 -p 密码 -d --add-drop-table 数据库名 > 导出的路径 
```
`-d 不含数据  `  
`--add-drop-table 在每个create语句之前增加一个drop table`

- 导出函数或者存储过程
```text
mysqldump -h 主机ip -u 用户名 -p 密码 -ntd -R 数据库名 > 导出的路径 
```
其中的 `-ntd` 是表示导出存储过程；`-R`是表示导出函数

[注：以上命令都需要在mysql的安装目录bin目录下执行mysqldump命令才能生效]


- 导入数据
```text
1 mysql命令
mysql -h 主机ip -u 用户名 -p 密码 数据库名 < backupfile.sql  //在mysql的安装目录bin，目录下执行该命令

2 source命令
mysql>source backupfile.sql  //在数据库操作命令窗口执行命令
```

参考文章：https://www.jianshu.com/p/6897c8c0aafa