[toc]
## 参数基础
### 查看参数
-  "-XX:+PrintVMOptions"：打印虚拟机接受到的参数（显式参数
-  "-XX:+PrintCommandLineFlags"：打印虚拟机显式和隐式参数
-  jinfo pid(启动之后查看参数的方式)

### 参数的写法

1. "-X" 开头的，比如-Xmx100M
2. "-XX:" 开头的，比如-XX:+UseG1GC
3. "-"" 开头的，比如-verbose:gc

> 其中-X和-开头的通常会被转换为一个或者多个-XX:开头的参数，只是一个简化的写法，比如说-Xmx100M，JVM里会自动转化为-XX:MaxHeapSize=100M，-verbose:class会自动转换为-XX:+TraceClassLoading -XX:+TraceClassUnloading

## 跟踪垃圾回收
- "-XX:+PrintGC",GC时打印简单日志。
```
[GC (Allocation Failure)  42000K->36496K(56320K), 0.0009016 secs]
[GC (Allocation Failure) -- 41616K->41616K(56320K), 0.0020593 secs]
[Full GC (Ergonomics)  41616K->5655K(56320K), 0.0070610 secs]
```
- "-XX:+PrintGCDetails",GC时打印详细日,志虚拟机在退出时，打印堆的详细信息
- "-XX:+PrintHeapAtGC",GC时打印前后日志。
- "-XX:+PrintGCTimeStamps",打印虚拟机启动后GC发生的时间偏移量
- "-XX:+PrintGCApplicationStoppedTime",GC时打印停顿时间
- "-XX:+PrintReferenceGC",打印软运用、弱引用、虚引用和Finallize队列
- "Xloggc"：GC日志输出到文件
```
-Xloggc:E:/gc.log
```

## 堆
- JVM在启动之后，整个堆大小虽然是固定的，但是**并不代表整个堆里的内存都可用**，在GC之后会根据一些参数进行动态的调整，比如我们设置Xmx和Xms不一样的时候，就表示堆里的新生代和老生代的可用内存都是存在不断变化的

![image](https://note.youdao.com/yws/res/37812/A6CC065DF2984156A4A94EB52574B4BB)
- 可以看出虽然设置了Xmx,但是堆启动的时候只启动了256M,这个时候NewSize+OldSize=InitialHeapSize的

### 初始堆大小
1. "-Xms"：堆初始大小。
2. "-Xmx"：堆最大大小。

> 默认Xms是256M，如果Xmx比256大，则Xms的默认值就是256，但是如果Xmx小于256则Xms=Xmx

> -XX:InitialHeapSize=128m -XX:MaxHeapSize=2g 和上面参数的效果一样

>  Xmx 必须比Xms大，否则进程无法启动，因为jvm 会判断这两个参数的大小，java 的初始堆大小也必须大于1M,否则也会因为参数错误导致进程无法启动

#### 技巧
> 技巧：实际工作中，设置初始堆和最大堆相等，可以减少垃圾回收次数，提供性能,其实这个可以反映出其实jvm 的堆大小是在动态变化的，变化范围是在 Xms 和 Xmx 之间

### 堆分布
#### 大小设置
1. "-Xmn"：新生代大小,设置新生代的初始值及最大值,同时影响老年代大小
> Xmn参数等价于**同时设置了NewSize和MaxNewSize，并且值都相等**，例如-Xmn128M，等同于-XX:NewSize=128M -XX:MaxNewSize=128M

2. "-XX:NewSize=512m" 设置新生代的初始值
> 1NewSize是设置新生代有效内存的初始化大小，也可以说是新生代有效内存的最小值，当新生代回收之后有效内存可能会进行缩容，这个参数就指定了能缩小到的最小值

3. "-XX:MaxNewSize=512m" 设置新生代的最大值
> ，就是设置新生代有效内存的最大值，当对新生代进行回收之后可能会对新生代的有效内存进行扩容，那到底能扩容到多大，这就是最大值

#### 比例设置
##### 新生代大小比例设置
1. "-XX:NewRatio"：新生代和老年代比例，即老年代/新生代，与"-Xmn"作用相同（旧参数）,和Xmn的意义一样,只不过这里是设置比例的方式
> G1 GC下不建议设置新生代这些参数，尽量自适应，GC效率会更高，这也是官方推荐的

> 设置老年代和年轻代的比例。比如：-XX:NewRatio=8 表示老年代内存:年轻代内存=8:1 => 老年代占堆内存的8/9;年轻代占堆内存的1/9

> 默认值：堆内存的1/4（这里要记住不是最大堆内存，还是已经分配的堆内存的1/4）

##### 设置幸存区的比例
2. "-XX:SurvivorRatio"：设置新生代和存活区的比例(这里需要注意的是存活区指的是**其中一个**),eden区和from/to区的比例，即eden/from和eden/to。
> -XX:SurvivorRatio=8表示新生代:存活区=8:1=>新生代占年轻代的8/10,每个存活区各占年轻代的1/10
- InitialSurvivorRatio和MinSurvivorRatio只针对PS GC算法有用
- 如果我们JVM参数里设置过SurvivorRatio，但是没有设置InitialSurvivorRatio，那么InitialSurvivorRatio的值会被默认设置为SurvivorRatio+2，MinSurvivorRatio在这种情况下也是一样的

> CMS GC下如果MaxTenuringThreshold设置为0，相当于每次GC都直接晋升到老生代，此时如果SurvivorRatio没有设置的话，会将SurvivorRatio默认设置为1024


#### 堆的动态变化
> MinHeapFreeRatio和MaxHeapFreeRatio在不同的GC算法里，它作用的范围不一样，比如G1作用于整个Heap，而SerialGC/Paralle GC/CMS GC作用在老生代

> 默认情况下MinHeapFreeRatio=40，MaxHeapFreeRatio=70，如果是PS GC，在默认开启自适应模式情况下，MinHeapFreeRatio=0，MaxHeapFreeRatio=100

2. Xminf和MinHeapFreeRatio是等价的，如-Xminf0.4等价于-XX:MinHeapFreeRatio=40,Xmaxf和MaxHeapFreeRatio是等价的，如-Xmaxf0.7等价于-XX:MaxHeapFreeRatio=70

3. "-XX:MinHeapFreeRatio=40",GC后，如果发现空闲堆内存占到整个堆内存的40%，则增大堆的大小。
4. "-XX:MaxHeapFreeRatio=70",：GC后，如果发现空闲堆内存占到整个堆内存的70%，则收缩堆得大小
> 如果HeapFreeRatio >MaxHeapFreeRatio，则需要进行堆缩容，缩容的时机应该在每次垃圾回收之后。

5. MinHeapDeltaBytes,表示当我们要扩容或者缩容的时候，决定是否要做或者尝试扩容的时候最小扩多少，默认为192K


#### 技巧
>  新生代大小对系统性能及GC有很大影响。实际工作中，应根据系统特点合理设置堆分布，基本策略是：**尽可能将对象预留在新生代，减少老年代GC的次数**。一般，新生代设置成整个堆的1/4~1/3左右

#### 例子
- 下面两个设置是等效的
```
-Xms60m -Xmx60m -Xmn20m -XX:SurvivorRatio=2 -XX:+PrintGCDetails
-Xms60m -Xmx60m -XX:NewRatio=2 -XX:SurvivorRatio=2 -XX:+PrintGCDetails
```
![image](https://note.youdao.com/yws/res/37811/273C4DAA5B6E4D1CA913C5E1D20D683A)

### 堆溢出
1. "-XX:+HeapDumpOnOutOfMemoryError"：堆溢出（OOM）时导出信息。
2. "-XX:HeapDumpPath"：堆溢出时导出的路径。使用MAT等工具可分析文件。
3. "-XX:OnOutOfMemoryError"：堆溢出时执行脚本，可用于奔溃程序自救、报警、通知等。

#### 例子
```
-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=D:/test.dump -XX:OnOutOfMemoryError=D:/shell.bat
```

## 跟踪类加载/卸载
1. "-verbose:class"：打印类加载和卸载。
2. "-XX:+TraceClassLoading"：打印类加载。
3. "-XX:+TraceClassUnloading"：打印类卸载。
>类存在形式：
    a) 一般，类以jar打包或class文件形式存放在文件系统。
    b)ASM等在运行时动态生成类。

## 方法区(MetaspaceSize)
1. "-XX:PermSize"：永久区初始大小（JDK 1.6/1.7）。
2. "-XX:MaxPermSize"：永久区最大大小（JDK 1.6/1.7）。
3. "-XX:MetaspaceSize" 元数据区初始大小

> 这两个参数是控制Metaspace触发GC的,并不是设置元数据区大小的

> 初始元空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值

4. "-XX:MaxMetaspaceSize"：元数据区最大大小（JDK 1.8）、
> 设置元空间的最大值，默认是没有上限的，也就是说你的系统内存上限是多少它就是多少
5. "CompressedClassSpaceSize",从JDK8开始，JVM启动的时候会专门分配一块内存，大小是CompressedClassSpaceSize，正常情况会类似Perm一样挨着Heap分配，这块内存专门来存类元数据的klass部分
> 如果UseCompressedClassPointers未开启，CompressedClassSpaceSize参数没有效果，那块内存也不会有
6. "InitialBootClassLoaderMetaspaceSize"主要指定BootClassLoader的存储非klass部分的数据的第一个Metachunk的大小，64位下默认4M，32位下默认2200K，而存储klass部分的数据的第一个Metachunk的大小默认大概是384K


### 技巧
1. 动态生成大量类（如动态代理、AOP）时，可能导致方法区溢出（OOM）
2. . JDK 1.6/1.7，默认永久区最大大小为64MB；JDK
3. 1.8，默认元数据区耗尽所有可用系统内存。

## 栈内存
1. "-Xss"：单个线程的栈最大大小。
> Xss和ThreadStackSize是等价的(-Xss100K等价于-XX:ThreadStackSize=100)，主要是设置Java线程栈的大小，单位是KB，真正生效的值会按照pageSize对齐(并不修改参数的值)，而ThreadStackSize的值64位os下默认是1M，32位下默认位512K，Linux下线程栈的最小值是228k


> "-Xss"影响函数调用深度、局部变量大小等。栈上分配速度快，同时避免垃圾回收，但栈相比堆较小，不适合大对象。

2. "-XX:+DoEscapeAnalysis"：开启逃逸分析（仅限Server模式下使用）。
3. "-XX:+EliminateAllocations"：开启标量替换（默认已开启，允许对象打散分配在栈上）
4. "-XX:CompilerThreadStackSize",置编译线程栈的大小，64位os下默认大小为4M，32位下默认位2M，比如C2 CompilerThread等线程
5. "-XX:VMThreadStackSize",设置JVM里的那些线程的线程栈大小，VMThreadStackSize的默认值和ThreadStackSize的默认值是一样的，比如VM Thread/GC task thread/VM Periodic Task Thread等


## 直接内存
- "-XX:MaxDirectMemorySize"：直接内存最大大小。如不设置，默认为"-Xmx"
> 直接内存适合申请次数较少、访问较频繁的场景。因为申请堆空间的速度远远高于直接内存。

## 工作模式
1. "-client"：Client模式
2. "-server"：Server模式
3. "-version"：查看模式

### 区别
- Client模式：启动速度较快。适合用户界面，运行时间不长。
- Server模式：启动速度较慢（启动时尝试收集更多系统性能信息，使用更复杂的优化算法优化程序），完全启动并稳定后，执行速度远远快于Client模式。适合后台长期运行的系统。
- 两种模式下的各种参数默认值可能不同，可使用“-XX:+PrintFlagsFinal”参数查看。