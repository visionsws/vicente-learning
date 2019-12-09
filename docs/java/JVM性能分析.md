### 查看Linux环境信息

1、查看cpu的情况
   2个插槽，每个8核

```
cat /proc/cpuinfo
physical id     : 0 //物理ID
siblings        : 8
core id         : 0
cpu cores       : 8  //8核
```

或者

```
# lscpu    //cpu统计信息
# fdisk  -l       //查看硬盘及分区信息
# lsblk    //查看硬盘与分区信息
# cat /proc/meminfo      //查看内存使用量和交换区使用量
#free –h    //查看统计内存信息
```



2、查看内存的情况

```
free -h
```

**used + free + buff 基本等于 total**

- used是被使用的
- free是完全没有被使用的
- shared是被程序之间可以(已经被)共享使用的
- buffers是指用来给块设备做的缓冲大小，它只记录文件系统的metadata以及 tracking in-flight pages
- cached是用来给文件做缓冲

也就是 buffers是用来存储目录里面有什么内容，权限等等。

而cached直接用来缓存我们打开的文件

**available**才是你的"可用内存" , 而不是像过去那样简单的把free和buffer加起来

available 小于 free+buffer 是一定的了



## 配置调整

#### 配置tomcat内存

修改/data/apps/tomcat-linux/bin/catalina.sh
添加上
配置成一样的, 省得没事儿就去动态调整堆大小, 一定程度上可以提高一点性能

```
JAVA_OPTS="$JAVA_OPTS -Xms32000M -Xmx32000M"
```

配置gc日志，在catalina.sh上增加

```
JAVA_OPTS="$JAVA_OPTS -Xms32000M -Xmx32000M -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintGCDetails -Xloggc:$CATALINA_HOME/logs/gc.%p.log"
```

在JVM的启动参数中加入-XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCApplicationStoppedTime，按照参数的顺序分别输出GC的简要信息，GC的详细信息、GC的时间信息及GC造成的应用暂停的时间。

要设置CATALINA_HOME的环境变量

安装完成后需要配置一下环境变量，编辑/etc/profile文件：
在文件尾部添加如下配置：

```
JAVA_HOME=/data/apps/jdk1.8.0_111
JRE_HOME=/data/apps/jdk1.8.0_111/jre
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$JAVA_HOME/bin:$PATH
CATALINA_HOME=/data/apps/apache-tomcat-8.5.39
export PATH JAVA_HOME CLASSPATH CATALINA_HOME
```

执行

```
source /etc/profile
```

查看：export

使用catalina命令重启项目

```
 ./catalina.sh start
```

#### 参考配置

#-Xms16384m：设置JVM初始内存为16384m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。
#-Xmx16384m：设置JVM最大可用内存为16384M。
#-Xmn6144m：设置年轻代大小为6144M。整个JVM内存大小=年轻代大小 + 年老代大小 + 持久代大小。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。
#-Xss128k：设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。
#-XX:+UseParallelGC：选择垃圾收集器为并行收集器。此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。
#-XX:ParallelGCThreads=20：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。
#JAVA_OPTS="-Xms16384m -Xmx16384m -Xmn6144m -Xss5m -XX:+UseParallelGC -XX:ParallelGCThreads=8"
JAVA_OPTS="-Xms22528m -Xmx22528m -Xmn5632m -Xss5m -XX:+UseParallelGC -XX:ParallelGCThreads=8"

**访问：[GC分析网站](https://gceasy.io/index.jsp#banner) 可以上传GC日志来分析系统的GC情况**

#### 配置tomcat的线程数

打开%Tomcat_HOME%/conf/server.xml文档，找到<Connector port="8080"....>一栏。

在Connector port = "8080"后面加上相应地参数控制线程数，控制参数如下：

| 参数        | 含义                                               | 默认值 |
| ----------- | -------------------------------------------------- | ------ |
| maxThreads  | tomcat起动的最大线程数，即同时处理的任务个数       | 200    |
| acceptCount | 当tomcat起动的线程数达到最大时，接受排队的请求个数 | 100    |

要调整Tomcat的默认最大连接数，可以增加这两个属性的值，并且使acceptCount大于等于maxThreads，设置完成后如下：

```
    <Connector port="8080" protocol="HTTP/1.1" 
               connectionTimeout="20000" 
               redirectPort="8443" acceptCount="500" maxThreads="400" />
```

#### 



### 监控工具

#### uptime

```
`[root@centos7_template ~]``# uptime 10:31:42 up 4 days, 1:01, 1 user,load average: 0.02, 0.02, 0.05`
```

该命令将显示目前服务器持续运行的时间，以及负载情况。

```
10:31:42    ``//``当前系统时间
up 4 days, 1:01 ``//``持续运行时间,时间越大，说明你的机器越稳定。
1 user  ``//``用户连接数，是总连接数而不是用户数
load average: 0.02, 0.02, 0.05  ``//``系统平均负载，统计最近1，5，15分钟的系统平均负载
```

通过这个命令，可以最简便的看出系统当前基本状态信息，这里面最有用是负载指标，如果你还想查看当前系统的CPU/内存以及相关的进程状态，可以使用TOP命令。

#### TOP

通过TOP命令可以详细看出当前系统的CPU、内存、负载以及各进程状态（PID、进程占用CPU、内存、用户）等。从上面的结果看出该系统上安装了MySQL、java，可以看到他们各自的进程ID，假如这时Java进程占用较高的CPU和内存，那么你就要留心了，如果程序中没有计算量特别大、占用内存特别多的代码，可能你的java程序出现了未知的问题，可以根据进程ID做进一步的跟踪。除了通过TOP命令找到进程信息以外，还可以通过jdk自带的工具JPS直接找到java程序的进程号。



#### JPS

可以看到jps命令直接罗列出了当前系统中存在的java进程，通过这种方法查询到java程序的进程ID后，可以进一步通过：

top -p 3618 // 这里的3618就是上面查询到的java程序的进程ID

过此方法可以准确的查看指定java进程的CPU/内存使用情况。

除此之外，vmstat命令也可以查看系统CPU/内存、swap、io等情况



#### vmstat

```
vmstat 1 4
```

上面的命令每隔1秒采样一次，一共采样四次。

CPU占用率很高，上下文切换频繁，说明系统有线程正在频繁切换，这可能是你的程序开启了大量的线程存在资源竞争的情况。另外swap也是值得关注的指标，如果swpd过高则可能系统能使用的物理内存不足，不得不使用交换区内存，还有一个例外就是某些程序优先使用swap，导致swap飙升，而物理内存还有很多空余，这些情况是需要注意的。



#### pidstat

查看系统指标，还有一个第三方工具：pidstat，这个工具还是很好用的，需要先安装：

```
yum install sysstat
```

运行

```
pidstat -p 3618 -u 1 4
```

该命令监控进程id为3618的CPU状态，每隔1秒采样一次，一共采样四次。“%CPU”表示CPU使用情况，“CPU”则表示正在使用哪个核的CPU，这里为0表示正在使用第一个核。如果还要显示线程ID，则可以使用：

```
pidstat -p 3618 -u -t 1 4
```

如果要监控磁盘读写情况，这可以使用：

```
pidstat -p 3618 -d 1 4
```

pidstat还有其他的参数，可以通过pidstat --help获取，再次不再赘述。



### JDK自带工具

#### jstat

jstat：用于输出java程序内存使用情况，包括新生代、老年代、元数据区容量、垃圾回收情况。

可以实时监测系统资源占用与jvm运行情况

```
jstat -gcutil 3618 2000 20
```

上述命令输出进程ID为3618的内存使用情况（每2000毫秒输出一次，一共输出20次）

- S0：幸存1区当前使用比例
- S1：幸存2区当前使用比例
- E：伊甸园区使用比例
- O：老年代使用比例
- M：元数据区使用比例
- CCS：压缩使用比例
- YGC：年轻代垃圾回收次数
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

#### jmap

jmap：用于输出java程序中内存对象的情况，包括有哪些对象，对象的数量。

```
jmap -histo 3618
```

上述命令打印出进程ID为3618的内存情况。但我们常用的方式是将指定进程的内存heap输出到外部文件，再由专门的heap分析工具进行分析,例如mat（Memory Analysis Tool），所以我们常用的命令是：

```
jmap -dump:live,format=b,file=heap.hprof 3618
```

将heap.hprof传输出来到window电脑上使用mat工具分析：

或者使用命令生成dump文件

```
jmap -dump:format=b,file=文件名 pid
```

可采用
```
jmap -histo pid > a.log
```
日志将其保存到文件中，在一段时间后，使用文本对比工具，可以对比出GC回收了哪些对象

  -finalizerinfo 打印正等候回收的对象的信息
$jmap -finalizerinfo 3772

```
Attaching to process ID 3772, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.0-b11
Number of objects pending for finalization: 0 (等候回收的对象为0个)
```
Ø  -heap 打印heap的概要信息，GC使用的算法，heap的配置及wise heap的使用情况.
$jmap –heap 3772

```
using parallel threads in the new generation.  ##新生代采用的是并行线程处理方式
using thread-local object allocation.   
Concurrent Mark-Sweep GC   ##同步并行垃圾回收
 
Heap Configuration:  ##堆配置情况
   MinHeapFreeRatio = 40 ##最小堆使用比例
   MaxHeapFreeRatio = 70 ##最大堆可用比例
   MaxHeapSize      = 2147483648 (2048.0MB) ##最大堆空间大小
   NewSize          = 268435456 (256.0MB) ##新生代分配大小
   MaxNewSize       = 268435456 (256.0MB) ##最大可新生代分配大小
   OldSize          = 5439488 (5.1875MB) ##老生代大小
   NewRatio         = 2  ##新生代比例
   SurvivorRatio    = 8 ##新生代与suvivor的比例
   PermSize         = 134217728 (128.0MB) ##perm区大小
   MaxPermSize      = 134217728 (128.0MB) ##最大可分配perm区大小

Heap Usage: ##堆使用情况
New Generation (Eden + 1 Survivor Space):  ##新生代（伊甸区 + survior空间）
   capacity = 241631232 (230.4375MB)  ##伊甸区容量
   used     = 77776272 (74.17323303222656MB) ##已经使用大小
   free     = 163854960 (156.26426696777344MB) ##剩余容量
   32.188004570534986% used ##使用比例

Eden Space:  ##伊甸区
   capacity = 214827008 (204.875MB) ##伊甸区容量
   used     = 74442288 (70.99369812011719MB) ##伊甸区使用
   free     = 140384720 (133.8813018798828MB) ##伊甸区当前剩余容量
   34.65220164496263% used ##伊甸区使用情况

From Space: ##survior1区
   capacity = 26804224 (25.5625MB) ##survior1区容量
   used     = 3333984 (3.179534912109375MB) ##surviror1区已使用情况
   free     = 23470240 (22.382965087890625MB) ##surviror1区剩余容量
   12.43827838477995% used ##survior1区使用比例

To Space: ##survior2 区
   capacity = 26804224 (25.5625MB) ##survior2区容量
   used     = 0 (0.0MB) ##survior2区已使用情况
   free     = 26804224 (25.5625MB) ##survior2区剩余容量
   0.0% used ## survior2区使用比例

concurrent mark-sweep generation: ##老生代使用情况
   capacity = 1879048192 (1792.0MB) ##老生代容量
   used     = 30847928 (29.41887664794922MB) ##老生代已使用容量
   free     = 1848200264 (1762.5811233520508MB) ##老生代剩余容量
   1.6416783843721663% used ##老生代使用比例

Perm Generation: ##perm区使用情况 永久代
   capacity = 134217728 (128.0MB) ##perm区容量 
   used     = 47303016 (45.111671447753906MB) ##perm区已使用容量
   free     = 86914712 (82.8883285522461MB) ##perm区剩余容量
   35.24349331855774% used ##perm区使用比例
```

Ø  -histo[:live] 打印每个class的实例数目,内存占用,类全名信息. VM的内部类名字开头会加上前缀”*”. 如果live子参数加上后,只统计活的对象数量. 

$jmap–histo:live 3772
```
num     #instances         #bytes  class name
   1:         65220        9755240  <constMethodKlass>
   2:         65220        8880384  <methodKlass>
   3:         11721        8252112  [B
   4:          6300        6784040  <constantPoolKlass>
   5:         75224        6218208  [C
```




#### jstack

jstack：用户输出java程序线程栈的情况，常用于定位因为某些线程问题造成的故障或性能问题。

```
jstack 3618 > jstack.out
```

上述命令将进程ID为3618的栈信息输出到外部文件，便于传输到windows电脑上进行分析。



### Windows环境下的监控工具

Windows环境下的监控工具也有很多，但是本文主要推荐jvisualvm.exe、MemoryAnalyzer.exe，有了他们其他工具几乎不需要了。



#### jvisualvm

jvisualvm.exe在JDK安装目录下的bin目录下面，双击即可打开。

双击左侧你需要监控的java程序即可对它进行监控，这个工具包括对CPU、内存、线程、类都做了监控，功能非常强大，上文中介绍的所有功能，其他在这个工具上都已经有了。当然怎么用、如何分析它需要花时间去一点点积累。



#### MemoryAnalyzer

MemoryAnalyzer.exe：上文我们已经提到，常用于分析内存堆使用情况，也是非常强大的工具。详细使用方法，这里就不再赘述，可以下载下来尝试一下。

上述介绍了基于Linux、Windows环境的监控工具，有了这些工具我们就要利用他们做对应的事情，下面将通过一个简单的案例，说明如何使用他们。





### 其他

打堆栈

1、查出进程

ps -ef | grep tomcat

2、打堆栈，一般每隔3秒打一个，打三至四个

jstack pid > a1.txt

jstack pid > a2.txt

jstack pid > a3.txt

/data/apps/tomcat-linux-jdk/jdk1.8.0_192/bin/jstack 17973



tar -zcvf /data/apps/tomcat-linux/logs/cataout20190716.tar.gz /data/apps/tomcat-linux/logs/catalina.out



```
while true; do (echo "%CPU %MEM ARGS $(date)" && ps -e -o pcpu,pmem,args --sort=pcpu | cut -d" " -f1-5 | tail) >> ps.log; sleep 5; done
```



```
while true; do uptime >> uptime.log; sleep 1; done
```



### 案例分析

首先通过上述的介绍，我们对故障排查流程应该有了一个印象，这里先梳理出来：

[![image](http://images2017.cnblogs.com/blog/352511/201709/352511-20170901171925202-246051550.png)](http://images2017.cnblogs.com/blog/352511/201709/352511-20170901171924671-781490871.png)

案例：



一个java应用启动以后，使用人员发现应用不可用，针对该现象我们做以下分析排查：

1、首先查看服务器上的应用状态。使用jps命令查询当前在运行中的java进程：

[![image](http://images2017.cnblogs.com/blog/352511/201709/352511-20170901174151046-1496475117.png)](http://images2017.cnblogs.com/blog/352511/201709/352511-20170901174150515-1659080977.png)

这里进程ID为6400的java应用就是我们刚启用的，说明应用并没有挂掉，还在运行中。



2、通过进程ID查询所占用的CPU、内存以及当前负载情况,top -p 6400。

[![image](http://images2017.cnblogs.com/blog/352511/201709/352511-20170901174152515-1915226687.png)](http://images2017.cnblogs.com/blog/352511/201709/352511-20170901174151780-1234862760.png)

从以上结果看出该应用并没有引起系统负载过高，CPU、内存也没有出现异常情况。

3、通过上述结果我们推测因为内存原因引起的故障可性能较小，所以我们优先排查线程栈，使用jstack命令，导出线程栈。

jstack 6400 > stack.out

我们将该文件传输出来便于查看。

![picture](http://images2017.cnblogs.com/blog/352511/201709/352511-20170901171926718-1218927452.png)

查看线程栈可以看出，主线程处于运行状态，而子线程ThreadA、ThreadB、ThreadC、ThreadD一边在等待一个锁，同时又持有另外一个锁，其实看到这里我们基本推断该应用程序存在死锁，因此造成线程等待，应用不可用。通过以上栈的信息，我们就可以到程序代码中详细查看代码了，并且修改bug解决此问题。

死锁原理补充：

![image](http://images2017.cnblogs.com/blog/352511/201709/352511-20170901171928421-1145072913.png)
如图所示，造成死锁的原因是线程之间存在相互制约的情况，而任一线程都无法继续执行。



1. 堆大小设置

   JVM 中最大堆大小有三方面限制：相关操作系统的数据模型（32-bt还是64-bit）限制；系统的可用虚拟内存限制；系统的可用物理内存限制。32位系统下，一般限制在1.5G~2G；64为操作系统对内存无限制。我在Windows Server 2003 系统，3.5G物理内存，JDK5.0下测试，最大可设置为1478m。

   典型设置：

   - java **-Xmx3550m -Xms3550m -Xmn2g** **-Xss128k**
     **-****Xmx3550m**：设置JVM最大可用内存为3550M。
     **-Xms3550m**：设置JVM促使内存为3550m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。
     **-Xmn2g**：设置年轻代大小为2G。**整个JVM内存大小=年轻代大小 + 年老代大小 + 持久代大小**。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。
     **-Xss128k**：设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。
   - java -Xmx3550m -Xms3550m -Xss128k **-XX:NewRatio=4 -XX:SurvivorRatio=4 -XX:MaxPermSize=16m -XX:MaxTenuringThreshold=0**
     **-XX:NewRatio=4**:设置年轻代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）。设置为4，则年轻代与年老代所占比值为1：4，年轻代占整个堆栈的1/5
     **-XX:SurvivorRatio=4**：设置年轻代中Eden区与Survivor区的大小比值。设置为4，则两个Survivor区与一个Eden区的比值为2:4，一个Survivor区占整个年轻代的1/6
     **-XX:MaxPermSize=16m**:设置持久代大小为16m。
     **-XX:MaxTenuringThreshold=0**：设置垃圾最大年龄。**如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代**。对于年老代比较多的应用，可以提高效率。**如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间**，增加在年轻代即被回收的概论。

2. 回收器选择

   JVM给了三种选择：

   串行收集器、并行收集器、并发收集器

   ，但是串行收集器只适用于小数据量的情况，所以这里的选择主要针对并行收集器和并发收集器。默认情况下，JDK5.0以前都是使用串行收集器，如果想使用其他收集器需要在启动时加入相应参数。JDK5.0以后，JVM会根据当前

   系统配置

   进行判断。

   1. 吞吐量优先

      的并行收集器

      如上文所述，并行收集器主要以到达一定的吞吐量为目标，适用于科学技术和后台处理等。

      典型配置

      ：

      - java -Xmx3800m -Xms3800m -Xmn2g -Xss128k **-XX:+UseParallelGC -XX:ParallelGCThreads=20**
        **-XX:+UseParallelGC**：选择垃圾收集器为并行收集器。**此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。****-XX:ParallelGCThreads=20**：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。
      - java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20 **-XX:+UseParallelOldGC****-XX:+UseParallelOldGC**：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。
      - java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC  **-XX:MaxGCPauseMillis=100****-XX:MaxGCPauseMillis=100****:**设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。
      - java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC  -XX:MaxGCPauseMillis=100 **-XX:+UseAdaptiveSizePolicy-XX:+UseAdaptiveSizePolicy**：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。

   2. 响应时间优先

      的并发收集器

      如上文所述，并发收集器主要是保证系统的响应时间，减少垃圾收集时的停顿时间。适用于应用服务器、电信领域等。

      典型配置

      ：

      - java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:ParallelGCThreads=20 **-XX:+UseConcMarkSweepGC -XX:+UseParNewGC****-XX:+UseConcMarkSweepGC**：设置年老代为并发收集。测试中配置这个以后，-XX:NewRatio=4的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。
        **-XX:+UseParNewGC**:设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。
      - java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseConcMarkSweepGC **-XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection**
        **-XX:CMSFullGCsBeforeCompaction**：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。
        **-XX:+UseCMSCompactAtFullCollection**：打开对年老代的压缩。可能会影响性能，但是可以消除碎片

3. 辅助信息

   JVM提供了大量命令行参数，打印信息，供调试使用。主要有以下一些：

   - -XX:+PrintGC

     输出形式

     ：[GC 118250K->113543K(130112K), 0.0094143 secs]

     ​                **[Full GC 121376K->10414K(130112K), 0.0650971 secs]**

   - -XX:+PrintGCDetails

     输出形式

     ：[GC [DefNew: 8614K->781K(9088K), 0.0123035 secs] 118250K->113543K(130112K), 0.0124633 secs]

     ​                **[GC [DefNew: 8614K->8614K(9088K), 0.0000665 secs][Tenured: 112761K->10414K(121024K), 0.0433488 secs] 121376K->10414K(130112K), 0.0436268 secs]**

   - **-XX:+PrintGCTimeStamps** -XX:+PrintGC：PrintGCTimeStamps可与上面两个混合使用
     输出形式：**11.851: [GC 98328K->93620K(130112K), 0.0082960 secs]**

   - **-XX:+PrintGCApplicationConcurrentTime:**打印每次垃圾回收前，程序未中断的执行时间。可与上面混合使用
     输出形式：**Application time: 0.5291524 seconds**

   - **-XX:+PrintGCApplicationStoppedTime**：打印垃圾回收期间程序暂停的时间。可与上面混合使用
     输出形式：**Total time for which application threads were stopped: 0.0468229 seconds**

   - **-XX:PrintHeapAtGC**:打印GC前后的详细堆栈信息

   - **-Xloggc:filename**:与上面几个配合使用，把相关日志信息记录到文件以便分析。

4. 常见配置汇总

   1. 堆设置
      - **-Xms**:初始堆大小
      - **-Xmx**:最大堆大小
      - **-XX:NewSize=n**:设置年轻代大小
      - **-XX:NewRatio=n:**设置年轻代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
      - **-XX:SurvivorRatio=n**:年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5
      - **-XX:MaxPermSize=n**:设置持久代大小
   2. 收集器设置
      - **-XX:+UseSerialGC**:设置串行收集器
      - **-XX:+UseParallelGC**:设置并行收集器
      - **-XX:+UseParalledlOldGC**:设置并行年老代收集器
      - **-XX:+UseConcMarkSweepGC**:设置并发收集器
   3. 垃圾回收统计信息
      - **-XX:+PrintGC**
      - **-XX:+PrintGCDetails**
      - **-XX:+PrintGCTimeStamps**
      - **-Xloggc:filename**
   4. 并行收集器设置
      - **-XX:ParallelGCThreads=n**:设置并行收集器收集时使用的CPU数。并行收集线程数。
      - **-XX:MaxGCPauseMillis=n**:设置并行收集最大暂停时间
      - **-XX:GCTimeRatio=n**:设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)
   5. 并发收集器设置
      - **-XX:+CMSIncrementalMode**:设置为增量模式。适用于单CPU情况。
      - **-XX:ParallelGCThreads=n**:设置并发收集器年轻代收集方式为并行收集时，使用的CPU数。并行收集线程数。


**四、调优总结**

1. 年轻代大小选择

   - **响应时间优先的应用**：**尽可能设大，直到接近系统的最低响应时间限制**（根据实际情况选择）。在此种情况下，年轻代收集发生的频率也是最小的。同时，减少到达年老代的对象。
   - **吞吐量优先的应用**：尽可能的设置大，可能到达Gbit的程度。因为对响应时间没有要求，垃圾收集可以并行进行，一般适合8CPU以上的应用。

2. 年老代大小选择

   - 响应时间优先的应用

     ：年老代使用并发收集器，所以其大小需要小心设置，一般要考虑

     并发会话率

     和

     会话持续时间

     等一些参数。如果堆设置小了，可以会造成内存碎片、高回收频率以及应用暂停而使用传统的标记清除方式；如果堆大了，则需要较长的收集时间。最优化的方案，一般需要参考以下数据获得：

     - 并发垃圾收集信息
     - 持久代并发收集次数
     - 传统GC信息
     - 花在年轻代和年老代回收上的时间比例

     减少年轻代和年老代花费的时间，一般会提高应用的效率

   - **吞吐量优先的应用**：一般吞吐量优先的应用都有一个很大的年轻代和一个较小的年老代。原因是，这样可以尽可能回收掉大部分短期对象，减少中期的对象，而年老代尽存放长期存活对象。

3. 较小堆引起的碎片问题

   因为年老代的并发收集器使用标记、清除算法，所以不会对堆进行压缩。当收集器回收时，他会把相邻的空间进行合并，这样可以分配给较大的对象。但是，当堆空间较小时，运行一段时间以后，就会出现“碎片”，如果并发收集器找不到足够的空间，那么并发收集器将会停止，然后使用传统的标记、清除方式进行回收。如果出现“碎片”，可能需要进行如下配置：

   - **-XX:+UseCMSCompactAtFullCollection**：使用并发收集器时，开启对年老代的压缩。
   - **-XX:CMSFullGCsBeforeCompaction=0**：上面配置开启的情况下，这里设置多少次Full GC后，对年老代进行压缩



#### 参考文章
[深入理解JVM（七）——性能监控工具](https://blog.csdn.net/qq3401247010/article/details/77778620)

 [JVM调优总结 -Xms -Xmx -Xmn -Xss](https://www.cnblogs.com/likehua/p/3369823.html)

