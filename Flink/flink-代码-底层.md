[toc]
## 背景
- 大多数的datastream api 是无法访问一些底层的属性的，例如时间戳，watermark 等

![image-20210202201508659](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201508659.png)

> **灵活性高，但开发比较复杂**
## 功能

- 大多数 flink sql 函数都是利用处理函数实现的
- 位于最底层， 是core API 的底层实现
- 都实现了RichFunction 接口，支持open(), close ()
- 所有功能的实现都依赖Context 对象
- 利用低阶，构建一些新的组件(比如:利用其定时做一定情况下的匹配和缓存)
- 常说的状态编程都是在这个阶段完成的


### 访问时间戳和watermark
### 注册定时器
### 副输出

## 分类
### BroadcastProcessFunction
### CoProcessFunction
### KeyedBroadcastProcessFunction
### KeyedProcessFunction
### ProcessAllWindowFunction
### ProcessFunction
### ProcessJoinFunction
### ProcessWindowFunction