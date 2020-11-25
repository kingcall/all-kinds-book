## 设置 Redis slowlog
### 配置文件配置
1. 可以通过配置 redis.conf 来完成。

### 命令行配置
2. 运行时，使用 CONFIG GET 和 CONFIG SET 命令配置
    - config set slowlog-max-len 10000 设置队列大小
    - config set slowlog-log-slower-than 1 设置阈值大小

> 其中slowlog-log-slower-than表示slowlog的划定界限，只有query执行时间大于slowlog-log-slower-than的才会定义成慢查询，才会被slowlog进行记录。slowlog-log-slower-than设置的单位是微妙，默认是10000微妙，也就是10


> slowlog-max-len表示慢查询最大的条数，当slowlog超过设定的最大值后，会将最早的slowlog删除，是个FIFO队列


## 常用的命令

SLOWLOG LEN 查看慢日志的条数

SLOWLOG GET 10 获得前10条慢日志

SLOWLOG RESET 清空慢查询日志


对于用scala来写spark程序员来说，spark streaming连接redis spark-redis.jar肯定是首选，但有一个坑就是ssc.sparkContext.fromRedisKV 底层是通过scan来获取数据的，当redis中数据过多时，效率十分低下
设置并行度的时候，要注意不能大于redis的个数

