[监控指标](https://mp.weixin.qq.com/s/ORfbLyjg48QWdyTZ-tNdfA)

## slow-log 
- config get slowlog*，默认阈值10ms,0.01s,队列长度128

![image-20201125212020539](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:20:21-image-20201125212020539.png)
### 最佳实现
#### slower-than 参数的合理设置
- slow log 默认的查询时间阈值是10毫秒，但是如果平均查询时间超过10 毫秒的话，redis的qps就不足100了，所以可以根据并发量适当降低超时阈值。

#### 慢查询日志的持久化
- 对于线上任务可以适当调整慢查询日志队列的大小，慢查询日志本身占用的内存空间是很小的，可以适当调大
- 对于慢查询日志可以将其持久化到外部系统(mysql)，以便更好的分析
- 也可以通过日志工具直接输出到监控平台，直接监控分析


## 键迁移
### move
在redis 内部使用（redis 多个数据库之间进行迁移）
### dump+restore
可以在不同的redis实例之间进行迁移
#### 过程
1 dump key   dump命令会将键值进行序列化，默认采用的

2 restore key value
### migrate
- 实际上是 dump restore del 三个命令的原子组合
- 3.0版本后支持多个键 在水平扩容中有重要作用