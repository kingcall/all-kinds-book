[toc]

## StreamElement
### StreamRecord
- private T value;
- private long timestamp;
- private boolean hasTimestamp
### Watermark
- private final long timestamp

```
public final class Watermark extends StreamElement {
    public static final Watermark MAX_WATERMARK = new Watermark(9223372036854775807L);
    private final long timestamp;

    public Watermark(long timestamp) {
        this.timestamp = timestamp;
    }

    public long getTimestamp() {
        return this.timestamp;
    }

    public boolean equals(Object o) {
        return this == o || o != null && o.getClass() == Watermark.class && ((Watermark)o).timestamp == this.timestamp;
    }

    public int hashCode() {
        return (int)(this.timestamp ^ this.timestamp >>> 32);
    }

    public String toString() {
        return "Watermark @ " + this.timestamp;
    }
}
```
### StreamStatus
- public static final int IDLE_STATUS = -1;
- public static final int ACTIVE_STATUS = 0;

### LatencyMarker
- 为了监控Flink数据端到端的数据延迟
- LatencyMarker表示的是延时标记，其携带了一个时间戳，由source端周期性发出，用于统计由source端到下游operator所需耗时，通常用于延时测试使用，该标记在默认情况下是关闭的，可通过配置metrics.latency.interval开启，表示产生LatencyMarker的周期
- latencyMarker仅在source处产生,latencyMarker对象包含的是source operator的区分标志subIndex以及vertexID（根据subIndex以及vertexID区分是否是同一个Marker）以及携带了发送的时间
- LatencyMark在各个operator间传递，在每个operator处将会比较LatencyMark和它当前的系统时间来决定延迟的大小，并存入LatencyGauge，每一个operator都会维护这样一个Metric（因此LatencyMark的实现就是基于TM和JM集群的机器系统时间是进行过同步的）
- 当Operator有多个Output的时候，**会随机选择一个来发送，这确保了每一个Marker在整个流中只会出现一次**，repartition也不会导致LatencyMark的数量暴增。
- 在sink operator处会维护source的最近128个latencyMarker，通过一个LatencyGauge来展示

#### 具体实现
- 默认latencyTrackingInterval是2000，也就是2s发送一个LatencyMarker。在StreamSource中判断，如果开启了latency track.那么就会定期发送LatencyMarker。

```
latencyMarkTimer = processingTimeService.scheduleAtFixedRate(
   new ProcessingTimeCallback() {
      
      public void (long timestamp) throws Exception {
         try {
            
            output.emitLatencyMarker(new LatencyMarker(timestamp, vertexID, subtaskIndex));
         } catch (Throwable t) {
            // we catch the Throwables here so that we don't trigger the processing
            // timer services async exception handler
            LOG.warn("Error while emitting latency marker.", t);
         }
      }
   },
   0L,
   latencyTrackingInterval);
```
- 在Operator中处理：首先进行LatencyMarker处理再发送，在AbstractStreamOperator类中定义的latencyMarker处理
```
public void reportLatency(LatencyMarker marker, boolean isSink) {
   LatencySourceDescriptor sourceDescriptor = LatencySourceDescriptor.of(marker, !isSink);
   DescriptiveStatistics sourceStats = latencyStats.get(sourceDescriptor);
   if (sourceStats == null) {
      // 512 element window (4 kb)
      sourceStats = new DescriptiveStatistics(this.historySize);
      latencyStats.put(sourceDescriptor, sourceStats);
   }
   long now = System.currentTimeMillis();
   sourceStats.addValue(now - marker.getMarkedTime());  //所以latency的计算时间是当前时间减去marker摄入的时间
}
 
1.首先生成一个Latency的描述符，sink operator的区分不同subIndex为不同LatencySource，其他operator不区分subIndex，只按照vertexID来区分
2.然后生成对应Latency的描述符的最近"historySize:128"个Latency的值（WindowSize controls the number of values that contribute to the reported statistics. ）
3.在这里值没有在web展示的原因是因为guage展示的不是一个数字，因而无法被展示
```
- 然后只要不是sink类型的operator，就会往后继续传递LatencyMarker。随机选择一个Channel来发送,这里就是为了保证一个latencyMarker在整个流中只会出现一次。这里和watermark的机制有点不一样，waterMark是遍历全部的channel来发送。

```
public void emitLatencyMarker(LatencyMarker latencyMarker) {
   if(outputs.length <= 0) {
      // ignore
   } else if(outputs.length == 1) {
      outputs[0].emitLatencyMarker(latencyMarker);
   } else {
      // randomly select an output
      outputs[RNG.nextInt(outputs.length)].emitLatencyMarker(latencyMarker);
   }
}
```
- LatencyMarker不会参与窗口时间的计算，应该说是不参与任何operator的计算，因此**只能用来衡量数据在整个DAG中流通的速度不能衡量operator计算的时间**，这个只能通过单测来进行计算
- StreamInputProcessor#processInput 这里进行对进入的element进行处理，对于watermark和LatencyMarker类型会先处理发送掉，不会经由后面的windowOperator或其他operator来处理

```
if (result.isFullRecord()) {
   StreamElement recordOrMark = deserializationDelegate.getInstance();
 
   if (recordOrMark.isWatermark()) {
      long watermarkMillis = recordOrMark.asWatermark().getTimestamp();
      if (watermarkMillis > watermarks[currentChannel]) {
         watermarks[currentChannel] = watermarkMillis;
         long newMinWatermark = Long.MAX_VALUE;
         for (long watermark: watermarks) {
            newMinWatermark = Math.min(watermark, newMinWatermark);
         }
         if (newMinWatermark > lastEmittedWatermark) {
            lastEmittedWatermark = newMinWatermark;
            synchronized (lock) {
               streamOperator.processWatermark(new Watermark(lastEmittedWatermark));
            }
         }
      }
      continue;
   } else if(recordOrMark.isLatencyMarker()) {
      // handle latency marker
      synchronized (lock) {
         streamOperator.processLatencyMarker(recordOrMark.asLatencyMarker());
      }
      continue;
   } else {
      // now we can do the actual processing
      StreamRecord<IN> record = recordOrMark.asRecord();
      synchronized (lock) {
         numRecordsIn.inc();
         streamOperator.setKeyContextElement1(record);
         streamOperator.processElement(record);
      }
      return true;
   }
  }
```
- sink中只进行report不再进行forward了（StreamSink.java）
```
protected void reportOrForwardLatencyMarker(LatencyMarker maker) {
   // all operators are tracking latencies
   this.latencyGauge.reportLatency(maker, true);
 
   // sinks don't forward latency markers
}
```
- 非sinkOperator

  

![image-20210202090545282](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202090545282.png)

#### 总结

- LatencyMarker能够较好的监控因网络抖动或数据反压引起的延迟，可以提前预警反压情况
- 在正常情况（没有反压）下数据在DAG图中的流动延迟大概0.5s左右，所以说Flink的确是一个很快的引擎：）

