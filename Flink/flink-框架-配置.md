[toc]
## 内存配置
### 垃圾回收器配置
#### 通用设置
-  env.java.opts: -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+AlwaysPreTouch -server -XX:+HeapDumpOnOutOfMemoryError
- 也可以通过在qi'd启动命令中添加
```
flink run -m yarn-cluster -yn 4 -yjm 2048 -ytm 8086 -c beam.count.OpsCount -yqu data-default \
-yD taskmanager.heap.size=4096 -yD yarn.heap-cutoff-ratio=0.6 -yD taskmanager.debug.memory.startLogThread=true -yD taskmanager.debug.memory.logIntervalMs=600000 \
-yz toratest -yst -yd ./beampoc-bundled-0.0.1-SNAPSHOT.jar --parallelism=4
-yD env.java.opts="-XX:NewRatio=2"
```
![image-20210202200907711](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202200907711.png)

#### 单独配置
- env.java.opts.jobmanager: -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+AlwaysPreTouch -server -XX:+HeapDumpOnOutOfMemoryError

![image-20210202200922774](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202200922774.png)

## 故障恢复和重启策略
- 自动故障恢复是 Flink 提供的一个强大的功能，在实际运行环境中，我们会遇到各种各样的问题从而导致应用挂掉，比如我们经常遇到的非法数据、网络抖动等。Flink 提供了强大的可配置故障恢复和重启策略来进行自动恢复

### 故障恢复
- Flink 支持了不同级别的故障恢复策略，jobmanager.execution.failover-strategy 的可配置项有两种：full 和 region。
- 当我们配置的故障恢复策略为 full时，集群中的Task发生故障，那么该任务的所有Task都会发生重启。而在实际生产环境中，我们的大作业可能有几百个Task，出现一次异常如果进行整个任务重启，那么经常会导致长时间任务不能正常工作，导致数据延迟。
- 但是事实上，我们可能只是集群中某一个或几个 Task 发生了故障，只需要重启有问题的一部分即可，这就是 Flink 基于 Region 的局部重启策略。在这个策略下，Flink 会把我们的任务分成不同的 Region，当某一个 Task 发生故障时，Flink 会计算需要故障恢复的最小 Region。
- Flink 在判断需要重启的 Region 时，采用了以下的判断逻辑：
    - 发生错误的 Task 所在的 Region 需要重启；
    - 如果当前 Region 的依赖数据出现损坏或者部分丢失，那么生产数据的 Region 也需要重启；
    - 为了保证数据一致性，当前 Region 的下游 Region 也需要重启。


### 重启策略
- Flink 在判断使用的哪种重启策略时做了默认约定，如果用户配置了 checkpoint，但没有设置重启策略，那么会按照固定延迟重启策略模式进行重启；如果用户没有配置 checkpoint，那么默认不会重启。

#### 固定延迟重启策略模式
- 固定延迟重启策略模式需要指定两个参数，首先Flink会根据用户配置的重试次数进行重试，每次重试之间根据配置的时间间隔进行重试

![image-20210202200940356](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202200940356.png)
```
restart-strategy: fixed-delay
restart-strategy.fixed-delay.attempts: 3
restart-strategy.fixed-delay.delay: 5 s
```
#### 失败率重启策略模式
- 这种策略的配置理解较为困难，我们举个例子，假如 5 分钟内若失败了 3 次，则认为该任务失败，每次失败的重试间隔为 5 秒

  ![image-20210202200957653](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202200957653.png)
```
restart-strategy.failure-rate.max-failures-per-interval: 3
restart-strategy.failure-rate.failure-rate-interval: 5 min
restart-strategy.failure-rate.delay: 5 s
```
#### 无重启策略模式
- 在这种情况下，如果我们的作业发生错误，任务会直接退出。
```
restart-strategy: none
```
```
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setRestartStrategy(RestartStrategies.noRestart());
```