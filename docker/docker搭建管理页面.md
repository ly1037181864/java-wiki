####docker搭建本地dockerUI管理界面

- 从dokcerHub拉取镜像
```text
docker search portainer     
```     
![avatar](../imags/docker/docker-18.png)        
```text
docker pull portainer/portainer     
```
[注意：portainer本身也是镜像]        
- 启动容器
```text
docker run -d -p 9002:9000 --restart=always  -v /var/run/docker.sock:/var/run/docker.sock  --name prtainer  portainer/portainer
```

- 登录管理界面
浏览器输入：`http://ip:9002       
首次登录时需要创建用户及密码          
![avatar](../imags/docker/docker-19.png)              
![avatar](../imags/docker/docker-20.png)   

     
![avatar](../imags/docker/docker-22.png)        
点击红色框进入详情
![avatar](../imags/docker/docker-23.png)            
本地镜像等的详细信息


![avatar](../imags/docker/docker-24.png)        
本地容器列表信息-可单击进入每个容器查看详情信息

![avatar](../imags/docker/docker-25.png)        
容器的详细信息及操作界面

![avatar](../imags/docker/docker-26.png)        
镜像的列表信息

![avatar](../imags/docker/docker-27.png)    
镜像的相信信息及操作界面

