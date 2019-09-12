##Java虚拟机监控相关

###相关的监控命令
- jps   
显示所有Java相关的进程   
jps -l 显示所有Java相关的进程并输出相关的类的全路径

- jinfo  
jinfo -flags pid 输出所有的虚拟机参数
jinfo -flag 虚拟机参数名 pid 输出指定虚拟机参数信息，如是否开启打印gc日志功能，或者当前的新生代垃圾收集器等

- jstat  
jstat -gc pid 1000 5            输出指定线程pid的gc收集信息，每秒刷新一次，共计打印5次 如果不写5则会每秒一直刷新下去  
````
S0C: Current survivor space 0 capacity (kB).表示survivor 0区的总大小   
S1C: Current survivor space 1 capacity (kB).表示survivor 1区的总大小   
S0U: Survivor space 0 utilization (kB).表示survivor 0区使用了的大小  
S1U: Survivor space 1 utilization (kB).表示survivor 1区使用了的大小  
EC: Current eden space capacity (kB).表示eden区总大小 
EU: Eden space utilization (kB).表示eden区使用了的大小   
OC: Current old space capacity (kB).表示old区总大小   
OU: Old space utilization (kB).表示old区使用了的大小 
MC: Metaspace capacity (kB).表示Metaspace区总大小 
MU: Metacspace utilization (kB).表示Metaspace区使用了的大小  
CCSC: Compressed class space capacity (kB).表示压缩类空间总量    
CCSU: Compressed class space used (kB).表示压缩类空间使用量   
YGC: Number of young generation garbage collection events.表示Young GC的次数
YGCT: Young generation garbage collection time.表示Young GC的时间
FGC: Number of full GC events.表示full GC的次数
FGCT: Full garbage collection time.表示full GC的时间
GCT: Total garbage collection time.表示总的 GC的时间
````
jstat -gcutil pid 1000 5        输出指定线程pid的gc收集信息(相对简单)，每秒刷新一次，共计打印5次 如果不写5则会每秒一直刷新下去
jstat -class 29159 1000 10      查看一个进程id为29159的java进程，每隔1s输出，一共输出10次
````
Loaded：表示类加载的个数 
Bytes：表示类加载的大小，单位为kb    
UnLoaded：表示类卸载的个数   
Bytes：表示类卸载的大小，单位为kb    
Time：表示类加载和卸载的时间    
````
jstat -compiler pid 1000 5
````
Compiled：表示编译成功的方法数量
Failed：表示编译失败的方法数量
Invalid：表示编译无效的方法数量
Time：编译所花费的时间
FailedType：编译失败类型
FailedMethod：编译失败方法
````
    
- jmap  
jmap -heap pid  查看指定线程的堆信息
jmap -dump:format=b,file=路径/heap.hprof 进程id

- jstack    
jstack pid  查看指定线程的线程栈信息
````
localhost-startStop-1：线程名
daemon：后台线程
prio：优先级
os_prio：系统优先级
tid：线程id
nid：操作系统id
java.lang.Thread.State：线程状态；NEW-线程尚未启动， RUNNABLE- 线程运行中，BLOCKED-等待一个锁， WAITINH-等待另一个线程， TIMED_WAITING-限时等待另一个线程， TERMINATED-已退出
````
jstack 7930 > 7930.txt  导出线程栈信息
top -p 7930 -H  打印线程栈信息

- jconsole  
监控平台，主要是监控堆、新生代、老生带、元空间内存使用情况、线程栈的运行情况还有就是class信息等
- jvisualvm     
监控平台，主要是监控堆的活动情况，还有就是cpu和gc的情况，以及提供内存溢出分析

- 远程虚拟机监控配置     
-Dfile.encoding=utf-8 -Djava.rmi.server.hostname=10.10.0.237    
-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1099     
-Dcom.sun.management.jmxremote.authenticate=false   
-Dcom.sun.management.jmxremote.ssl=false    

- JVM运行时参数查看      
-XX:+printFlagsInitial 查看JVM运行时初始值  
-XX:+printFlagsFinal 查看JVM运行时最终值    
-XX:+UnlockExperimentalVMOptions 解锁实验参数 
-XX:+UnlockDiagnosticVMOptions 解锁诊断参数   
-XX:+PrintCommandLineFlags 打印命令行参数  
如查看JVM版本信息：java -XX:+PrintFlagsFinal -version > ~/version.txt   

####相关虚拟机参数
#####虚拟机参数  
- 跟踪垃圾回收    
-XX:+PrintGC                                    使用这个参数启动Java虚拟机后，只要遇到GC，就会打印日志。    
-XX:+PrintGCDetails                             获取更详细的GC信息。 
-XX:+PrintHeapAtGC                              可以在每次GC前后分别打印堆的信息。   
-XX:+PrintGCTimeStamps                          会在每次GC发生时，额外输出GC发生的时间，该输出时间为虚拟机启动后的时间偏移量。    
-XX:+PrintGCApplicationConcurrentTime           可以打印应用程序的执行时间 
-XX:+PrintGCApplicationStoppedTime              可以打印应用程序由于GC而产生的停顿时间。    
-XX:+PrintReferenceGC                           用来跟踪系统内的软引用、弱引用、虚引用和Finallize队列。 
-XX:+UseParallelGC                              指定垃圾收集器(7大垃圾收集器，4大垃圾收集算法) 
-XX:+UseParrallelOldGC                          指定老年代ParrallelOld垃圾收集器
-XX:+UseConcMarkSweepGC                         指定老年代的GC收集器为CMS
-XX:+UseParNewGC                                指定新生代GC收集器为ParNew
-XX:+UseG1GC                                    指定G1收集器
-XX:+UseSerialGC                                指定新生代Serial收集器
-XX:+UseSerialOldGC                             指定老年代SerialOld收集器
-XX:ParallelGCThreads                           限制GC收集器的线程数
-XX:MaxGCPauseMillis                            设定GC最大停顿时间（以牺牲吞吐量为代价）
-XX:GCTimeRatio                                 设定GC吞吐量，计算方式为吞吐量=1/(1+n)，n为设定的值
-XX:CMSInitiatingOccupancyFraction              用于设置触发GC的百分比，在jdk 1.6中，这个值时92%

- 类加载/卸载的跟踪  
-XX:+TraceClassLoading                  跟踪类的加载。  
-XX:+TraceClassUnloading                跟踪类的卸载。    
-XX:+PrintClassHistogram                打印、查看系统中类的分布情况。    

- 系统参数查看    
-XX:+PrintVMOptions：在运行时，打印虚拟机接收到命令行显示参数。   
-XX:+PrintCommandLineFlags：打印传递给虚拟机的显式和隐式参数，隐式参数未必是通过命令行直接给出的，它可能是由虚拟机启动时自行设置的。     
-XX:+PrintFlagsFinal：打印所有的系统参数的值。

- 堆参数配置     
-Xmx                                最大堆空间大小
-Xms                                初始堆空间大小。 在实际工作中，可以直接将初始堆和最大堆空间设置相等，这样做可以减少垃圾回收次数，提高程序性能。    
-Xmn                                新生代的大小。新生代的大小会直接影响到老年代的大小，这个参数堆系统性能以及GC行为由很大的影响。新生代的大小一般设置为整个堆空间的1/3到1/4左右。
-Xss                                指定线程的栈大小。   
-XX:SurvivorRatio                   新生代中eden空间和from/to空间的比例关系。    
-XX:+HeapDumpOnOutOfMemoryError     在内存溢出时导出整个堆信息。  
-XX:+HeapDumpPath                   指定堆的存放路径。
-XX:+HeapDumpOnCtrlBreak            使用Ctrl+Break键可以让虚拟机生成heapdump文件     
-XX:MaxMetaspaceSize                指定元空间的最大可用值。   
-XX:MaxDirectMemorySize             最大可用直接内存，如不设置，默认值为最大堆空间。直接内存的溢出依然会引起系统的OOM。
-XX:InitialHeapSize                 初始化的堆大小     
-XX:MaxHeapSize                     最大堆大小   
-XX:MaxNewSize                      最大新生代堆大小    
-XX:MinHeapDeltaBytes                       
-XX:NewSize                         新生代大小             
-XX:OldSize                         老生代大小   
-XX:NewRatio                        新生代（Eden+2S）和老年代的比值，4表示1：4，则整个新生代占整个堆的1/5  
-XX:MaxPermSize                     永久代上限
-XX:PermSize                        永久代大小
-XX:+/-UseTLAB                      是否使用TLAB来创建对象(线程本地分配缓存)
-XX:PretenureSizeThreshold          晋升老年代对象大小  

- 其他虚拟机参数   
-client和-server：分别指定虚拟机使用Client模式和Server模式。
-XX:+UseCompressedClassPointers     压缩指针，起到节约内存占用   
-XX:+UseCompressedOops              压缩指针，起到节约内存占用
-XX:CICompilerCount=4   
-XX:+ManagementServer 

- 相关资料：
````
jdk8工具集
https://docs.oracle.com/javase/8/docs/technotes/tools/unix/index.html
Troubleshooting
https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/
jps
https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jps.html
jinfo
https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jinfo.html
jstat
https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html
jmap：
https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html
mat:
http://www.eclipse.org/mat/downloads.php
jstack：
https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstack.html
java线程的状态
https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr034.html
java线程状态转化：
https://mp.weixin.qq.com/s/GsxeFM7QWuR--Kbpb7At2w
死循环导致CPU负载高
https://blog.csdn.net/goldenfish1919/article/details/8755378
正则表达式导致死循环：
https://blog.csdn.net/goldenfish1919/article/details/49123787
````

