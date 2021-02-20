

![image-20210202090637764](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202090637764.png)

- 其中 Runtime-层对不同的执行环境提供了一套统一的分布式执行引擎
- 针对不同的执行环境，Flink 提供了一套统一的分布式作业执行引擎，也就是
Flink Runtime 这层。
- Flink 在 Runtime 层之上提供了 DataStream 和 DataSet 两
套 API，分别用来编写流作业与批作业，以及一组更高级的 API 来简化特定作业的
编写。


## accumulators
## broadcast
## client
## dispatcher
## filecache
## jobmanager
## jobmaster
## jobgraph
## taskmanager
## taskexecutor
## resourcemanager
## shuffle
## scheduler