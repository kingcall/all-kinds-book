[toc]
## 背景
- JDK 1.4中的java.nio.*包中引入新的JavaI/O库，其目的是提高速度。实际上，“旧”的I/O包已经使用NIO重新实现过，即使我们不显式的使用NIO编程，也能从中受益。
> nio翻译成 no-blocking io 或者 new io 都无所谓啦，
- 在NIO中并不是以流的方式来处理数据的，而是以buffer缓冲区和Channel管道配合使用来处理数据。
- NIO提供了一套异步IO解决方案，其目的在于使用单线程来监控多路异步IO，使得IO效率，尤其是服务器端的IO效率得到大幅提高。为了实现这一套异步IO解决方案，NIO引入了三个概念，即缓冲区（Buffer）、通道（Channel）和选择器（Selector）
- NIO采用的是一种多路复用的机制，利用单线程轮询事件，高效定位就绪的Channel来决定做什么，只是Select阶段是阻塞式的，能有效避免大量连接数时，频繁线程的切换带来的性能或各种问题



## 对比IO
![image-20201126174640630](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:46:41-image-20201126174640630.png)
- IO是面向流的处理，NIO是面向块(缓冲区)的处理，面向流的I/O系统一次一个字节地处理数据，面向块(缓冲区)的I/O系统以块的形式处理数据
- 通常使用NIO是在网络中使用的，网上大部分讨论NIO都是在网络通信的基础之上的！**说NIO是非阻塞的NIO也是网络中体现的**，在网络中使用NIO往往是I/O模型的多路复用模型
- Channel和传统IO中的Stream很相似。虽然很相似，但是有很大的区别，主要区别为：通道是双向的，通过一个Channel既可以进行读，也可以进行写；而Stream只能进行单向操作，通过一个Stream只能进行读或者写；

![image-20201126174653785](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:46:54-image-20201126174653785.png)

## 三个核心部分组成
- 相对于传统IO而言，流是单向的。对于NIO而言，有了Channel管道这个概念，我们的读写都是双向的
> (铁路上的火车能从广州去北京、自然就能从北京返还到广州)！

> Channel管道比作成铁路，buffer缓冲区比作成火车(运载着货物),NIO就是通过Channel管道运输着存储数据的Buffer缓冲区的来实现数据的处理

> Channel不与数据打交道，它只负责运输数据。与数据打交道的是Buffer缓冲区,Channel-->运输,Buffer-->数据

### buffer缓冲区
- Buffer是缓冲区的抽象类,其中ByteBuffer是用得最多的实现类(在管道中读写字节数据)。
- 非直接缓冲区是需要经过一个：copy的阶段的(从内核空间copy到用户空间)，直接缓冲区不需要经过copy阶段，也可以理解成--->内存映射文件
- 缓冲区是对Java原生数组的对象封装，它除了包含其数组外，还带有四个描述缓冲区特征的属性以及一组用来操作缓冲区的API。
- 缓冲区的根类是Buffer，其重要的子类包括ByteBuffer、MappedByteBuffer、CharBuffer、IntBuffer、DoubleBuffer、ShortBuffer、LongBuffer、FloatBuffer。从其名称可以看出这些类分别对应了**存储不同类型数据的缓冲区**

![image-20201126174708705](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:47:09-image-20201126174708705.png)

#### 常见的buffer
#####  ByteBuffer
- ByteBuffer中直接存储字节，所以在不同的操作系统、硬件平台、文件系统和JDK之间传递数据时不涉及编码、解码和乱码问题，也不涉及Big-Endian和Little-Endian大小端问题，所以它是使用最为便利的一种缓冲区
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer
- StringCharBuffer
- FileChannel
- SocketChanel
- ServerSocketChannel
- DatagramChannel
##### 视图缓冲区
- ByteBuffer中存储的是字节，有时为了方便，可以使用asCharBuffer()等方法将ByteBuffer转换为存储某基本类型的视图，例如CharBuffer、IntBuffer、DoubleBuffer、ShortBuffer、LongBuffer和FloatBuffer。 
- 转换后，这两个缓冲区共享同一个内部数组，但是对数组内元素的视角不同。以CharBuffer和ByteBuffer为例，ByteBuffer将其视为一个个的字节（1个字节），而CharBuffer则将其视为一个个的字符（2个字节）。若此ByteBuffer的capacity为12，则对应的CharBuffer的capacity为12/2=6

#### buffer 的操作和属性
- Buffer类维护了4个核心变量属性来提供关于其所包含的数组的信息

##### 容量Capacity
缓冲区能够容纳的数据元素的最大数量。容量在缓冲区创建时被设定，并且永远不能被改变。(不能被改变的原因也很简单，底层是数组嘛)

##### 上界Limit
- 缓冲区里的数据的总数，代表了当前缓冲区中一共有多少数据。

##### 位置Position
- 下一个要被读或写的元素的位置。Position会自动由相应的 get( )和 put( )函数更新。

##### 标记Mark
- 一个备忘位置。用于记录上一次读写的位置
- 调用mark时mark=positon，调用reset时position=mark。 

#### 切换成读模式
- NIO给了我们一个flip()方法。这个方法可以改动position和limit的位置，limit变成了position的位置了，而position变成了0
- 般我们称filp()为“切换成读模式”

![image-20201126174735739](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:47:36-image-20201126174735739.png)


#### remaining 和  hasRemaining
- remaining()会返回缓冲区中目前存储的元素个数，在使用参数为数组的get方法中，提前知道缓冲区存储的元素个数是非常有用的。 
- 事实上，由于缓冲区的读或者写模式并不清晰，因此实际上remaining()返回的仅仅是limit – position的值。 
- 而hasRemaining()的含义是查询缓冲区中是否还有元素，这个方法的好处是它是线程安全的

#### compact
- 压缩compact()方法是为了将读取了一部分的buffer，其剩下的部分整体挪动到buffer的头部（即从0开始的一段位置），便于后续的写入或者读取。其含义为limit=limit-position，position=0

#### duplicate
- 复制缓冲区，两个缓冲区对象实际上指向了同一个内部数组，但分别管理各自的属性。

#### slice缓冲区切片
- 缓冲区切片，将一个大缓冲区的一部分切出来，作为一个单独的缓冲区，但是它们公用同一个内部数组。
- 切片从原缓冲区的position位置开始，至limit为止。原缓冲区和切片各自拥有自己的属性

#### 使用直接缓冲区有两种方式
- 缓冲区创建的时候分配的是直接缓冲区
- 在FileChannel上调用map()方法，将文件直接映射到内存中创建

### Channel管道
- Channel通道只负责传输数据、不直接操作数据的。操作数据都是通过Buffer缓冲区来进行操作
- 在JDK1.8官方文档中，是这样描述的：一个通道代表了一个通向某实体的连接，这些实体可能是一个硬件设备，一个文件，一个网络套接字，或者是一个程序组件，它们具有执行某种或多种独立的IO操作的能力，例如读或写
#### SocketChannel

#### DatagramChannel
- ServerSocket和Socket是面向连接的，只有连接成功后才能发送数据；它们对应的ServerSocketChannel和SocketChannel也是。 
- 而DatagramSocket以及它对应的DatagramChannel则是无连接的，它们不需要连接就可以向指定地址和端口发送数据。
- 通过DatagramChannel，以UDP协议来向网络连接的两端读写数据。

#### PipeChannel
- Pipe管道的概念最先应该出现在Unix系统里，用来表示连接不同进程间的一种单向数据通道，很多Unix系统都提供了支持管道的API。Java NIO借用了这个概念，发明了NIO中的Pipe，它是指同一个Java进程内，不同线程间的一种单向数据管道，其sink端通道写入数据，source端通道则读出数据，其间可以保证数据按照写入顺序到达。

### Selector选择器
- Selector选择器就可以比喻成麦当劳的广播，一个线程能够管理多个Channel的状态
- 它是NIO中最关键的一个部分，Selector的作用就是用来轮询每个注册的Channel，一旦发现Channel有注册的事件发生，便获取事件然后进行处理
- 用单线程处理一个Selector，然后通过Selector.select()方法来获取到达事件，在获取了到达事件之后，就可以逐个地对这些事件进行响应处理。

![image-20201126174752840](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:47:53-image-20201126174752840.png)
> 用单线程处理一个Selector，然后通过Selector.select()方法来获取到达事件，在获取了到达事件之后，就可以逐个地对这些事件进行响应处理。

- 将Socket通道注册到Selector中，监听感兴趣的事件当感兴趣的时间就绪时，则会进去我们处理的方法进行处理每处理完一次就绪事件，删除该选择键(因为我们已经处理完了)
>其实这个时候已经可以看出是非阻塞的了


