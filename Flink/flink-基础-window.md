[toc]
# 窗口

   - 一个Window代表有限对象的集合。一个窗口有一个最大的时间戳，该时间戳意味着在其代表的某时间点——所有应该进入这个窗口的元素都已经到达 
   - Window就是用来对一个无限的流设置一个有限的集合，在有界的数据集上进行操作的一种机制。window又可以分为基于时间（Time-based）的window以及基于数量（Count-based）的window。
   - Flink DataStream API提供了Time和Count的window，同时增加了基于Session的window。同时，由于某些特殊的需要，DataStream API也提供了定制化的window操作，供用户自定义window。

## 窗口的组成

### 窗口分配器
- assignWindows将某个带有时间戳timestamp的元素element分配给一个或多个窗口，并返回窗口集合
- getDefaultTrigger 返回跟WindowAssigner关联的默认触发器
- getWindowSerializer返回WindowAssigner分配的窗口的序列化器

 - 窗口分配器定义如何将数据元分配给窗口。这是通过WindowAssigner 在window(...)（对于被Keys化流）或windowAll()（对于非被Keys化流）调用中指定您的选择来完成的。 
- WindowAssigner负责将每个传入数据元分配给一个或多个窗口。Flink带有预定义的窗口分配器，用于最常见的用例
     即翻滚窗口， 滑动窗口，会话窗口和全局窗口。
- 您还可以通过扩展WindowAssigner类来实现自定义窗口分配器。
- 所有内置窗口分配器（全局窗口除外）都根据时间为窗口分配数据元，这可以是处理时间或事件时间。

### State
- 状态，用来存储窗口内的元素，如果有 AggregateFunction，则存储的是增量聚合的中间结果。

### 窗口函数
选择合适的计算函数，减少开发代码量提高系统性能
#### 增量聚合函数(窗口只维护状态)
- ReduceFunction
- AggregateFunction
- FoldFunction
#### 全量聚合函数(窗口维护窗口内的数据)
- ProcessWindowFunction
    - 全量计算
    - 支持功能更加灵活
    - 支持状态操作

### 触发器

![image-20210202200655485](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202200655485.png)

- EventTimeTrigger基于事件时间的触发器，对应onEventTime
- ProcessingTimeTrigger
 基于当前系统时间的触发器，对应onProcessingTime
 ProcessingTime 有最好的性能和最低的延迟。但在分布式计算环境中ProcessingTime具有不确定性，相同数据流多次运行有可能产生不同的计算结果。
- ContinuousEventTimeTrigger
- ContinuousProcessingTimeTrigger
- CountTrigger


   - Trigger确定何时窗口函数准备好处理窗口（由窗口分配器形成）。每个都有默认值。
        如果默认触发器不符合您的需要，您可以使用指定自定义触发器。WindowAssignerTriggertrigger(...)
   - 触发器界面有五种方法可以Trigger对不同的事件做出反应：
      - onElement()为添加到窗口的每个数据元调用该方法。
      - onEventTime()在注册的事件时间计时器触发时调用该方法。
      - onProcessingTime()在注册的处理时间计时器触发时调用该方法。
      - 该onMerge()方法与状态触发器相关，并且当它们的相应窗口合并时合并两个触发器的状态，例如当使用会话窗口时。
      - 最后，该clear()方法在移除相应窗口时执行所需的任何动作。
   - 默认触发器
      - 默认触发器GlobalWindow是NeverTrigger从不触发的。因此，在使用时必须定义自定义触发器GlobalWindow。
      - 通过使用trigger()您指定触发器会覆盖a的默认触发器WindowAssigner。例如，如果指定a CountTrigger，TumblingEventTimeWindows则不再根据时间进度获取窗口，
      而是仅按计数。现在，如果你想根据时间和数量做出反应，你必须编写自己的自定义触发器。
      - event-time窗口分配器都有一个EventTimeTrigger作为默认触发器。该触发器在watermark通过窗口末尾时出发。  
#### 触发器分类 
##### CountTrigger
一旦窗口中的数据元数量超过给定限制，就会触发。所以其触发机制实现在onElement中
##### ProcessingTimeTrigger
基于处理时间的触发。
##### EventTimeTrigger 
根据 watermarks 度量的事件时间进度进行触发。

#####  PurgingTrigger 
- 另一个触发器作为参数作为参数并将其转换为清除触发器。
- 其作用是在 Trigger 触发窗口计算之后将窗口的 State 中的数据清除。

- ![image-20210202200710573](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202200710573.png)前两条数据先后于20:01和20:02进入窗口，此时 State 中的值更新为3，同时到了Trigger的触发时间，输出结果为3。

  ![image-20210202200733128](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202200733128.png)

- 由于 PurgingTrigger 的作用，State 中的数据会被清除。

![image-20210202200744793](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202200744793.png)

#####  DeltaTrigger

###### DeltaTrigger 的应用
- 有这样一个车辆区间测试的需求，车辆每分钟上报当前位置与车速，每行进10公里，计算区间内最高车速。

![image-20210202200802480](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202200802480.png)
#### 触发器原型
   - onElement
   - onProcessingTime
   - onEventTime
   - onMerge 
   - clear

#### 说明
   - TriggerResult可以是以下之一
        - CONTINUE 什么都不做
        - FIRE_AND_PURGE 触发计算，然后清除窗口中的元素
        - FIRE 触发计算  默认情况下，内置的触发器只返回 FIRE，不会清除窗口状态。
        - PURGE 清除窗口中的元素   
   -  所有的事件时间窗口分配器都有一个 EventTimeTrigger 作为默认触发器。一旦 watermark 到达窗口末尾，这个触发器就会被触发。   
   - 全局窗口(GlobalWindow)的默认触发器是永不会被触发的 NeverTrigger。因此，在使用全局窗口时，必须自定义一个触发器。
   - 通过使用 trigger() 方法指定触发器，将会覆盖窗口分配器的默认触发器。例如，如果你为 TumblingEventTimeWindows 指定 CountTrigger，
        那么不会再根据时间进度触发窗口，而只能通过计数。目前为止，如果你希望基于时间以及计数进行触发，则必须编写自己的自定义触发器。  

## 窗口的分类
### 被Keys化Windows
  可以理解为按照原始数据流中的某个key进行分类，拥有同一个key值的数据流将为进入同一个window，多个窗口并行的逻辑流

    ```text
    stream
           .keyBy(...)               <-  keyed versus non-keyed windows
           .window(...)              <-  required: "assigner"
          [.trigger(...)]            <-  optional: "trigger" (else default trigger)
          [.evictor(...)]            <-  optional: "evictor" (else no evictor)
          [.allowedLateness(...)]    <-  optional: "lateness" (else zero)
          [.sideOutputLateData(...)] <-  optional: "output tag" (else no side output for late data)
           .reduce/aggregate/fold/apply()      <-  required: "function"
          [.getSideOutput(...)]      <-  optional: "output tag"
    ```
### 非被Keys化Windows
- 不做分类，每进入一条数据即增加一个窗口，多个窗口并行，每个窗口处理1条数据
- WindowAll 将元素按照某种特性聚集在一起，该函数不支持并行操作，默认的并行度就是1，所以如果使用这个算子的话需要注意一下性能问题

    ```text
    stream
           .windowAll(...)           <-  required: "assigner"
          [.trigger(...)]            <-  optional: "trigger" (else default trigger)
          [.evictor(...)]            <-  optional: "evictor" (else no evictor)
          [.allowedLateness(...)]    <-  optional: "lateness" (else zero)
          [.sideOutputLateData(...)] <-  optional: "output tag" (else no side output for late data)
           .reduce/aggregate/fold/apply()      <-  required: "function"
          [.getSideOutput(...)]      <-  optional: "output tag"
    ```
### 区别
   - 对于被Key化的数据流，可以将传入事件的任何属性用作键（此处有更多详细信息）。
   - 拥有被Key化的数据流将允许您的窗口计算由多个任务并行执行，因为每个逻辑被Key化的数据流可以独立于其余任务进行处理。
     引用相同Keys的所有数据元将被发送到同一个并行任务。
### Time-Based window
   每一条记录来了以后会根据时间属性值采用不同的window assinger 方法分配给一个或者多个窗口。
   - EventTime 数据本身携带的时间
   - ProcessingTime 处理时间
   - IngestionTime
### Count-Based window  
### 会话(session)窗口
   - SessionWindow中的Gap是一个非常重要的概念，它指的是session之间的间隔。
   - 如果session之间的间隔大于指定的间隔，数据将会被划分到不同的session中。比如，设定5秒的间隔，0-5属于一个session，5-10属于另一个session
### 总结
SlidingEventTimeWindows,  
SlidingProcessingTimeWindows,   
TumblingEventTimeWindows,  
TumblingProcessingTimeWindows  
   - 基于时间的滑动窗口
        - SlidingEventTimeWindows
        - SlidingProcessingTimeWindows
   - 基于时间的翻滚窗口
        - TumblingEventTimeWindows
        - TumblingProcessingTimeWindows
   - 基于计数的滑动窗口
        - 
   - 基于计数的翻滚窗口
   - 会话窗口
        会话窗口：一条记录一个窗口
        - ProcessingTimeSessionWindows
        - EventTimeSessionWindows
   - 全局窗口(GlobalWindows)
        - GlobalWindow是一个全局窗口，被实现为单例模式。其maxTimestamp被设置为Long.MAX_VALUE。
        - 该类内部有一个静态类定义了GlobalWindow的序列化器：Serializer。




## 延迟
默认情况下，当水印超过窗口末尾时，会删除延迟数据元。
但是，Flink允许为窗口 算子指定最大允许延迟。允许延迟指定数据元在被删除之前可以延迟多少时间，并且其默认值为0.
在水印通过窗口结束之后但在通过窗口结束加上允许的延迟之前到达的数据元，仍然添加到窗口中。
根据使用的触发器，延迟但未丢弃的数据元可能会导致窗口再次触发。就是这种情况EventTimeTrigger。

当指定允许的延迟大于0时，在水印通过窗口结束后保持窗口及其内容。在这些情况下，当迟到但未掉落的数据元到达时，它可能触发窗口的另一次触发。
这些射击被称为late firings，因为它们是由迟到事件触发的，与之相反的main firing 是窗口的第一次射击。在会话窗口的情况下，后期点火可以进一步导致窗口的合并，因为它们可以“桥接”两个预先存在的未合并窗口之间的间隙。
后期触发发出的数据元应该被视为先前计算的更新结果，即，您的数据流将包含同一计算的多个结果。根据您的应用程序，您需要考虑这些重复的结果或对其进行重复数据删除。

## 窗口的使用
   - Flink为每个窗口创建一个每个数据元的副本。鉴于此，翻滚窗口保存每个数据元的一个副本（一个数据元恰好属于一个窗口，除非它被延迟）
     动窗口会每个数据元创建几个复本，如“ 窗口分配器”部分中所述。因此，尺寸为1天且滑动1秒的滑动窗口可能不是一个好主意。 
   - ReduceFunction，AggregateFunction并且FoldFunction可以显着降低存储要求，因为它们急切地聚合数据元并且每个窗口只存储一个值。
     相反，仅使用 ProcessWindowFunction需要累积所有数据元。 
## Evictor
   - 它剔除元素的时机是：在触发器触发之后，在窗口被处理(apply windowFunction)之前
   - Flink 的窗口模型允许在窗口分配器和触发器之外指定一个可选的驱逐器(Evictor)。可以使用 evictor(...) 方法来完成。
      驱逐器能够在触发器触发之后，以及在应用窗口函数之前或之后从窗口中移除元素
   - 默认情况下，所有内置的驱逐器在窗口函数之前使用
   - 指定驱逐器可以避免预聚合(pre-aggregation)，因为窗口内所有元素必须在应用计算之前传递给驱逐器。
   - Flink不保证窗口内元素的顺序。这意味着虽然驱逐者可以从窗口的开头移除元素，但这些元素不一定是先到的还是后到的。
###  内置的Evitor
   - TimeEvitor
       - 以毫秒为单位的时间间隔作为参数，对于给定的窗口，找到元素中的最大的时间戳max_ts，并删除时间戳小于max_ts - interval的所有元素。
       - 本质上是将罪行的元素选出来
   - CountEvitor
       - 保持窗口内元素数量符合用户指定数量，如果多于用户指定的数量，从窗口缓冲区的开头丢弃剩余的元素。
   - DeltaEvitor   
       - 使用 DeltaFunction和 一个阈值，计算窗口缓冲区中的最后一个元素与其余每个元素之间的 delta 值，并删除 delta 值大于或等于阈值的元素。 
       - 通过定义的DeltaFunction 和 Threshold ,计算窗口中元素和最新元素的 Delta 值，将Delta 值超过 Threshold的元素删除
## watermark
   - watermark是一种衡量Event Time进展的机制，它是数据本身的一个隐藏属性。
   - watermark Apache Flink为了处理EventTime 窗口计算提出的一种机制,本质上也是一种时间戳，
      由Apache Flink Source或者自定义的Watermark生成器按照需求Punctuated或者Periodic两种方式生成的一种系统Event，
      与普通数据流Event一样流转到对应的下游算子，接收到Watermark Event的算子以此不断调整自己管理的EventTime clock。 
      算子接收到一个Watermark时候，框架知道不会再有任何小于该Watermark的时间戳的数据元素到来了，所以Watermark可以看做是告诉Apache Flink框架数据流已经处理到什么位置(时间维度)的方式。
   - 通常基于Event Time的数据，自身都包含一个timestamp.watermark是用于处理乱序事件的，而正确的处理乱序事件，通常用watermark机制结合window来实现。 
   - waterMark 的触发时间机制(waterMark >= window_end_time)
       - 当第一次触发之后，以后所有到达的该窗口的数据(迟到数据)都会触发该窗口
       - 定义允许延迟，所以 waterMark<window_end_time+allowedLateness 的这段时间内，有数据落入窗口也会触发计算，当
       waterMark>=window_end_time+allowedLateness 是窗口被关闭，数据被丢弃
        - 对于out-of-order的数据，Flink可以通过watermark机制结合window的操作，来处理一定范围内的乱序数据,（新进来的数据）晚于前面进来的数据，但是该数据所在窗口没有被触发，
             这个时候数据还是有效的——EventTime<WaterMark 的
       - 对于out-of-order的数据，延迟太多
       - 注意，如果不定义允许最大迟到时间，并且在有很多数据迟到的情况下，会严重影响正确结果，只要Event Time < watermark时间就会触发窗口，也就是说迟到的每一条数据都会触发
       该窗口    
### 产生方式
   - Punctuated 
        - 数据流中每一个递增的EventTime都会产生一个Watermark（其实是根据某个计算条件来做判断）。 
        - 在实际的生产中Punctuated方式在TPS很高的场景下会产生大量的Watermark在一定程度上对下游算子造成压力，所以只有在实时性要求非常高的场景才会选择Punctuated的方式进行Watermark的生成。
        - 每个事件都会携带事件，可以根据该时间产生一个watermark 或者可以根据事件携带的其他标志——业务的结束标志
   - Periodic - 周期性的（一定时间间隔或者达到一定的记录条数）产生一个Watermark。

    在实际的生产中Periodic的方式必须结合时间和积累条数两个维度继续周期性产生Watermark，否则在极端情况下会有很大的延时。
### 背景
   - 流处理从事件产生，到流经source，再到operator，中间是有一个过程和时间的。虽然大部分情况下，流到operator的数据都是按照事件产生的时间顺序来的
   - 但是也不排除由于网络、背压等原因，导致乱序的产生（out-of-order或者说late element）。   
   - 对于late element，我们又不能无限期的等下去，必须要有个机制来保证一个特定的时间后，必须触发window去进行计算了
   - 它表示当达到watermark到达之后,在watermark之前的数据已经全部达到(即使后面还有延迟的数据 
### 解决的问题
   - Watermark的时间戳可以和Event中的EventTime 一致，也可以自己定义任何合理的逻辑使得Watermark的时间戳不等于Event中的EventTime，
      Event中的EventTime自产生那一刻起就不可以改变了，不受Apache Flink框架控制，
      而Watermark的产生是在Apache Flink的Source节点或实现的Watermark生成器计算产生(如上Apache Flink内置的 Periodic Watermark实现),
      Apache Flink内部对单流或多流的场景有统一的Watermark处理。
  - 默认情况下小于watermark 时间戳的event 会被丢弃吗 
### 多流waterMark
   - 在实际的流计算中往往一个job中会处理多个Source的数据，对Source的数据进行GroupBy分组，那么来自不同Source的相同key值会shuffle到同一个处理节点，
      并携带各自的Watermark，Apache Flink内部要保证Watermark要保持单调递增，多个Source的Watermark汇聚到一起时候可能不是单调自增的
   - Apache Flink内部实现每一个边上只能有一个递增的Watermark， 当出现多流携带Eventtime汇聚到一起(GroupBy or Union)时候，
      Apache Flink会选择所有流入的Eventtime中最小的一个向下游流出。从而保证watermark的单调递增和保证数据的完整性
### 理解
   - 默认情况下watermark 已经触发过得窗口，即使有新数据（迟到）落进去不会被计算 ，迟到的意思

    watermark>=window_n_end_time && window_n_start_time<=vent_time<window_n_end_time(即数据属于这个窗口)
   - 允许迟到
      watermark>=window_n_end_time && watermark<window_n_end_time+lateness && window_n_start_time<=vent_time<window_n_end_time 
      在 watermark 大于窗口结束时间不超过特定延迟范围时，落在此窗口内的数据是有效的，可以触发窗口。
## 窗口作为缓存的工具，否则每次都要和数据库进行交互     
## 窗口聚合
   - 增量聚合
        - 窗口内来一条数据就计算一次
   - 全量聚合    
        - 一次计算整个窗口里的所有元素（可以进行排序，一次一批可以针对外部链接）
        - 使用
            - 窗口之后调用 apply ,创建的元素里面方法的参数是一个迭代器
## 常用的一些方法
   - window
   - timeWindow和 countWind
   - process 和 apply


AssignerWithPeriodicWatermarks或接口AssignerWithPunctuatedWatermarks。
简而言之，前一个接口将会周期性发送Watermark，而第二个接口根据一些到达数据的属性，例如一旦在流中碰到一个特殊的element便发送Watermark。   


### 自定义窗口

- Window Assigner：负责将元素分配到不同的window。
- Trigger即触发器，定义何时或什么情况下Fire一个window。 
  - 对于CountWindow，我们可以直接使用已经定义好的Trigger：CountTrigger trigger(CountTrigger.of(2))
- Evictor（可选） 驱逐者，即保留上一window留下的某些元素。
- 最简单的情况，如果业务不是特别复杂，仅仅是基于Time和Count，我们其实可以用系统定义好的WindowAssigner以及Trigger和Evictor来实现不同的组合： 



## window 出现数据倾斜
- window 产生数据倾斜指的是数据在不同的窗口内堆积的数据量相差过多。本质上产生这种情况的原因是数据源头发送的数据量速度不同导致的。出现这种情况一般通过两种方式来解决：
- 在数据进入窗口前做预聚合
- 重新设计窗口聚合的 key



