## Flink Window

[TOC]

目前有许多数据分析的场景从批处理到流处理的演变，虽然可以将批处理作为流处理的特殊情况来处理，但是分析无穷集的流数据通常需要思维方式的转变并且具有其自己的术语，例如，“windowing（窗口化）”、“at-east-once（至少一次）”、“exactly-once（只有一次）” 。

对于刚刚接触流处理的人来说，这种转变和新术语可能会非常混乱。 Apache Flink 是一个为生产环境而生的流处理器，具有易于使用的API，可以用于定义高级流分析程序。Flink 的 API 在数据流上具有非常灵活的窗口定义，使其在其他开源流处理框架中脱颖而出。

在本节将讨论用于流处理的窗口的概念，介绍 Flink 的内置窗口，并解释它对自定义窗口语义的支持。

### 什么是 Window？

下面我们结合一个现实的例子来说明。

就拿交通传感器的示例：统计经过某红绿灯的汽车数量之和？

假设在一个红绿灯处，我们每隔 15 秒统计一次通过此红绿灯的汽车数量，如下图：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-064257.png)

可以把汽车的经过看成一个流，无穷的流，不断有汽车经过此红绿灯，因此无法统计总共的汽车数量。但是，我们可以换一种思路，每隔 15秒，我们都将与上一次的结果进行 sum 操作（滑动聚合），如下：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-064320.png)

这个结果似乎还是无法回答我们的问题，根本原因在于流是无界的，我们不能限制流，但可以在有一个有界的范围内处理无界的流数据。因此，我们需要换一个问题的提法：每分钟经过某红绿灯的汽车数量之和？

这个问题，就相当于一个定义了一个 Window（窗口），Window 的界限是 1分钟，且每分钟内的数据互不干扰，因此也可以称为翻滚（不重合）窗口，如下图：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-065851.png)

第一分钟的数量为 18，第二分钟是 28，第三分钟是 24……这样，1 个小时内会有 60 个 Window。再考虑一种情况，每 30 秒统计一次过去 1 分钟的汽车数量之和：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-071008.png)

此时，Window 出现了重合。这样，1 个小时内会有 120 个 Window。

### Window 有什么作用？

通常来讲，Window 就是用来对一个无限的流设置一个有限的集合，在有界的数据集上进行操作的一种机制。Window 又可以分为基于时间（Time-based）的 Window 以及基于数量（Count-based）的 window。

### Flink 自带的 Window

Flink 在 KeyedStream（DataStream 的继承类） 中提供了下面几种 Window：

  * 以时间驱动的 Time Window
  * 以事件数量驱动的 Count Window
  * 以会话间隔驱动的 Session Window

提供上面三种 Window 机制后，由于某些特殊的需要，DataStream API 也提供了定制化的 Window 操作，供用户自定义 Window。下面将先围绕上面说的三种 Window 来进行分析并教大家如何使用，然后对其原理分析，最后在解析其源码实现。

### Time Window 使用及源码分析

正如命名那样，Time Window 根据时间来聚合流数据。例如：一分钟的时间窗口就只会收集一分钟的元素，并在一分钟过后对窗口中的所有元素应用于下一个算子。

在 Flink 中使用 Time Window 非常简单，输入一个时间参数，这个时间参数可以利用 Time 这个类来控制，如果事前没指定
TimeCharacteristic 类型的话，则默认使用的是 ProcessingTime，如果对 Flink 中的 Time
还不了解的话，可以看前一篇文章 [Flink 中 Processing Time、Event Time、Ingestion Time对比及其使用场景分析]() 如下：


​    
    dataStream.keyBy(1)
        .timeWindow(Time.minutes(1)) //time Window 每分钟统计一次数量和
        .sum(1);


时间窗口的数据窗口聚合流程如下图所示：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-05-15-155127.jpg)

在第一个窗口中（1 ～ 2 分钟）和为 7、第二个窗口中（2 ～ 3 分钟）和为 12、第三个窗口中（3 ～ 4 分钟）和为 7、第四个窗口中（4 ～ 5分钟）和为 19。

该 timeWindow 方法在 KeyedStream 中对应的源码如下：

```java
//时间窗口
public WindowedStream<T, KEY, TimeWindow> timeWindow(Time size) {
    if (environment.getStreamTimeCharacteristic() == TimeCharacteristic.ProcessingTime) {
        return window(TumblingProcessingTimeWindows.of(size));
    } else {
        return window(TumblingEventTimeWindows.of(size));
    }
}
```

另外在 Time Window 中还支持滑动的时间窗口，比如定义了一个每 30s 滑动一次的 1 分钟时间窗口，它会每隔 30s
去统计过去一分钟窗口内的数据，同样使用也很简单，输入两个时间参数，如下：


​    
    dataStream.keyBy(1)
        .timeWindow(Time.minutes(1), Time.seconds(30)) //sliding time Window 每隔 30s 统计过去一分钟的数量和
        .sum(1);


滑动时间窗口的数据聚合流程如下图所示：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-05-15-155522.jpg)

在该第一个时间窗口中（1 ～ 2 分钟）和为 7，第二个时间窗口中（1:30 ~ 2:30）和为 10，第三个时间窗口中（2 ~ 3 分钟）和为12，第四个时间窗口中（2:30 ~ 3:30）和为 10，第五个时间窗口中（3 ~ 4 分钟）和为 7，第六个时间窗口中（3:30 ~ 4:30）和为11，第七个时间窗口中（4 ~ 5 分钟）和为 19。

该 timeWindow 方法在 KeyedStream 中对应的源码如下：

```java
//滑动时间窗口
public WindowedStream<T, KEY, TimeWindow> timeWindow(Time size, Time slide) {
    if (environment.getStreamTimeCharacteristic() == TimeCharacteristic.ProcessingTime) {
        return window(SlidingProcessingTimeWindows.of(size, slide));
    } else {
        return window(SlidingEventTimeWindows.of(size, slide));
    }
}
```


### Count Window 使用及源码分析

Apache Flink 还提供计数窗口功能，如果计数窗口的值设置的为 3 ，那么将会在窗口中收集 3 个事件，并在添加第 3
个元素时才会计算窗口中所有事件的值。

在 Flink 中使用 Count Window 非常简单，输入一个 long 类型的参数，这个参数代表窗口中事件的数量，使用如下：


​    
    dataStream.keyBy(1)
        .countWindow(3) //统计每 3 个元素的数量之和
        .sum(1);


计数窗口的数据窗口聚合流程如下图所示：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-05-16-045758.jpg)

该 countWindow 方法在 KeyedStream 中对应的源码如下：

    //计数窗口
    public WindowedStream<T, KEY, GlobalWindow> countWindow(long size) {
        return window(GlobalWindows.create()).trigger(PurgingTrigger.of(CountTrigger.of(size)));
    }


另外在 Count Window 中还支持滑动的计数窗口，比如定义了一个每 3 个事件滑动一次的 4 个事件的计数窗口，它会每隔 3 个事件去统计过去 4个事件计数窗口内的数据，使用也很简单，输入两个 long 类型的参数，如下：


​    
    dataStream.keyBy(1) 
        .countWindow(4, 3) //每隔 3 个元素统计过去 4 个元素的数量之和
        .sum(1);


滑动计数窗口的数据窗口聚合流程如下图所示：

![](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2019-05-16-065833.jpg)

该 countWindow 方法在 KeyedStream 中对应的源码如下：

    //滑动计数窗口
    public WindowedStream<T, KEY, GlobalWindow> countWindow(long size, long slide) {
        return window(GlobalWindows.create()).evictor(CountEvictor.of(size)).trigger(CountTrigger.of(slide));
    }


### Session Window 使用及源码分析

Apache Flink
还提供了会话窗口，是什么意思呢？使用该窗口的时候你可以传入一个时间参数（表示某种数据维持的会话持续时长），如果超过这个时间，就代表着超出会话时长。

在 Flink 中使用 Session Window 非常简单，你该使用 Flink KeyedStream 中的 window 方法，然后使用
ProcessingTimeSessionWindows.withGap()（不一定就是只使用这个），在该方法里面你需要做的是传入一个时间参数，如下：

    dataStream.keyBy(1)
        .window(ProcessingTimeSessionWindows.withGap(Time.seconds(5)))//表示如果 5s 内没出现数据则认为超出会话时长，然后计算这个窗口的和
        .sum(1);


会话窗口的数据窗口聚合流程如下图所示：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-05-16-150258.jpg)

该 Window 方法在 KeyedStream 中对应的源码如下：


​    
    //提供自定义 Window
    public <W extends Window> WindowedStream<T, KEY, W> window(WindowAssigner<? super T, W> assigner) {
        return new WindowedStream<>(this, assigner);
    }


### 如何自定义 Window？

当然除了上面几种自带的 Window 外，Apache Flink 还提供了用户可自定义的
Window，那么该如何操作呢？其实细心的同学可能已经发现了上面我写的每种 Window 的实现方式了，它们有 assigner、evictor、trigger。如果你没发现的话，也不要紧，这里我们就来了解一下实现 Window 的机制，这样我们才能够更好的学会如何自定义
Window。

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-073301.png)

### Window 源码定义

上面说了 Flink 中自带的 Window，主要利用了 KeyedStream 的 API 来实现，我们这里来看下 Window 的源码定义如下：

    public abstract class Window {
        //获取属于此窗口的最大时间戳
        public abstract long maxTimestamp();
    }


查看源码可以看见 Window 这个抽象类有如下实现类：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-17-163050.png)

**TimeWindow** 源码定义如下:


​    
    public class TimeWindow extends Window {
        //窗口开始时间
        private final long start;
        //窗口结束时间
        private final long end;
    }

**GlobalWindow** 源码定义如下：

    public class GlobalWindow extends Window {
    
        private static final GlobalWindow INSTANCE = new GlobalWindow();
    
        private GlobalWindow() { }
        //对外提供 get() 方法返回 GlobalWindow 实例，并且是个全局单例
        public static GlobalWindow get() {
            return INSTANCE;
        }
    }


### Window 组件之 WindowAssigner 使用及源码分析

到达窗口操作符的元素被传递给 WindowAssigner。WindowAssigner 将元素分配给一个或多个窗口，可能会创建新的窗口。

窗口本身只是元素列表的标识符，它可能提供一些可选的元信息，例如 TimeWindow中的开始和结束时间。注意，元素可以被添加到多个窗口，这也意味着一个元素可以同时在多个窗口存在。我们来看下 WindowAssigner 的代码的定义吧：

    public abstract class WindowAssigner<T, W extends Window> implements Serializable {
        //分配数据到窗口并返回窗口集合
        public abstract Collection<W> assignWindows(T element, long timestamp, WindowAssignerContext context);
    }


查看源码可以看见 WindowAssigner 这个抽象类有如下实现类：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-17-163413.png)

这些 WindowAssigner 实现类的作用介绍：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-05-16-155715.jpg)

如果你细看了上面图中某个类的具体实现的话，你会发现一个规律，比如我拿 TumblingEventTimeWindows 的源码来分析，如下：


​    
```java
public class TumblingEventTimeWindows extends WindowAssigner<Object, TimeWindow> {
    //定义属性
    private final long size;
    private final long offset;

    //构造方法
    protected TumblingEventTimeWindows(long size, long offset) {
        if (Math.abs(offset) >= size) {
            throw new IllegalArgumentException("TumblingEventTimeWindows parameters must satisfy abs(offset) < size");
        }
        this.size = size;
        this.offset = offset;
    }

    //重写 WindowAssigner 抽象类中的抽象方法 assignWindows
    @Override
    public Collection<TimeWindow> assignWindows(Object element, long timestamp, WindowAssignerContext context) {
        //实现该 TumblingEventTimeWindows 中的具体逻辑
    }

    //其他方法，对外提供静态方法，供其他类调用
}
```


从上面你就会发现 **套路** ：

1、定义好实现类的属性

2、根据定义的属性添加构造方法

3、重写 WindowAssigner 中的 assignWindows 等方法

4、定义其他的方法供外部调用

### Window 组件之 Trigger 使用及源码分析

Trigger 表示触发器，每个窗口都拥有一个 Trigger（触发器），该 Trigger决定何时计算和清除窗口。当先前注册的计时器超时时，将为插入窗口的每个元素调用触发器。在每个事件上，触发器都可以决定触发，即清除（删除窗口并丢弃其内容），或者启动并清除窗口。一个窗口可以被求值多次，并且在被清除之前一直存在。注意，在清除窗口之前，窗口将一直消耗内存。说了这么一大段，我们还是来看看 Trigger 的源码，定义如下：    

```java
public abstract class Trigger<T, W extends Window> implements Serializable {
    //当有数据进入到 Window 运算符就会触发该方法
    public abstract TriggerResult onElement(T element, long timestamp, W window, TriggerContext ctx) throws Exception;
    //当使用触发器上下文设置的处理时间计时器触发时调用
    public abstract TriggerResult onProcessingTime(long time, W window, TriggerContext ctx) throws Exception;
    //当使用触发器上下文设置的事件时间计时器触发时调用该方法
    public abstract TriggerResult onEventTime(long time, W window, TriggerContext ctx) throws Exception;
}
```


当有数据流入 Window 运算符时就会触发 onElement 方法、当处理时间和事件时间生效时会触发 onProcessingTime 和
onEventTime 方法。每个触发动作的返回结果用 TriggerResult 定义。继续来看下 TriggerResult 的源码定义：


​    
```java
public enum TriggerResult {

    //不做任何操作
    CONTINUE(false, false),

    //处理并移除窗口中的数据
    FIRE_AND_PURGE(true, true),

    //处理窗口数据，窗口计算后不做清理
    FIRE(true, false),

    //清除窗口中的所有元素，并且在不计算窗口函数或不发出任何元素的情况下丢弃窗口
    PURGE(false, true);
}
```


查看源码可以看见 Trigger 这个抽象类有如下实现类：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-17-163751.png)

这些 Trigger 实现类的作用介绍：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-05-17-145735.jpg)

如果你细看了上面图中某个类的具体实现的话，你会发现一个规律，拿 CountTrigger 的源码来分析，如下：


​    
```java
public class CountTrigger<W extends Window> extends Trigger<Object, W> {
    //定义属性
    private final long maxCount;

    private final ReducingStateDescriptor<Long> stateDesc = new ReducingStateDescriptor<>("count", new Sum(), LongSerializer.INSTANCE);
    //构造方法
    private CountTrigger(long maxCount) {
        this.maxCount = maxCount;
    }

    //重写抽象类 Trigger 中的抽象方法 
    @Override
    public TriggerResult onElement(Object element, long timestamp, W window, TriggerContext ctx) throws Exception {
        //实现 CountTrigger 中的具体逻辑
    }

    @Override
    public TriggerResult onEventTime(long time, W window, TriggerContext ctx) {
        return TriggerResult.CONTINUE;
    }

    @Override
    public TriggerResult onProcessingTime(long time, W window, TriggerContext ctx) throws Exception {
        return TriggerResult.CONTINUE;
    }
}
```


**套路** ：

  1. 定义好实现类的属性

  2. 根据定义的属性添加构造方法

  3. 重写 Trigger 中的 onElement、onEventTime、onProcessingTime 等方法

  4. 定义其他的方法供外部调用

### Window 组件之 Evictor 使用及源码分析

Evictor 表示驱逐者，它可以遍历窗口元素列表，并可以决定从列表的开头删除首先进入窗口的一些元素，然后其余的元素被赋给一个计算函数，如果没有定义Evictor，触发器直接将所有窗口元素交给计算函数。

我们来看看 Evictor 的源码定义如下：


​    
    public interface Evictor<T, W extends Window> extends Serializable {
        //在窗口函数之前调用该方法选择性地清除元素
        void evictBefore(Iterable<TimestampedValue<T>> elements, int size, W window, EvictorContext evictorContext);
        //在窗口函数之后调用该方法选择性地清除元素
        void evictAfter(Iterable<TimestampedValue<T>> elements, int size, W window, EvictorContext evictorContext);
    }


查看源码可以看见 Evictor 这个接口有如下实现类：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-17-163942.png)

这些 Evictor 实现类的作用介绍：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-05-17-153505.jpg)

如果你细看了上面三种中某个类的实现的话，你会发现一个规律，比如我就拿 CountEvictor 的源码来分析，如下：


​    
```java
public class CountEvictor<W extends Window> implements Evictor<Object, W> {
    private static final long serialVersionUID = 1L;

    //定义属性
    private final long maxCount;
    private final boolean doEvictAfter;

    //构造方法
    private CountEvictor(long count, boolean doEvictAfter) {
        this.maxCount = count;
        this.doEvictAfter = doEvictAfter;
    }
    //构造方法
    private CountEvictor(long count) {
        this.maxCount = count;
        this.doEvictAfter = false;
    }

    //重写 Evictor 中的 evictBefore 方法
    @Override
    public void evictBefore(Iterable<TimestampedValue<Object>> elements, int size, W window, EvictorContext ctx) {
        if (!doEvictAfter) {
            //调用内部的关键实现方法 evict
            evict(elements, size, ctx);
        }
    }

    //重写 Evictor 中的 evictAfter 方法
    @Override
    public void evictAfter(Iterable<TimestampedValue<Object>> elements, int size, W window, EvictorContext ctx) {
        if (doEvictAfter) {
            //调用内部的关键实现方法 evict
            evict(elements, size, ctx);
        }
    }

    private void evict(Iterable<TimestampedValue<Object>> elements, int size, EvictorContext ctx) {
        //内部的关键实现方法
    }

    //其他的方法
}
```


发现 **套路** ：

  1. 定义好实现类的属性

  2. 根据定义的属性添加构造方法

  3. 重写 Evictor 中的 evictBefore 和 evictAfter 方法

  4. 定义关键的内部实现方法 evict，处理具体的逻辑

  5. 定义其他的方法供外部调用

上面我们详细讲解了 Window 中的组件 WindowAssigner、Trigger、Evictor，然后继续回到问题：如何自定义 Window？上文讲解了 Flink 自带的 Window（Time Window、Count Window、Session Window），然后还分析了他们的源码实现，通过这几个源码，我们可以发现，它最后调用的都有一个方法，那就是 Window 方法，如下：

```java
//提供自定义 Window
public <W extends Window> WindowedStream<T, KEY, W> window(WindowAssigner<? super T, W> assigner) {
    return new WindowedStream<>(this, assigner);
}

//构造一个 WindowedStream 实例
public WindowedStream(KeyedStream<T, K> input,
        WindowAssigner<? super T, W> windowAssigner) {
    this.input = input;
    this.windowAssigner = windowAssigner;
    //获取一个默认的 Trigger
    this.trigger = windowAssigner.getDefaultTrigger(input.getExecutionEnvironment());
}
```

可以看到这个 Window 方法传入的参数是一个 WindowAssigner 对象（你可以利用 Flink 现有的WindowAssigner，也可以根据上面的方法来自定义自己的 WindowAssigner），然后再通过构造一个 WindowedStream
实例（在构造实例的会传入 WindowAssigner 和获取默认的 Trigger）来创建一个 Window。

另外你可以看到滑动计数窗口，在调用 window 方法之后，还调用了 WindowedStream 的 evictor 和 trigger
方法，trigger 方法会覆盖掉你之前调用 Window 方法中默认的 trigger，如下：


​    
```java
//滑动计数窗口
public WindowedStream<T, KEY, GlobalWindow> countWindow(long size, long slide) {
    return window(GlobalWindows.create()).evictor(CountEvictor.of(size)).trigger(CountTrigger.of(slide));
}

//trigger 方法
public WindowedStream<T, K, W> trigger(Trigger<? super T, ? super W> trigger) {
    if (windowAssigner instanceof MergingWindowAssigner && !trigger.canMerge()) {
        throw new UnsupportedOperationException("A merging window assigner cannot be used with a trigger that does not support merging.");
    }

    if (windowAssigner instanceof BaseAlignedWindowAssigner) {
        throw new UnsupportedOperationException("Cannot use a " + windowAssigner.getClass().getSimpleName() + " with a custom trigger.");
    }
    //覆盖之前的 trigger
    this.trigger = trigger;
    return this;
}
```


从上面的各种窗口实现，你就会发现了：Evictor 是可选的，但是 WindowAssigner 和 Trigger 是必须会有的，这种创建 Window的方法充分利用了 KeyedStream 和 WindowedStream 的 API，再加上现有的WindowAssigner、Trigger、Evictor，你就可以创建 Window了，另外你还可以自定义这三个窗口组件的实现类来满足你公司项目的需求。

### 小结与反思

本节从生活案例来分享关于 Window 方面的需求，进而开始介绍 Window 相关的知识，并把 Flink中常使用的三种窗口都一一做了介绍，并告诉大家如何使用，还分析了其实现原理。最后还对 Window 的内部组件做了详细的分析，为自定义 Window提供了方法。

不知道你看完本节后对 Window 还有什么疑问吗？你们是根据什么条件来选择使用哪种 Window 的？在使用的过程中有遇到什么问题吗？

