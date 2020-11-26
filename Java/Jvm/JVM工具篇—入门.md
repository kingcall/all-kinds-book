[toc]
# JVM 工具
## jdk 自带工具

工具名称 | 主要作用描述
---|---
javap | 反编译
jps| 列出素有java 虚拟机进程
jinfo | 显示虚拟机配置信息(java -XX:+PrintFlagsFinal 也可以查看)
jstat | 汇总统计信息(各种gc、编译、类加载的信息)
jmap | 堆的使用统计信息，类实例信息、生成内存快照
jhat | 主要用于分析生成的快照文件(一个页面，支持OQl语言查询)
jatck | 分析虚拟机的线程快照
jconsole | 可视化分分析工具，支持远程连接(可查看的内容比较有限)
jvisualvm | 可视化分分析工具，支持远程连接，支持加载堆文件进行分析

### javap
- jdk 提供的反编译工具
#### 选项参数
- p。默认情况下javap会打印所有非私有的字段和方法当加了 -p 选项后，它还将打印私有的字段和方法
- v 它尽可能地打印所有信息。如果你只需要查阅方法对应的字节码，那么可以用 -c 选项来替换 -v

### jps
- 输出当前机器的虚拟机(Hotspot)进程

#### 选项参数
- q 不输出类名、Jar名和传入main方法的参数
- m 输出传入main方法的参数
- l 输出main类或Jar的全限名
- v 输出传入JVM的参数

### jinfo
#### 查看系统的信息
#### 查看运行程序的启动参数
#### 改变当前正在运行的系统的一些参数(添加参数) 
##### 确定可变参数
- 可变的一定是可以改变的参数,可以通过下面的方式查看哪些是可变参数
```
java -XX:+PrintFlagsFinal -version | grep manageable
intx CMSAbortablePrecleanWaitMillis            = 100                                 {manageable}
intx CMSTriggerInterval                        = -1                                  {manageable}
intx CMSWaitDuration                           = 2000                                {manageable}
bool HeapDumpAfterFullGC                       = false                               {manageable}
bool HeapDumpBeforeFullGC                      = false                               {manageable}
bool HeapDumpOnOutOfMemoryError                = false                               {manageable}
cstr HeapDumpPath                              =                                    {manageable}
intx MaxHeapFreeRatio                          = 100                                {manageable}
intx MinHeapFreeRatio                          = 0                                  {manageable}
bool PrintClassHistogram                       = false                               {manageable}
bool PrintClassHistogramAfterFullGC            = false                               {manageable}
bool PrintClassHistogramBeforeFullGC           = false                               {manageable}
bool PrintConcurrentLocks                      = false                               {manageable}
bool PrintGC                                   = false                               {manageable}
bool PrintGCDateStamps                         = false                               {manageable}
bool PrintGCDetails                            = false                               {manageable}
bool PrintGCID                                 = false                               {manageable}
bool PrintGCTimeStamps                         = false                              {manageable}
```
##### 修改(添加)可变参数
```
jinfo -flag +HeapDumpAfterFullGC pid
```

```
Attaching to process ID 1580, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.181-b13
Java System Properties:

java.runtime.name = Java(TM) SE Runtime Environment
java.vm.version = 25.181-b13
sun.boot.library.path = D:\worksoft\java\jdk\jre\bin
java.vendor.url = http://java.oracle.com/
java.vm.vendor = Oracle Corporation
path.separator = ;
file.encoding.pkg = sun.io
java.vm.name = Java HotSpot(TM) 64-Bit Server VM
sun.os.patch.level =
sun.java.launcher = SUN_STANDARD
user.script =
user.country = CN
user.dir = D:\workspace\code\interview\algorithm
java.vm.specification.name = Java Virtual Machine Specification
java.runtime.version = 1.8.0_181-b13
java.awt.graphicsenv = sun.awt.Win32GraphicsEnvironment
os.arch = amd64
java.endorsed.dirs = D:\worksoft\java\jdk\jre\lib\endorsed
line.separator =

java.io.tmpdir = C:\Users\king\AppData\Local\Temp\
java.vm.specification.vendor = Oracle Corporation
user.variant =
os.name = Windows 10
sun.jnu.encoding = GBK
java.library.path = D:\worksoft\java\jdk\bin;C:\WINDOWS\Sun\Java\bin;C:\WINDOWS\system32;C:\WINDOWS;C:\Program Files (x86)\Common Files\Oracle\Java\javapath;C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\System32\WindowsPowerShell\v1.0\;C:\WINDOWS\System32\OpenSSH\;D:\worksoft\maven\bin;D:\worksoft\Git\cmd;D:\worksoft\mysql\bin;D:\worksoft\netcat;D:\worksoft\Redis;D:\worksoft\MongoDB\bin;D:\worksoft\nginx-1.15.5;D:\worksoft\zookeeper\bin;D:\worksoft\zeppelin-0.8.0\bin;D:\worksoft\nodejs\;C:\ProgramData\chocolatey\bin;D:\worksoft\zookeeperbin;D:\worksoft\java\jdk\bin;D:\worksoft\scala-2.12\bin;D:\worksoft\gradle-4.7\bin;D:\worksoft\hadoop\bin;D:\worksoft\spark-2.2.3\bin;D:\worksoft\flink-1.5.1\bin;D:\worksoft\protoc;D:\worksoft\Snipaste-1.16.2-x64;D:\worksoft\kafka_2.11\bin\windows;D:\worksoft\OpenVPN\bin;C:\Program Files\PowerShell\7-preview\preview;D:\worksoft\influxdb-1.7.7-1;D:\worksoft\Apache24\bin;D:\worksoft\Subversion\bin;D:\worksoft\Apache2.2\bin;D:\worksoft\Anaconda3;D:\worksoft\Anaconda3\Library\mingw-w64\bin;D:\worksoft\Anaconda3\Library\usr\bin;D:\worksoft\Anaconda3\Library\bin;D:\worksoft\Anaconda3\Scripts;C:\Users\king\AppData\Local\Microsoft\WindowsApps;C:\Users\king\AppData\Roaming\npm;D:\worksoft\Microsoft VS Code\bin;;.
java.specification.name = Java Platform API Specification
java.class.version = 52.0
sun.management.compiler = HotSpot 64-Bit Tiered Compilers
os.version = 10.0
user.home = C:\Users\king
user.timezone =
java.awt.printerjob = sun.awt.windows.WPrinterJob
file.encoding = UTF-8
java.specification.version = 1.8
user.name = king
java.class.path = D:\worksoft\java\jdk\jre\lib\charsets.jar;D:\worksoft\java\jdk\jre\lib\deploy.jar;D:\worksoft\java\jdk\jre\lib\ext\access-bridge-64.jar;D:\worksoft\java\jdk\jre\lib\ext\cldrdata.jar;D:\worksoft\java\jdk\jre\lib\ext\dnsns.jar;D:\worksoft\java\jdk\jre\lib\ext\jaccess.jar;D:\worksoft\java\jdk\jre\lib\ext\jfxrt.jar;D:\worksoft\java\jdk\jre\lib\ext\localedata.jar;D:\worksoft\java\jdk\jre\lib\ext\nashorn.jar;D:\worksoft\java\jdk\jre\lib\ext\sunec.jar;D:\worksoft\java\jdk\jre\lib\ext\sunjce_provider.jar;D:\worksoft\java\jdk\jre\lib\ext\sunmscapi.jar;D:\worksoft\java\jdk\jre\lib\ext\sunpkcs11.jar;D:\worksoft\java\jdk\jre\lib\ext\zipfs.jar;D:\worksoft\java\jdk\jre\lib\javaws.jar;D:\worksoft\java\jdk\jre\lib\jce.jar;D:\worksoft\java\jdk\jre\lib\jfr.jar;D:\worksoft\java\jdk\jre\lib\jfxswt.jar;D:\worksoft\java\jdk\jre\lib\jsse.jar;D:\worksoft\java\jdk\jre\lib\management-agent.jar;D:\worksoft\java\jdk\jre\lib\plugin.jar;D:\worksoft\java\jdk\jre\lib\resources.jar;D:\worksoft\java\jdk\jre\lib\rt.jar;D:\workspace\code\interview\algorithm\target\classes;D:\workspace\reposity\commons-io\commons-io\2.6\commons-io-2.6.jar;D:\workspace\reposity\org\apache\commons\commons-math3\3.6.1\commons-math3-3.6.1.jar;D:\workspace\reposity\org\apache\commons\commons-text\1.6\commons-text-1.6.jar;D:\workspace\reposity\org\apache\commons\commons-lang3\3.8.1\commons-lang3-3.8.1.jar;D:\workspace\reposity\org\apache\commons\commons-configuration2\2.4\commons-configuration2-2.4.jar;D:\workspace\reposity\commons-logging\commons-logging\1.2\commons-logging-1.2.jar;D:\workspace\reposity\com\google\guava\guava\27.0.1-jre\guava-27.0.1-jre.jar;D:\workspace\reposity\com\google\guava\failureaccess\1.0.1\failureaccess-1.0.1.jar;D:\workspace\reposity\com\google\guava\listenablefuture\9999.0-empty-to-avoid-conflict-with-guava\listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar;D:\workspace\reposity\com\google\code\findbugs\jsr305\3.0.2\jsr305-3.0.2.jar;D:\workspace\reposity\org\checkerframework\checker-qual\2.5.2\checker-qual-2.5.2.jar;D:\workspace\reposity\com\google\errorprone\error_prone_annotations\2.2.0\error_prone_annotations-2.2.0.jar;D:\workspace\reposity\com\google\j2objc\j2objc-annotations\1.1\j2objc-annotations-1.1.jar;D:\workspace\reposity\org\codehaus\mojo\animal-sniffer-annotations\1.17\animal-sniffer-annotations-1.17.jar;D:\workspace\reposity\junit\junit\4.12\junit-4.12.jar;D:\workspace\reposity\org\hamcrest\hamcrest-core\1.3\hamcrest-core-1.3.jar;D:\workspace\reposity\org\projectlombok\lombok\1.18.10\lombok-1.18.10.jar;D:\workspace\reposity\com\lmax\disruptor\3.4.2\disruptor-3.4.2.jar;D:\workspace\reposity\mysql\mysql-connector-java\8.0.18\mysql-connector-java-8.0.18.jar;D:\workspace\reposity\com\google\protobuf\protobuf-java\3.6.1\protobuf-java-3.6.1.jar;D:\worksoft\idea\lib\idea_rt.jar
java.vm.specification.version = 1.8
sun.arch.data.model = 64
sun.java.command = jvm.gc.mat.FullGCProblem
java.home = D:\worksoft\java\jdk\jre
user.language = zh
java.specification.vendor = Oracle Corporation
awt.toolkit = sun.awt.windows.WToolkit
java.vm.info = mixed mode
java.version = 1.8.0_181
java.ext.dirs = D:\worksoft\java\jdk\jre\lib\ext;C:\WINDOWS\Sun\Java\lib\ext
sun.boot.class.path = D:\worksoft\java\jdk\jre\lib\resources.jar;D:\worksoft\java\jdk\jre\lib\rt.jar;D:\worksoft\java\jdk\jre\lib\sunrsasign.jar;D:\worksoft\java\jdk\jre\lib\jsse.jar;D:\worksoft\java\jdk\jre\lib\jce.jar;D:\worksoft\java\jdk\jre\lib\charsets.jar;D:\worksoft\java\jdk\jre\lib\jfr.jar;D:\worksoft\java\jdk\jre\classes
java.vendor = Oracle Corporation
file.separator = \
java.vendor.url.bug = http://bugreport.sun.com/bugreport/
sun.io.unicode.encoding = UnicodeLittle
sun.cpu.endian = little
sun.desktop = windows
sun.cpu.isalist = amd64

VM Flags:
Non-default VM flags: -XX:CICompilerCount=3 -XX:InitialHeapSize=20971520 -XX:MaxHeapSize=20971520 -XX:MaxNewSize=6946816 -XX:MinHeapDeltaBytes=196608 -XX:NewSize=6946816 -XX:OldSize=14024704 -XX:+PrintGC -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:-UseLargePagesIndividualAllocation -XX:+UseSerialGC
Command line:  -verbose:gc -XX:+UseSerialGC -Xms20m -Xmx20m -XX:+PrintGCDetails -javaagent:D:\worksoft\idea\lib\idea_rt.jar=53463:D:\worksoft\idea\bin -Dfile.encoding=UTF-8
```
### jstat
#### 时间参数
- 默认只会打印一次，我们可以提供时间间隔参数和最大打印次数(如果没有一直打印知道程序停止)
- jstat -gc pid 1000(时间间隔) 10(最大打印次数)
- -t 选项可以打印出当前进程已经运行的时间，我们可以用gc 的总时间和这个时间的比例来判断系统gc的运行情况，超过20% 说明gc 压力比较大；同理我们可以获得老年代和年轻代的变化情况，判断系统是否存在内存泄漏。
#### 选项参数

 参数 | 含义
---|---
gc | gc 的统计信息，主要是各个代和区域的内存使用情况、gc 发生的次数和gc 所占用的时间
gccapacity| 和gc 参数基本相同，统计信息主要关注点是到当前堆区域使用的最值，以及gc 次数
gcutil |和gc 参数基本相同，输出的主要是空间的使用占比
gcnew | 新生代gc 情况
gcnewcapacity | 和 gcnew 基本一致，主要关注点是到当前新生代区域使用的最值，以及gc 次数
gcold | 同理
gcoldcapacity | 同理
gcmetacapacity | 永久代的最值
gccause | 和gcutil 一样，但是输出上次gc 的原因
class | 类加载、卸载、以及当前占用空间和加载耗时
compiler | 编译的情况，成功数、失败数、无效数、耗时
printcompilation | 被JIT 编译的方法


##### 具体参数
- gc (GC 的信息统计)
    - S0C：第一个幸存区的大小
    - S1C：第二个幸存区的大小
    - S0U：第一个幸存区的使用大小
    - S1U：第二个幸存区的使用大小
    - EC：伊甸园区的大小
    - EU：伊甸园区的使用大小
    - OC：老年代大小
    - OU：老年代使用大小
    - MC：方法区大小
    - MU：方法区使用大小
    - CCSC:压缩类空间大小
    - CCSU:压缩类空间使用大小
    - YGC：年轻代垃圾回收次数
    - YGCT：年轻代垃圾回收消耗时间
    - FGC：老年代垃圾回收次数
    - FGCT：老年代垃圾回收消耗时间
    - GCT：垃圾回收消耗总时间
- class (类加载的信息统计)
- compiler(编译情况统计)
- gccapacity(堆内存信息统计)
    - NGCMN：新生代最小容量
    - NGCMX：新生代最大容量
    - NGC：当前新生代容量
    - S0C：第一个幸存区大小
    - S1C：第二个幸存区的大小
    - EC：伊甸园区的大小
    - OGCMN：老年代最小容量
    - OGCMX：老年代最大容量
    - OGC：当前老年代大小
    - OC:当前老年代大小
    - MCMN:最小元数据容量
    - MCMX：最大元数据容量
    - MC：当前元数据空间大小
    - CCSMN：最小压缩类空间大小
    - CCSMX：最大压缩类空间大小
    - CCSC：当前压缩类空间大小
    - YGC：年轻代gc次数
    - FGC：老年代GC次数
- gcnew(新生代垃圾统计)
    - NGCMN：新生代最小容量
    - NGCMX：新生代最大容量
    - NGC：当前新生代容量
    - S0CMX：最大幸存1区大小
    - S0C：当前幸存1区大小
    - S1CMX：最大幸存2区大小
    - S1C：当前幸存2区大小
    - ECMX：最大伊甸园区大小
    - EC：当前伊甸园区大小
    - YGC：年轻代垃圾回收次数
    - FGC：老年代回收次数
- gcold(老年代内存统计)
    - MC：方法区大小
    - MU：方法区使用大小
    - CCSC:压缩类空间大小
    - CCSU:压缩类空间使用大小
    - OC：老年代大小
    - OU：老年代使用大小
    - YGC：年轻代垃圾回收次数
    - FGC：老年代垃圾回收次数
    - FGCT：老年代垃圾回收消耗时间
    - GCT：垃圾回收消耗总时间
- gcmetacapacity(元数据空间统计)
    - MCMN: 最小元数据容量
    - MCMX：最大元数据容量
    - MC：当前元数据空间大小
    - CCSMN：最小压缩类空间大小
    - CCSMX：最大压缩类空间大小
    - CCSC：当前压缩类空间大小
    - YGC：年轻代垃圾回收次数
    - FGC：老年代垃圾回收次数
    - FGCT：老年代垃圾回收消耗时间
    - GCT：垃圾回收消耗总时间
- gcutil
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

### jmap(Memory Map)
- 主要用来输出统计信息、生成堆快照文件、其他信息
    - 基本的设置参数
    - 垃圾收集器参数
    - 各个区的使用情况，各代的使用情况

#### 可以拿到快照的几种方式
- jmap
- -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./(内存溢出的时候生成快照文件)

#### 选项
- histo(类的统计报表)
    - (统计信息，哪个类产生了多少个实例，占了多少个字节)
    - [:live] 如果带上live则只统计活对
- heap(堆的使用情况统计)
    - heap:format=b  产生一个堆内存文件(这个方式是比较老的一种方式了)
- dump:live,format=b,file=xxx 2657
- clstats
- F 强制执行

![image](https://note.youdao.com/yws/res/16876/98A5585EDFE34A45B1B40769C400679F)

### jstack(Java进程内的线程堆栈信息)
- -l 除了堆栈外，输出锁持有情况
- -m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息
- -F 当正常请求不被响应的时候，强制输出。
```
 Map<Thread, StackTraceElement[]> Thread.getAllStackTraces(),可以获取虚拟机中所有的线程的StackTraceElement对象， 可以用来做监控
```

### jcmd
- jcmd 可以看做是以上工具的一个综合(除了jstat)——工具集合
- 查看帮助 jcmd 10478 help
```
The following commands are available:
JFR.stop
JFR.start
JFR.dump
JFR.check
VM.native_memory
VM.check_commercial_features
VM.unlock_commercial_features
ManagementAgent.stop
ManagementAgent.start_local
ManagementAgent.start
GC.rotate_log
Thread.print
GC.class_stats
GC.class_histogram
GC.heap_dump
GC.run_finalization
GC.run
VM.uptime
VM.flags
VM.system_properties
VM.command_line
VM.version
help
```
- jcmd 10478 Thread.print


### jconsole
- 主要监控内存、线程(可以检测死锁，当前有哪些线程)、类、mbean

### jvisualvm(多合一故障处理工具)
- 它的特点就是多合一， All-in-One

![](https://soaringbirdblog.oss-cn-shanghai.aliyuncs.com/oneblog/20191201205628421.png)
#### 特点
- 除了监控还提供了其他的功能，例如性能分析
- 比起专业的收费工具 JProfiler、YourKit 等不会逊色多少，而且不要求程序运行在特殊的Agent 环境中，因此它对程序的性能影响很小，可以直接在生产环境中使用。
- 通过插件可以扩展功能，插件可以采取离线安装的方式也可以选取在线安装的方式，推荐在线安装，点击窗口上的工具按钮，即可看待插件按钮，然后点击，选择你需要的插件，离线方式点击分析页面的起始页，然后点击图中的扩展链接，即可调到对应的插件下载页面，下载你所需要的即可

![](https://soaringbirdblog.oss-cn-shanghai.aliyuncs.com/oneblog/20191201205820467.png)

#### 插件
- 插件安装完成后，重启jvisualvm，选择一个要分析的程序，即可看到新的分析工具，在这里我添加了一个 Visual GC 插件，补充说明一下，这个插件看gc 的一些细节，怎一个爽字了得。

![](https://soaringbirdblog.oss-cn-shanghai.aliyuncs.com/oneblog/20191201210308403.png)
- jvisualvm 有idea 版本的插件，有需要的课自行安装。
#### 功能
- 基本的监控(jconsole的功能)
- 执行垃圾回收和dump 功能(jmap)
- jvm的默认参数和自定义参数(jps、jinfo)
- 栈分析(栈快照,jstack)
- 内存抽样

#### 远程连接
##### 无需认证的链接
```
-Dcom.sun.management.jmxremote.port=60001 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false 
-Djava.rmi.server.hostname=192.168.199.134
```
##### 需要认证的链接
```
-Dcom.sun.management.jmxremote.port=60001 
-Dcom.sun.management.jmxremote.authenticate=true 
-Dcom.sun.management.jmxremote.ssl=false 
-Dcom.sun.management.jmxremote.pwd.file= {jmxremote.password}Path 
-Djava.rmi.server.hostname=192.168.1.50  
在/usr/lib/jvm/java-8-oracle/jre/lib/management下的jmxremote.access、jmxremote.password。
jmxremote.access中存储用户名和对应的权限关系。
jmxremote.password中存储用户名密码。
注：在jmxremote.password和jmxremote.access的权限为600，所有者rw权限。

```
#### 堆分析
- 主要是可以分析从其他途径来的堆文件,文件——>载入

### jhat(Java Heap Analysis Tool)
-  jhat是用来分析java堆的命令，可以将堆中的对象以html的形式显示出来，包括对象的数量，大小等等，并支持对象查询语言
-  主要和jmap 配合使用

#### Other Queries
- Execute Object Query Language (OQL) query

## 第三方工具
### mat

### arthas
- 在线排查工具
- 有时候修改代码需要层层审批，会大大影响问题排查的进度



## 例子
- ps -Lfp 611
- top -Hp  6118
-  printf "%x" 6155