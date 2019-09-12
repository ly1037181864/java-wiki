###top命令详解

第一行，任务队列信息，同 uptime 命令的执行结果
````
top - 16:40:41 up 21 days,  5:17,  1 user,  load average: 0.01, 0.02, 0.05
系统时间：16:40:41
运行时间：up 21 days,
当前登录用户：  1 user
负载均衡(uptime)  load average: 0.01, 0.02, 0.05
average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。
load average数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了
````

第二行，Tasks — 任务（进程）
````
Tasks: 274 total,   1 running, 273 sleeping,   0 stopped,   0 zombie
总进程:274 total, 运行:1 running, 休眠:273 sleeping, 停止: 1 stopped, 僵尸进程: 0 zombie
````

第三行，cpu状态信息
````
%Cpu(s):  0.1 us,  0.0 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
0.1%us【user space】— 用户空间占用CPU的百分比。
0.0%sy【sysctl】— 内核空间占用CPU的百分比。
0.0%ni【】— 改变过优先级的进程占用CPU的百分比
99.9%id【idolt】— 空闲CPU百分比
0.0%wa【wait】— IO等待占用CPU的百分比
0.0%hi【Hardware IRQ】— 硬中断占用CPU的百分比
0.0%si【Software Interrupts】— 软中断占用CPU的百分比
````

第四行,内存状态
````
KiB Mem : 32779812k total, 24243432k free,  5420328k used,  3116052k buff/cache【缓存的内存量】

````

第五行，swap交换分区信息
````
KiB Swap: 32767996k total, 32767476k free,      520k used. 26375808k avail Mem【缓冲的交换区总量】
````

备注：
可用内存=free + buffer + cached     
对于内存监控，在top里我们要时刻监控第五行swap交换分区的used，如果这个数值在不断的变化，说明内核在不断进行内存和swap的数据交换，这是真正的内存不够用了。     
第四行中使用中的内存总量（used）指的是现在系统内核控制的内存数，第四行中空闲内存总量（free）是内核还未纳入其管控范围的数量。          
纳入内核管理的内存不见得都在使用中，还包括过去使用过的现在可以被重复利用的内存，内核并不把这些可被重新使用的内存交还到free中去，因此在linux上free内存会越来越少，但不用为此担心。     
    
第六行，空行      
第七行以下：各进程（任务）的状态监控      
````
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                                                  
  975 root      20   0       0      0      0 S   0.3  0.0   9:47.77 xfsaild/dm-2                                                                                                                                                                                             
 2991 gdm       20   0  636556  24848   9076 S   0.3  0.1  30:58.74 gsd-color                                                                                                                                                                                                
 4588 root      20   0  162144   2492   1612 R   0.3  0.0   0:00.04 top
 
PID — 进程id
USER — 进程所有者
PR — 进程优先级
NI — nice值。负值表示高优先级，正值表示低优先级
VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
SHR — 共享内存大小，单位kb
S —进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
%CPU — 上次更新到现在的CPU时间占用百分比
%MEM — 进程使用的物理内存百分比
TIME+ — 进程使用的CPU时间总计，单位1/100秒
COMMAND — 进程名称（命令名/命令行）
````

- 多U多核CPU监控
````
top命令后 在监控台按数字1键
%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu4  :  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu5  :  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu6  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu7  :  0.0 us,  0.0 sy,  0.0 ni, 99.7 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
````

- 高亮显示当前运行进程
````
敲击键盘“b”（打开/关闭加亮效果）
 PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                                                  
 5159 root      20   0  162144   2500   1612 R   1.4  0.0   0:00.03 top  (高亮显示)                                                                                                                                                                                                    
 2991 gdm       20   0  636556  24848   9076 S   0.7  0.1  30:59.74 gsd-color
````
我们发现进程id为2419的“top”进程被加亮了，top进程就是视图第二行显示的唯一的运行态（runing）的那个进程，可以通过敲击“y”键关闭或打开运行态进程的加亮效果。
b和y的区别在于b是整行即tr行都会变白，而y只是将整行tr的内容即字体变白

- 进程字段排序
````
默认进入top时，各进程是按照CPU的占用量来排序的
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                                                  
 5238 root      20   0  162144   2500   1612 R   0.7  0.0   0:00.96 top                                                                                                                                                                                                      
 9481 root      20   0 6939860 510092  14152 S   0.3  1.6   5:24.24 java                                                                                                                                                                                                     
11813 root      20   0   10.7g   1.9g  13972 S   0.3  5.9   8:43.31 java                                                                                                                                                                                                     
20543 oracle    -2   0  739516  12524  10792 S   0.3  0.0  66:31.68 oracle  
````
敲击键盘“x”（打开/关闭排序列的加亮效果）,快速找到当前命令是按照哪个字段排序的

- 通过”shift + >”或”shift + <”可以向右或左改变排序列
````
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                                                  
11813 root      20   0   10.7g   1.9g  13972 S   0.3  5.9   8:43.72 java                                                                                                                                                                                                     
 5299 root      20   0   13.4g   1.0g  10988 S   0.0  3.2  42:27.12 java                                                                                                                                                                                                     
28382 root      20   0 4995796 513296  13556 S   0.0  1.6   0:38.62 java                                                                                                                                                                                                     
 9481 root      20   0 6939860 510092  14152 S   0.0  1.6   5:24.41 java                                                                                                                                                                                                     
12857 mysql     20   0 3031284 425920   9216 S   0.3  1.3  15:39.46 mysqld                                                                                                                                                                                                   
20566 oracle    20   0  746180 267188 263120 S   0.0  0.8   1:05.61 oracle                                                                                                                                                                                                   
 2884 gdm       20   0 3984308 241788  53868 S   0.7  0.7  36:35.62 gnome-shell 
````
按一次”shift + >”的效果图,视图现在已经按照%MEM来排序，再按一次按时间排

- top交互命令
````
h 显示帮助画面，给出一些简短的命令总结说明
k 终止一个进程。
i 忽略闲置和僵死进程。这是一个开关式命令。
q 退出程序
r 重新安排一个进程的优先级别
S 切换到累计模式
s 改变两次刷新之间的延迟时间（单位为s
f或者F 从当前显示中添加或者删除项目
o或者O 改变显示项目的顺序
l 切换显示平均负载和启动时间信息
m 切换显示内存信息
t 切换显示进程和CPU状态信息
c 切换显示命令名称和完整命令行
M 根据驻留内存大小进行排序
P 根据CPU使用百分比大小进行排序
T 根据时间/累计时间进行排序
W 将当前设置写入~/.toprc文件中
````

- 常用命令
````
top -c  显示 完整命令
top -n 2 表示更新两次后终止更新显示
top -d 3 表示更新周期为3秒
````