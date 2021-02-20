## 延迟监控
```
如果每一条数据都打上时间监控 输出时间- 输入时间，会大量的消耗性能,所以flink 是通过另外一种方法做的
```
### 原理
- 在source周期性的插入一条特殊的数据LatencyMarker,LatencyMarker初始化的时候会带上它产生时的时间
- 每次当task接收到的数据是LatencyMarker的时候他就用 当前时间 - LatencyMarker时间 = lateTime 并发送到指标收集系统
- 接着继续把这个LatencyMarker往下游emit
### 配置
- flink中通过开启配置   metrics.latency.interval  来开启latency后就可以在metric中看到TaskManagerJobMetricGroup/operator_id/operator_subtask_index/latency指标了

## 窗口监控
## watermark 监控
