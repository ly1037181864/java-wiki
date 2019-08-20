###Linux系统下Yum安装
##1 下载安装包
登录网站：http://yum.baseurl.org/下载最新的安装包yum-3.4.3.tar.gz

##2 解压安装包到安装目录
cd /opt 
tar -zxvf yum-3.4.3.tar.gz  
touch /etc/ yum.conf    
cd yum-3.4.3

##3 执行安装命令
yum install yum  
yum check-update    
yum update  
yum clean all  

参考：https://www.jianshu.com/p/c02d0bcd5a82 
