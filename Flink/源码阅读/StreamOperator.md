[toc]
## StreamOperator
StreamOperator是任务执行过程中实际处理类，上层由StreamTask调用，下层调用UserFunction,列举一些常见的StreamOperator
```
env.addSource对应StreamSource
dataStream.map 对应StreamMap
dataStrem.window对应WindowOperator
dataStream.addSink对应StreamSink
dataStream.keyBy(..).process对应KeyedProcessOperator
StreamOperator涉及数据处理、checkpoint、状态存储、定时调用等，本篇幅将从源码角度分析StreamOperator所涉及的核心调用流程。
```
## 源码分析
### 继承关系
```
public interface StreamOperator<OUT> extends CheckpointListener, KeyContext, Disposable, Serializable 

public abstract class AbstractStreamOperator<OUT>implements StreamOperator<OUT>, Serializable
public abstract class AbstractUdfStreamOperator<OUT, F extends Function> extends AbstractStreamOperator<OUT> implements OutputTypeConfigurable<OUT>
```
- CheckpointListener接口，notifyCheckpointComplete表示checkpoint完成后的回调方法
- KeyContext接口，用于当前key的切换，使用在KeyedStream中state的key设置
- Disposable接口，dispose方法定义了资源释放
- Serializable序列化接口
### AbstractStreamOperator
- AbstractStreamOperator是StreamOperator的基础抽象实现类，所有的operator都必须继承该抽象类
```
StreamOperator接口提供了其生命周期的抽象方法，例如初始化方法setup、open、initializeState，checkpoint相关方法prepareSnapshotPreBarrier、snapshotState，但是我们没有必要去自己一一实现这些方法，可以继承其抽象类AbstractStreamOperator，覆盖一些我们需要重写的方法
```

### AbstractUdfStreamOperator 
- AbstractUdfStreamOperator 是继承AbstractStreamOperator的抽象实现类，其内部包含了userFunction, 在Task的生命周期都会调用userFunction中对应的方法
- AbstractStreamOperator是所有operator的基础抽象类，而AbstractUdfStreamOperator则是面向userFunction调用

### OneInputStreamOperator
- OneInputStreamOperator 是继承StreamOperator的接口，表示处理一个输入，包含了processElement/processWatermark/processLatencyMarker方法
```
OneInputStreamOperator实现类StreamMap、WindowOperator、KeyedProcessOperator等单流入处理operator
```
### TwoInputStreamOperator
- TwoInputStreamOperator是继承StreamOperator的接口，分别表示两个输入的Operator，包含了processElement/processWatermark/processLatencyMarker方法
```
TwoInputStreamOperator实现类CoStreamMap、KeyedCoProcessOperator、IntervalJoinOperator等多流处理operator
```
### StreamSource
-对于source端不需要接受上游数据，也就不需要实现OneInputStreamOperator或者TwoInputStreamOperator接口，如果我们需要接收上游数据就必须实现这两个接口中的一个，主要看一个输入还是两个输入来选择
```
source端的operator,其既没有实现OneInputStreamOperator接口也没有实现TwoInputStreamOperator接口，由于其为流处理的源头，不需要接受输入
```