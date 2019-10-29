##Linux系统负载相关

一、什么是Load Average   
系统负载（System Load）是系统CPU繁忙程度的度量，即有多少进程在等待被CPU调度（进程等待队列的长度）。
平均负载（Load Average）是一段时间内系统的平均负载，这个一段时间一般取1分钟、5分钟、15分钟。[ 书本上为当前时间、5分钟前、10分钟前 ]

二、如何查看Load     
top命令，w命令，uptime等命令都可以查看系统负载： 
```text
[shenjian@dev02 ~]$ uptime      
13:53:39 up 130 days,  2:15,  1 user,  load average: 1.58, 2.58, 5.58  
```  
如上所示，dev02机器1分钟平均负载，5分钟平均负载，15分钟平均负载分别是1.58、2.58、5.58

三、Load的数值是什么含义     
把CPU比喻成一条（单核）马路，进程任务比喻成马路上跑着的汽车，Load则表示马路的繁忙程度：     
Load小于1：表示完全不堵车，汽车在马路上跑得游刃有余：   
![avatar](../imags/Linux/load-01.png)   [ Load<1，单核]       
Load等于1：马路已经没有额外的资源跑更多的汽车了：     
![avatar](../imags/Linux/load-02.png)   [ Load=1，单核]    
Load大于1：汽车都堵着等待进入马路：        
![avatar](../imags/Linux/load-03.png)   [Load>1，单核]         
如果有两个CPU，则表示有两条马路，此时即使Load大于1也不代表有汽车在等待：                
![avatar](../imags/Linux/load-04.png)   [Load==2，双核，没有等待]

四、什么样的Load值得警惕（单核）     
Load < 0.7时：系统很闲，马路上没什么车，要考虑多部署一些服务         
0.7 < Load < 1时：系统状态不错，马路可以轻松应对         
Load == 1时：系统马上要处理不多来了，赶紧找一下原因      
Load > 5时：马路已经非常繁忙了，进入马路的每辆汽车都要无法很快的运行          


五、三个Load值要先看哪一个        
结合具体情况具体分析：     
1）1分钟Load>5，5分钟Load<1，15分钟Load<1：短期内繁忙，中长期空闲，初步判断是一个“抖动”，或者是“拥塞前兆”       
2）1分钟Load>5，5分钟Load>1，15分钟Load<1：短期内繁忙，中期内紧张，很可能是一个“拥塞的开始”          
3）1分钟Load>5，5分钟Load>5，15分钟Load>5：短中长期都繁忙，系统“正在拥塞”           
4）1分钟Load<1，5分钟Load>1，15分钟Load>5：短期内空闲，中长期繁忙，不用紧张，系统“拥塞正在好转”   
书本上：        
1以下表示系统大部分时间处于空闲时间      
1-2表示系统正好以它的能力运行        
2-3表示系统轻度负载     
10以上表示系统严重过载        
针对不同的系统，过载的标准不同，目前大部分认为，load average不应该大于系统的CPU个数*2   


六、Load总结        
![avatar](../imags/Linux/load-01.png)   [ Load<1，单核]  
![avatar](../imags/Linux/load-02.png)   [ Load=1，单核]   
![avatar](../imags/Linux/load-03.png)   [Load>1，单核]    
![avatar](../imags/Linux/load-04.png)   [Load==2，双核，没有等待]      