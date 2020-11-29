[toc]

# 传统IO

## IO 模型

- 在UNIX可以归纳成5种I/O模型
- 阻塞I/O
- 非阻塞I/O
- I/O多路复用
- 信号驱动I/O
- 异步I/O
> 举个生活中简单的例子，你妈妈让你烧水，小时候你比较笨啊，在哪里傻等着水开（同步阻塞）。等你稍微再长大一点，你知道每次烧水的空隙可以去干点其他事，然后只需要时不时来看看水开了没有（同步非阻塞）。后来，你们家用上了水开了会发出声音的壶，这样你就只需要听到响声后就知道水开了，在这期间你可以随便干自己的事情，你需要去倒水了（异步非阻塞）

### 文件描述符
- Linux 的内核将所有外部设备都看做一个文件来操作，对一个文件的读写操作会调用内核提供的系统命令(api)，返回一个file descriptor（fd，文件描述符）。
- 对一个socket的读写也会有响应的描述符，称为socketfd（socket文件描述符），描述符就是一个数字，指向内核中的一个结构体（文件路径，数据区等一些属性）。

### I/O运行过程

![image-20201126174250027](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:42:50-image-20201126174250027.png)
- 当应用程序调用read方法时，是需要等待的--->从内核空间中找数据，再将内核空间的数据拷贝到用户空间的。

### 阻塞I/O

![image-20201126174305629](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:43:06-image-20201126174305629.png)

### 非阻塞I/O模型

![image-20201126174316845](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:43:17-image-20201126174316845.png)



### I/O复用模型
- 在Linux下对文件的操作是利用文件描述符(file descriptor)来实现的。
- 在Linux下它是这样子实现I/O复用模型的：调用select/poll/epoll/pselect其中一个函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。

![image-20201126174338671](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:43:39-image-20201126174338671.png)
1. 当用户进程调用了select，那么整个进程会被block；
2. 而同时，kernel会“监视”所有select负责的socket；
3. 当任何一个socket中的数据准备好了，select就会返回；
4. 这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程(空间)。
> 所以，I/O 多路复用的特点是**通过一种机制一个进程能同时等待多个文件描述符**，而这些文件描述符其中的任意一个进入读就绪状态，select()函数就可以返回。

> select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。

> 听广播取餐，广播不是为我一个人服务。广播喊到我了，我过去取就Ok了。



## IO 中断原理
![image-20201126174355625](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:43:56-image-20201126174355625.png)

1. 用户进程调用read等系统调用向操作系统发出IO请求，请求读取数据到自己的内存缓冲区中。自己进入阻塞状态。
2. 操作系统收到请求后，进一步将IO请求发送磁盘。
3. 磁盘驱动器收到内核的IO请求，把数据从磁盘读取到驱动器的缓冲中。此时不占用CPU。当驱动器的缓冲区被读满后，向内核发起中断信号告知自己缓冲区已满。
4. 内核收到中断，使用CPU时间将磁盘驱动器的缓存中的数据拷贝到内核缓冲区中。
5. 如果内核缓冲区的数据少于用户申请的读的数据，重复步骤3跟步骤4，直到内核缓冲区的数据足够多为止。
6. 将数据从内核缓冲区拷贝到用户缓冲区，同时从系统调用中返回。完成任务。
```
缺点：用户的每次IO请求，都需要CPU多次参与。
```
## DMA原理

![image-20201126174407189](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:44:07-image-20201126174407189.png)

1. 用户进程调用read等系统调用向操作系统发出IO请求，请求读取数据到自己的内存缓冲区中。自己进入阻塞状态。
2. 操作系统收到请求后，进一步将IO请求发送DMA。然后让CPU干别的活去。
3. DMA进一步将IO请求发送给磁盘。
4. 磁盘驱动器收到DMA的IO请求，把数据从磁盘读取到驱动器的缓冲中。当驱动器的缓冲区被读满后，向DMA发起中断信号告知自己缓冲区已满。
5. DMA收到磁盘驱动器的信号，将磁盘驱动器的缓存中的数据拷贝到内核缓冲区中。此时不占用CPU。这个时候只要内核缓冲区的数据少于用户申请的读的数据，内核就会一直重复步骤3跟步
6. 直到内核缓冲区的数据足够多为止。
7. 当DMA读取了足够多的数据，就会发送中断信号给CPU。
8. CPU收到DMA的信号，知道数据已经准备好，于是将数据从内核拷贝到用户空间，系统调用返回。

## Java IO
- java io 的实现是阻塞式 BIO

![image-20201126174429341](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:44:29-image-20201126174429341.png)
