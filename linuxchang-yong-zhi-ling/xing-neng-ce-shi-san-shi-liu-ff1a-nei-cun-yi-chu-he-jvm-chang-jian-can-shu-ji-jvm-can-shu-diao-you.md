[https://www.cnblogs.com/zhongyehai/p/10306827.html](https://www.cnblogs.com/zhongyehai/p/10306827.html)

堆内存溢出：

此种溢出，加内存只能缓解问题，不能根除问题，需优化代码  
堆内存中存在大量对象，这些对象都有被引用，当所有对象占用空间达到堆内存的最大值，就会出现内存溢出OutOfMemory:Java heap space

永久代溢出

如果发生，则是在初始化的时候，空间太小，解决办法，扩大空间  
类的一些信息，如类名、访问修饰符、字段描述、方法描述等，所占空间大于永久代最大值，就会出现OutOfMemoryError:PermGen space

内存溢出的检测方法：pid=1730

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124215618411-2049105386.png)

Jdk/bin目录下有很多检测工具  
图形界面：  
　　Jconsole  
　　Jvisualvm  
命令行工具：

Jstat –gcutil pid 1000 100 （只需要看O，如果达到100%，并且长期处于100%，则代表老年代内存不足）

pid：进程号、1000：1秒钟获取一次、100一共获取100次

E：eden区  
　　　　O：老年代  
　　　　P：永久代  
　　　　YGC：新生代的GC次数  
　　　　YGCT：当前统计的YGC一共花费的时间（毫秒）  
　　　　FGC：fullGC老年代的GC次数  
　　　　FGCT：当前统计的FGC一共花费的时间（毫秒）  
　　　　GCT：YGC+FGC

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124220555599-1252550293.png)

Jmap –histo pid \| head -20  
　　把当前JVM里面的所有对象打印出来，排序，head：头部，head -20 前20行

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124230307205-1367180003.png)

Jmap –heap pid（列出当前进程的堆的数据，一般用来看当前配置）

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124221541031-1790056843.png)

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124221645843-921239862.png)

在jvisualvm上面也可以看到JVM参数

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124222420394-1306894805.png)

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124222435318-1904063380.png)

FullGC频率：单次FullGC时间&lt;200ms

Jvm常见参数  
-Xms2048m，初始堆大小，建议&lt;物理内存的1/4，默认值为物理内存的1/64  
-Xmx2048m，最大堆大小，建议与-Xms保持一致，默认值为物理内存的1/4  
-Xmn512m，新生代大小，建议不超过堆内存的1/2  
-Xss256k,线程年轻代堆栈大小，建议256k  
-XX:PermSize=256m，永久代初始值，默认值为物理内存的1/64  
-XX:MaxPermSize=256m，永久代最大值，默认值为物理内存的1/4  
-XX:SurvivorRatio=8：年轻带中Eden区和Survivor区的比例，默认为8:1:1，即Eden（8），From Space（1），ToSpace（1）  
-XX:+UseConcMarkSweepGC：开启CMS垃圾回收器

看一下内存溢出的情况

先配置一下tomcat里面JVM的参数：vi /home/server/tomcat-PerfTeach01/bin/catalina.sh

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124213340328-68926197.png)

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124214804531-1825556553.png)

由于ip变了，改一下host里面的ip

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124214344795-88365146.png)

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124214317422-1711599186.png)

启动tomcat

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124213122166-1879349732.png)

能访问，代表启动成功：[http://localhost:8080/PerfTeach/MemoryLeak?userId=123&password=abc&waitTime=3](http://localhost:8080/PerfTeach/MemoryLeak?userId=123&password=abc&waitTime=3)

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124215117494-2083198097.png)

jmeter5个并发永远跑

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124223240832-1851751901.png)

TPS

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124224000072-1259274399.png)

响应时间

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124224133213-136940425.png)

cpu

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124224234416-1308748182.png)

jstat -gcutil 1730 1000 1000

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124225342511-1984261880.png)

jvisualvm

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124224956696-776479547.png)

看一下tomcat日志的后200行

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124224629319-1712119131.png)

有内存溢出的报错

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124224713425-802077873.png)

看当前JVM里面的所有对象，找com开头的，业务代码，再找非java开头的

可以看出org.apache可能有问题，com.lee是业务代码，一定有问题

直接告诉开发，让开发去解决![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124230904903-1383340107.png)

jvisualvm也可以生成快照，一开始出现内存泄漏的时候就点堆Dump在服务器下生成快照，下载后再用 jvisualvm打开， jvisualvm\_文件\_装入\_文件类型选“线程 Dump”

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124232152654-1762957857.png)

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190124232237072-1943056804.png)

由于这个内存泄漏的原因其实是session引起的，所以最好把tomcat的session持久化取消掉，以免下次启动tomcat又加载上一次保留的session

修改：tomcat目录下conf/context.xml，将 &lt;Manager pathname="" /&gt;这行的注释打开

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190125204507804-283225403.png)

把注释打开

![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190125204626495-320264402.png)![](https://img2018.cnblogs.com/blog/1406024/201901/1406024-20190125204653505-941748346.png)

内存泄漏的本质：老年代空间里面东西放满了，又不能被回收

程序一旦出现内存泄漏，就算停止压测也不能解决，只能重启

内存泄露会出现的现象  
1，tps出现大幅波动，并慢慢降低，甚至降为0，响应时间随之波动，慢慢升高  
2，通过jstat命令看到，Jvm中Old区不断增加，FullGC非常频繁，对应的FullGC消耗的时间也不断增加  
3，通过jconsole/jvisualvm可以看到，堆内存曲线不断上升，接近上限时，变成一条直线  
4，日志报错java.lang.OutOfMemoryError: Java heap space

内存泄露定位  
1，通过jmap命令：jmap -histo pid \| head -20，查看当前堆内存中实例数和占用内存最多的前20个对象  
2，通过jvisualvm，进行远程堆dump，然后把dump文件下载下来，用jvisualvm打开进行分析，可以看到更直观的jvm中对象的信息

监控内存泄露问题的场景  
1，在试压阶段，或任意场景都可以考虑通过jvisualvm和jstat监控jvm的情况  
2，在稳定性场景中，一定要关注Jvm内存使用的情况，在长时间的压测下，最容易看出内存泄露的问题

JVM参数调优

## Jvm常用参数

堆内存 = 年轻代+老年代  
年轻代 = Eden+Survivor

## Survivor = From Space+To Space

年轻代 = Eden+From Space+To Space

# 堆内存=Eden+From Space+To Space+老年代

-Xms2048m，初始堆大小，建议&lt;物理内存的1/4，默认值为物理内存的1/64  
-Xmx2048m，最大堆大小，建议与-Xms保持一致，默认值为物理内存的1/4  
-Xmn512m，新生代大小，建议不超过堆内存的1/2  
-Xss256k,线程堆栈大小，建议256k  
-XX:PermSize=256m，永久代初始值，默认值为物理内存的1/64  
-XX:MaxPermSize=256m，永久代最大值，默认值为物理内存的1/4  
-XX:SurvivorRatio=8：年轻带中Eden区和Survivor区的比例，默认为8:1，即Eden（8），From Space（1），ToSpace（1）  
-XX:MaxTenuringThreshold=15：晋升到老年代的对象年龄，每个对象坚持过一次MinorGC后对象年龄+1，默认值是15，年龄超过15进入到老年代，该参数在串行GC时有效  
-XX:PretenureSizeThreshold=3145728：单位字节，只对Serial和ParNew两款收集器有效，新生代采用Parallel Scavenge GC时无效，大于这个值的对象直接在老年代进行分配

非稳定参数，使用方式主要有以下三种：  
1，-XX +&lt;option&gt;：开启option参数  
2，-XX -&lt;option&gt;：关闭option参数  
3，-XX &lt;option&gt;=&lt;value&gt;：将option参数的值设置为value

client模式，Jvm默认垃圾收集器  
UseSerialGC：新生代采用Serial收集器，老年代采用Serial Old收集器

server模式，Jvm默认垃圾回收期  
UseParallelGc：新生代采用Parallel Scavenge收集器，吞吐量优先的收集器，老年代采用Serial Old收集器

-XX:+UseConcMarkSweepGC：默认关闭，ParNew+CMS+Serial Old，当CMS收集器出现ConcurrentModeFailure错误（Jvm预留空间不足以容纳程序使用），采用后备收集器Serial Old  
CMS相关参数  
-XX:CMSInitiatingOccupancyFraction=80：CMS收集器在老年代空间被使用多少时触发FullGC，默认为92，建议80左右  
-XX:+UseCMSCompactAtFullCollection：CMS收集器在FullGC时开启内存碎片的压缩，默认关闭  
-XX:CMSFullGCsBeforeCompaction=8：执行多少次不压缩FullGC后，进行一次压缩，默认是0（代表每次FullGC都进行压缩）  
-XX:+UseCMSInitiatingOccupancyOnly：使用手动定义初始化定义开始，禁止hostspot自行触发CMS GC  
-XX:ParallelGCThreads=8：并行收集器的线程数，此值最好配置与处理器数目相等 同样适用于CMS

日志参数：  
-XX:+HeapDumpOnOutOfMemoryError：当发生内存溢出时，进行堆内存dump，一般把这个打开

# -XX:+PrintGCDetails：打印GC的详细信息

企业实际配置  
-server -Xms1028m -Xmx1028m -XX:PermSize=256m -XX:MaxPermSize=256m -Xmn512m -XX:MaxDirectMemorySize=1g -XX:SurvivorRatio=10  
-XX:+UseConcMarkSweepGC  
-XX:+UseCMSCompactAtFullCollection  
-XX:CMSMaxAbortablePrecleanTime=5000  
-XX:+CMSClassUnloadingEnabled  
-XX:CMSInitiatingOccupancyFraction=80  
-XX:+UseCMSInitiatingOccupancyOnly  
-XX:ParallelGCThreads=8  
-Xloggc:/home/admin/logs/gc.log  
-XX:+PrintGCDetails  
-XX:+PrintGCDateStamps  
-XX:+HeapDumpOnOutOfMemoryError  
-XX:HeapDumpPath=/home/admin/logs/java.hprof

