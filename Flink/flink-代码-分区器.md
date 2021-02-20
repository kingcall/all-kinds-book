[toc]
## 分区器
- 在流进行转换操作后，Flink通过分区器来精确得控制数据流向。
- 分区器，最终会体现在DataStream的API中用来对数据流进行物理分区

![image-20210202201323790](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201323790.png)



## 分区器的基类
- StreamPartitioner是Flink流分区器的基类，它只定义了一个抽象方法：
```
public abstract StreamPartitioner<T> copy();
```
- 但这个方法并不是各个分区器之间互相区别的地方，定义不同的分区器的核心在于——各个分区器需要实现channel选择的接口方法
```
 int[] selectChannels(T record, int numChannels);
```
- 该方法针对当前的record以及所有的channel数目，返回一个针对当前这条记录采用的output channel的索引数组。（注意这里返回的是数组，说明一个记录可能会输出到多个channel这点我们后面会谈到）。
- 该接口方法来自于StreamPartitioner实现的接口ChannelSelector。

## 分区器的基类接口
- ChannelSelector: 接口，决定将记录写入哪个Channel

### 主要方法
- void setup(int numberOfChannels): 初始化输出Channel的数量。
- int selectChannel(T record): 根据当前记录以及Channel总数，决定应将记录写入下游哪个Channel。八大分区策略的区别主要在这个方法的实现上。
- boolean isBroadcast(): 是否是广播模式。决定了是否将记录写入下游所有Channel


## 分区器的实现
- BroadcastPartitioner
- ConfigurableStreamPartitioner
- CustomPartitionerWrapper
- ForwardPartitioner
- GlobalPartitioner
- KeyGroupStreamPartitioner
- RebalancePartitioner
- RescalePartitioner
- ShufflePartitioner
- StreamPartitioner

### GlobalPartitioner

- 默认选择了索引为0的channel进行输出——也就是说将记录输出到下游Operator的第一个实例
```
@Internal
public class GlobalPartitioner<T> extends StreamPartitioner<T> {
	private static final long serialVersionUID = 1L;

	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
	    // 对每条记录，只选择下游operator的第一个Channel(也就是第一个实例)
		return 0;
	}

	@Override
	public StreamPartitioner<T> copy() {
		return this;
	}

	@Override
	public String toString() {
		return "GLOBAL";
	}
}

```


### ForwardPartitioner
- 该分区器将记录转发给在本地运行的下游的(归属于subtask)的operattion
- 将记录输出到下游本地的operator实例。ForwardPartitioner分区器要求上下游算子并行度一样。上下游Operator同属一个SubTasks
```
@Internal
public class ForwardPartitioner<T> extends StreamPartitioner<T> {
	private static final long serialVersionUID = 1L;

	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
		return 0;
	}

	public StreamPartitioner<T> copy() {
		return this;
	}

	@Override
	public String toString() {
		return "FORWARD";
	}
}
```

### ShufflePartitioner
- 混洗分区器，该分区器会在所有output channel中选择一个随机的进行输出。

```
@Internal
public class ShufflePartitioner<T> extends StreamPartitioner<T> {
	private static final long serialVersionUID = 1L;

	private Random random = new Random();

	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
		return random.nextInt(numberOfChannels);
	}

	@Override
	public StreamPartitioner<T> copy() {
		return new ShufflePartitioner<T>();
	}

	@Override
	public String toString() {
		return "SHUFFLE";
	}
}

```

### BroadcastPartitioner
- 广播分区将上游数据集输出到下游Operator的每个实例中。适合于大数据集Join小数据集的场景。
- selectChannel 方法的注释很有意思——广播分区不需要也不支持选择Channel,因为会输出到下游每个Channel中
- 启用广播模式，此时Channel选择器会选择下游所有Channel，所以直接启用了广播模式
```
@Internal
public class BroadcastPartitioner<T> extends StreamPartitioner<T> {
	private static final long serialVersionUID = 1L;

	/**
	 * Note: Broadcast mode could be handled directly for all the output channels
	 * in record writer, so it is no need to select channels via this method.
	 */
	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
		throw new UnsupportedOperationException("Broadcast partitioner does not support select channels.");
	}

	@Override
	public boolean isBroadcast() {
		return true;
	}

	@Override
	public StreamPartitioner<T> copy() {
		return this;
	}

	@Override
	public String toString() {
		return "BROADCAST";
	}
}
```
### RescalePartitioner
- 这种分区器会根据上下游算子的并行度，循环的方式输出到下游算子的每个实例。
- 假设上游并行度为2，编号为A和B。下游并行度为4，编号为1，2，3，4。
那么A则把数据循环发送给1和2，B则把数据循环发送给3和4。
- 假设上游并行度为4，编号为A，B，C，D。下游并行度为2，编号为1，2。那么A和B则把数据发送给1，C和D则把数据发送给2。


```
@Internal
public class RescalePartitioner<T> extends StreamPartitioner<T> {
    private static final long serialVersionUID = 1L;
    private int nextChannelToSendTo = -1;

    public RescalePartitioner() {
    }

    public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
        if (++this.nextChannelToSendTo >= this.numberOfChannels) {
            this.nextChannelToSendTo = 0;
        }

        return this.nextChannelToSendTo;
    }

    public StreamPartitioner<T> copy() {
        return this;
    }

    public String toString() {
        return "RESCALE";
    }
}
```

### KeyGroupStreamPartitioner
- Hash分区器。会将数据按 Key的Hash值输出到下游算子实例中。

```
public class KeyGroupStreamPartitioner<T, K> extends StreamPartitioner<T> implements ConfigurableStreamPartitioner {
    private static final long serialVersionUID = 1L;
    private final KeySelector<T, K> keySelector;
    private int maxParallelism;

    public KeyGroupStreamPartitioner(KeySelector<T, K> keySelector, int maxParallelism) {
        Preconditions.checkArgument(maxParallelism > 0, "Number of key-groups must be > 0!");
        this.keySelector = (KeySelector)Preconditions.checkNotNull(keySelector);
        this.maxParallelism = maxParallelism;
    }

    public int getMaxParallelism() {
        return this.maxParallelism;
    }

    public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
        Object key;
        try {
            key = this.keySelector.getKey(((StreamRecord)record.getInstance()).getValue());
        } catch (Exception var4) {
            throw new RuntimeException("Could not extract key from " + ((StreamRecord)record.getInstance()).getValue(), var4);
        }

        return KeyGroupRangeAssignment.assignKeyToParallelOperator(key, this.maxParallelism, this.numberOfChannels);
    }

    public StreamPartitioner<T> copy() {
        return this;
    }

    public String toString() {
        return "HASH";
    }

    public void configure(int maxParallelism) {
        KeyGroupRangeAssignment.checkParallelismPreconditions(maxParallelism);
        this.maxParallelism = maxParallelism;
    }
}
```
### CustomPartitionerWrapper

## 分区器的使用

### dataStream.partitionCustom()
- 自定义分区
- 
```
  /**
   * Partitions a tuple DataStream on the specified key fields using a custom partitioner.
   * This method takes the key position to partition on, and a partitioner that accepts the key
   * type.
   *
   * Note: This method works only on single field keys.
   */     
  def partitionCustom[K: TypeInformation](partitioner: Partitioner[K], field: Int) : DataStream[T] =
    asScalaStream(stream.partitionCustom(partitioner, field))

  /**
   * Partitions a POJO DataStream on the specified key fields using a custom partitioner.
   * This method takes the key expression to partition on, and a partitioner that accepts the key
   * type.
   *
   * Note: This method works only on single field keys.
   */
  def partitionCustom[K: TypeInformation](partitioner: Partitioner[K], field: String)
        : DataStream[T] =
    asScalaStream(stream.partitionCustom(partitioner, field))

  /**
   * Partitions a DataStream on the key returned by the selector, using a custom partitioner.
   * This method takes the key selector to get the key to partition on, and a partitioner that
   * accepts the key type.
   *
   * Note: This method works only on single field keys, i.e. the selector cannot return tuples
   * of fields.
   */
  def partitionCustom[K: TypeInformation](partitioner: Partitioner[K], fun: T => K)
      : DataStream[T] = {
    
    val keyType = implicitly[TypeInformation[K]]
    val cleanFun = clean(fun)
    
    val keyExtractor = new KeySelector[T, K] with ResultTypeQueryable[K] {
      def getKey(in: T) = cleanFun(in)
      override def getProducedType(): TypeInformation[K] = keyType
    }

    asScalaStream(stream.partitionCustom(partitioner, keyExtractor))
  }
```
### dataStream.shuffle
- 根据均匀分布随机分配元素
```
@PublicEvolving
public DataStream<T> shuffle() {
	return setConnectionType(new ShufflePartitioner<T>());
}
```
### dataStream.rebalance
- 分区元素循环，每个分区创建相等的负载。在存在数据偏斜时用于性能优化。
```
public DataStream<T> rebalance() {
	return setConnectionType(new RebalancePartitioner<T>());
}

```


### dataStream.rescale

- 分区元素循环到下游操作的子集。如果您希望拥有管道，例如，从源的每个并行实例扇出到多个映射器的子集以分配负载但又不希望发生rebalance（）会产生完全重新平衡，那么这非常有用。
- 这将仅需要本地数据传输而不是通过网络传输数据，具体取决于其他配置值，例如TaskManagers的插槽数。上游操作发送元素的下游操作的子集取决于上游和下游操作的并行度。
- 例如，如果上游操作具有并行性2并且下游操作具有并行性4，则一个上游操作将元素分配给两个下游操作，而另一个上游操作将分配给另外两个下游操作。另一方面，如果下游操作具有并行性2而上游操作具有并行性4，那么两个上游操作将分配到一个下游操作，而另外两个上游操作将分配到其他下游操作。在不同并行度不是彼此的倍数的情况下，一个或多个下游操作将具有来自上游操作的不同数量的输入

### dataStream.broadcast

### dataStream.global
```
public DataStream<T> global() {
		return setConnectionType(new GlobalPartitioner<T>());
	}
```
