[toc]

## StreamTask 概论
- StreamOperator上层是由StreamTask调用，也就是说StreamTask会在发生不同阶段、不同动作去调用StreamOperator对应的方法，在Flink中将StreamTask称之为Invokable
## 继承结构



![image-20210218202640529](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218202640529.png)

### AbstractInvokable
- 是一个抽象类，代表最顶层的Invokable，在这个抽象类里面声明了最要的方法invoke，可以认为是task执行的起点
### StreamTask
```
AbstractTwoInputStreamTask
OneInputStreamTask
SourceStreamTask
StoppableSourceStreamTask
StreamIterationHead
StreamIterationTail
TwoInputSelectableStreamTask
TwoInputStreamTask
```

## StreamTask 执行过程
![image-20210218202653467](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218202653467.png)
### 初始化
- 初始化过程分为两种，一个是StreamTask初始化，另外一个是StreamOperator初始化：

#### StreamTask初始化
- 包括了与checkpoint相关的的例如Statebackend创建等，当然这些最终也会调用到StreamOperator,另外还会调用init方法，具体实现是在其实现类里面，主要完成StreamTwoInputProcessor初始化，其主要负责读取数据相关处理

#### StreamOperator初始化
- 调用initializeState、openAllOperators方法，initializeState会调用到StreamOperator的initializeState方法，完成状态初始化过程，openAllOperators会调用StreamOperator的open方法，调用与用户相关的初始化过程

### 执行过程
- 执行过程主要就是调用run方法，实现也是在其实现类里面，对于SourceStreamTask就是生产数据，对于OneInputStreamTask/TwoInputStreamTask主要就是执行读取数据与之后的数据处理流程，正常情况是会一直循环执行

### 资源释放
- 任务正常结束或者是异常停止的执行动作，包括closeAllOperators、cleanup、disposeAllOperators等，在这里说明一点close与dispose，在正常流程里面是可以看到close的调用，但是在异常流程只能看到dispose的调用，但是在dispose里面是可以看到对于是否调用close的检测，如果没有调用则需要调用一次，也就是说close是一定会被调用的，因此我们在userFunction里面如果涉及到资源链接一定要在close里面执行资源的释放。