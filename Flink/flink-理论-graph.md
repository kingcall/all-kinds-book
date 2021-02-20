[toc]

## 概论
- Flink逻辑上将开发者编写的stream graph 的所有operator分解成job graph，物理上分解为并行的execution graph。每个并行的slot都是一个独立的task或subtask，同一个operator的不同的并行slot之间，并不共享数据
- 就数据本地性而言，Flink中的状态数据总是绑定到特定的task上。基于这种设计，一个task的状态总是本地的，在tasks之间没有通信。
- StreamGraph是对用户逻辑的映射。JobGraph在此基础上进行了一些优化，比如把一部分操作串成chain以提高效率。ExecutionGraph是为了调度存在的，加入了并行处理的概念。而在此基础上真正执行的是Task及其相关结构。

```
用户提交一个作业以后，Flink 首先在 client 端执行用户 main 函数以生成描述作业逻辑的拓扑（StreaGraph），其中 StreamGraph 的每个节点是用户定义的一个算子（Operator）。
随后 Flink 对 StreamGraph 进行优化，默认将不涉及 shuffle 并且并行度相同的相邻 Operator 串联起来成为 OperatorChain 形成 JobGraph，其中的每个节点称为 Vertice，是 OperatorChain 或独立的 Operator
```

### StreamGraph
- StreamGraph最接近代码所表达的逻辑层面的计算拓扑结构，按照用户代码的执行顺序向 StreamExecutionEnvironment 添加 StreamTransformation 构成流式图。
- 用来表示程序的拓扑结构（也就是根据用户代stream api代码生成最初的图）
- **在client端生成**

#### StreamGraph 的生成
- 构造StreamGraph的入口函数是 StreamGraphGenerator.generate(env, transformations)。
- 该函数会由触发程序执行的方法StreamExecutionEnvironment.execute()调用到。也就是说 StreamGraph 是在 Client 端构造的，这也意味着我们可以在本地通过调试观察 StreamGraph 的构造过程。
> env中存储 List<StreamTransformation<?>

```
public JobExecutionResult execute(String jobName) throws Exception {
	Preconditions.checkNotNull(jobName, "Streaming Job name should not be null.");

	return execute(getStreamGraph(jobName));
}

```

##### StreamTransformation
1. 描述DataStream之间的转化关系
2. 包含StreamOperator/UDF
3. OneInputTransformation/ transformTwoInputTransform/ SourceTransformation/ SinkTransformation/ SplitTransformation等

##### StreamNode/StreamEdge
- 通过StreamTransformation构造

### JobGraph
- 作业图（JobGraph）是Flink的运行时所能理解的作业表示，无论程序通过是DataStream还是DataSetAPI编写的，它们的JobGraph提交给JobManager以及之后的处理都将得到统一
- JobGraph从 StreamGraph生成，将可以串联合并的节点进行合并，将多个符合条件的节点chain在一起作为一个节点，**这样可以减少数据在节点之间流动所需要的序列化/反序列化/传输消耗**,设置节点之间的边，安排资源共享slot槽位和放置相关联的节点，上传任务所需的文件，设置检查点配置等。
- 相当于经过部分初始化和优化处理的任务图。

![image-20210202090913513](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202090913513.png)
```
The JobManager receives the JobGraph, which is a representation of the data flow consisting of operators (JobVertex) and intermediate results (IntermediateDataSet). 

Each operator has properties, like the parallelism and the code that it executes. In addition, the JobGraph has a set of attached libraries, that are necessary to execute the code of the operators.
```
- **在client端生成**

### ExecutionGraph
![image-20210202091019476](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202091019476.png)

- **JobManger根据JobGraph生成，并行化**

- ExecutionGraph由JobGraph转换而来，包含了任务具体执行所需的内容，是最贴近底层实现的执行图。

- ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。

- The JobManager transforms the JobGraph into an ExecutionGraph. The ExecutionGraph is a parallel version of the JobGraph

  ![image-20210202091040250](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202091040250.png)

### 物理执行图

- 实际执行图，不可见

## 流处理程序提交作业图
- Flink利用了“惰性求值”的概念，只有当最终调用execute方法时，才会真正开始执行。因此，execute方法是我们的切入点。

![image-20210202091054607](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202091054607.png)
### task
- 每个 Vertice 在执行层会被视为一个 Task，而一个 Task 对应多个 Subtask，Subtask 的数目即是用户设置的并行度。Subtask 根据 Flink 的调度策略和具体的部署环境及配置，会被分发到相同或者不同的机器或者进程上，其中有上下游依赖关系的 Subtask 会有数据传输的需要，这是通过基于 Netty 的 Network Stack 来完成的。
