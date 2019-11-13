##Linux系统运维相关
- top指令 
```text
top -p7052      查看某一进程的信息     
top -m          按照内存消耗排序 
```

- 环境变量
```text
查看环境变量  
env                 所有变量    
echo $JAVA_HOME     查看Java环境变量值 
        
编辑环境变量      
vi /etc/profile     
vi ~/.bash_profile      
source命令生效 source /etc/profile source ~/.bash_profile       
export PATH=$PATH:/usr/local/mysql/bin;         控制台直接设置  
``` 
