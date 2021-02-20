[toc]
## 背景
- Async I/O 是阿里巴巴贡献给社区的一个呼声非常高的特性，于1.2版本引入。主要目的是为了解决与外部系统交互时网络延迟成为了系统瓶颈的问题。
- 对于实时处理，当需要使用外部存储数据染色的时候，需要小心对待，不能让与外部系统之间的交互延迟对流处理的整个工作进度起决定性的影响。
```
在mapfunction等算子里访问外部存储，实际上该交互过程是同步的：比如请求a发送到数据库，那么mapfunction会一直等待响应。在很多案例中，这个等待过程是非常浪费函数时间的。
```
![image-20210218194258511](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194258511.png)
## 异步IO 的实现
- Flink中提供了一种异步IO的模式，不需要使用map函数阻塞式的加载数据，而是使用异步方法同时处理大量请求。不过这就需要数据库支持异步请求，如果不支持异步请求也可以手动维护线程池调用，只不过效率上没有原生的异步client更高效。比如Mysql可以通过Vertx支持异步查询，HBase2.x也支持异步查询
### 实现接口
```
class MyAsyncReq extends RichAsyncFunction<IN,OUT>{
    @Override
    public void open(..) throws Exception {}
    @Override
    public void close() throws Exception {}
    @Override
     def asyncInvoke(input: IN, resultFuture: ResultFuture[OUT]): Unit
}
或者
AsyncFunction
```
- 其中open中需要定义连接或者连接池，close中进行释放，asyncInvoke执行异步查询。
- asyncInvoke 方法本身的话，是一个**串行**调用,不会涉及到多线程的操作，在这个方法里面通过异步调用客户端进行依次调用
- 最终返回一个ResultFuture 对象

### 使用接口
```
AsyncDataStream.unorderedWait(
stream, 
new MyAsyncReq(), 
1000, 
TimeUnit.MILLISECONDS, 
100)
```
- stream为主要的事件流，
- myasyncreq是异步IO类，
- 1000为异步请求的超时时间，
- 100是同时进行异步请求的最大数


## 消息的顺序性
- Flink 使用队列来实现不同的输出模式，并抽象出一个队列的接口StreamElementQueue
- treamElementQueue有两种具体实现，分别是 OrderedStreamElementQueue 和 UnorderedStreamElementQueue。UnorderedStreamElementQueue 比较有意思，它使用了一套逻辑巧妙地实现完全无序和 EventTime 无序。
### 有序
- 有序比较简单，使用一个队列就能实现。所有新进入该算子的元素（包括 watermark），都会包装成 Promise 并按到达顺序放入该队列。如下图所示，尽管P4的结果先返回，但并不会发送，只有 P1 （队首）的结果返回了才会触发 Emitter 拉取队首元素进行发送。
![image-20210218194357777](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194357777.png)

#### 性能分析
- 该模式下，消息在异步I/O算子前后的顺序一致，先请求的先返回，即有序模式。为实现有序模式，算子将请求返回的结果放入缓存，直到该请求之前的结果全部返回或超时。
- 该模式通常情况下回引入额外的时延以及在checkpoint过程中会带来开销

### 无序
- 使用两个队列就能实现，一个 uncompletedQueue 一个 completedQueue。所有新进入该算子的元素，同样的包装成 Promise 并放入 uncompletedQueue 队列，当uncompletedQueue队列中任意的Promise返回了数据，则将该 Promise 移到 completedQueue 队列中，并通知 Emitter 消费。
![image-20210218194413949](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194413949.png)

#### 性能分析
- 当异步的请求完成时，其结果立马返回，不考虑结果顺序即乱序模式。当以processing time作为时间属性时，该模式可以获得最小的延时和最小的开销，
#### 无序表现
- 在 ProcessingTime 的情况下，完全无序，先返回的结果先发送。
- 在 EventTime 的情况下，watermark 不能超越消息，消息也不能超越 watermark，也就是说 watermark 定义的顺序的边界。在两个 watermark 之间的消息的发送是无序的，但是在watermark之后的消息不能先于该watermark之前的消息发送。
### ProcessingTime 无序

- ProcessingTime 无序也比较简单，因为没有 watermark，不需要协调 watermark 与消息的顺序性，所以使用两个队列就能实现，一个 uncompletedQueue 一个 completedQueue。所有新进入该算子的元素，同样的包装成 Promise 并放入 uncompletedQueue 队列，当uncompletedQueue队列中任意的Promise返回了数据，则将该 Promise 移到 completedQueue 队列中，并通知 Emitter 消费

### EventTime 无序
- EventTime 无序类似于有序与 ProcessingTime 无序的结合体。
- 因为有 watermark，需要协调 watermark 与消息之间的顺序性，所以uncompletedQueue中存放的元素从原先的 Promise 变成了 Promise 集合。
- 如果进入算子的是消息元素，则会包装成 Promise 放入队尾的集合中。
- 如果进入算子的是 watermark，也会包装成Promise并放到一个独立的集合中，再将该集合加入到 uncompletedQueue 队尾，最后再创建一个空集合加到 uncompletedQueue 队尾。
- 这样**，watermark 就成了消息顺序的边界**。只有处在队首的集合中的 Promise 返回了数据，才能将该 Promise 移到 completedQueue 队列中，由 Emitter 消费发往下游。
-** 只有队首集合空了，才能处理第二个集合**。这样就保证了当且仅当某个 watermark 之前所有的消息都已经被发送了，该 watermark 才能被发送。
![image-20210218194428267](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194428267.png)

## 异步客户端
- 为了实现以异步I/O访问数据库或K/V存储，数据库等需要有能支持异步请求的client
- 若是没有，可以通过创建多个同步的client并使用线程池处理同步call的方式实现类似并发的client，但是这方式没有异步I/O的性能好。
- AsyncFunction不是以多线程方式调用的，一个AsyncFunction实例按顺序为每个独立消息发送请求
### vertex
### lettuce
### 没有异步客户端

## 原理
- AsyncDataStream.orderedWait 的主要工作就是创建了一个 AsyncWaitOperator。AsyncWaitOperator 是支持异步 IO 访问的算子实现，该算子会运行 AsyncFunction 并处理异步返回的结果

### AsyncWaitOperator
![image-20210218194445472](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194445472.png)
- AsyncWaitOperator 主要由两部分组成：StreamElementQueue 和 Emitter
```
图中E5表示进入该算子的第五个元素（”Element-5”），在执行过程中首先会将其包装成一个 “Promise” P5，然后将P5放入队列。
最后调用 AsyncFunction 的 ayncInvoke 方法，该方法会向外部服务发起一个异步的请求，并注册回调。该回调会在异步请求成功返回时调用 AsyncCollector.collect 方法将返回的结果交给框架处理。
实际上 AsyncCollector 是一个 Promise ，也就是 P5，在调用 collect 的时候会标记 Promise 为完成状态，并通知 Emitter 线程有完成的消息可以发送了
Emitter 就会从队列中拉取完成的 Promise ，并从 Promise 中取出消息发送给下游。
```
#### StreamElementQueue 
StreamElementQueue 是一个 Promise 队列，所谓 Promise 是一种异步抽象表示将来会有一个值，这个队列是未完成的 Promise 队列，也就是进行中的请求队列。
#### Emitter 
Emitter 是一个单独的线程，负责发送消息（收到的异步回复）给下游。

## 异步与容错
- AsyncFunction 与 flink 的检查点进行了良好的集成，所有正在等待响应的记录都会被写入检查点并且在重启的时候进行重新发送。