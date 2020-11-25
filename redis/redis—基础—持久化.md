[toc]
## 持久化
Redis是一个支持数据持久化的内存数据库，可以对Redis设置，让Redis周期性的把更新的数据同步到磁盘中保证数据持久化。
Redis支持的持久化策略有两种，分别是：RDB和AOF。

> 不论是RDB还是AOF都是先写入一个临时文件，然后通过 rename 完成文件的替换工作。

### RDB 持久化
- 在指定的时间间隔能对你的数据进行快照存储，Redis默认的持久化方式就是RDB



#### 持久化原理
- RDB的工作原理为当 Redis 需要做持久化时，Redis 会 fork 一个子进程，子进程将数据写到磁盘上一个临时 RDB 文件中。当子进程完成写临时文件后，将原来的 RDB 替换掉，这样的好处就是可以 copy-on-write。


#### 自动持久化
```
# 时间策略
save 900 1
save 300 10
save 60 10000

# 文件名称
dbfilename dump.rdb

# 文件保存路径
dir /home/work/app/redis/data/

# 如果持久化出错，主进程是否停止写入
stop-writes-on-bgsave-error yes

# 是否压缩
rdbcompression yes

# 导入时是否检查
rdbchecksum yes
```
- save 60 1000
> 让Redis满足“60秒内至少有1000个键被改动”这一个条件时，自动保存一次数据集。

##### 策略的意义
- 因为Redis每个时段的**读写请求肯定不是均衡的**，为了**平衡性能与数据安全**，我们可以自由定制什么情况下触发备份。所以这里就是根据自身Redis写入情况来进行合理配置。

##### 自动触发的场景
1. 根据我们的 save m n 配置规则自动触发；
2. 从节点全量复制时，主节点发送rdb文件给从节点完成复制操作，主节点会触发 bgsave；
3. 执行 debug reload 时；
4. 执行 shutdown时，如果没有开启aof，也会触发。

##### 自动化持久流程



![image-20201125212855320](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:28:55-image-20201125212855320.png)

#### 手动持久化

- 除了在配置文件中使用save关键字设置RDB快照，还可以在命令行中手动执行命令生成RDB快照，进入redis客户端执行命令save或bgsave可以生成dump.rdb文件。
- 每次执行命令都会将所有redis内存快照保存到一个rdb文件里，**并覆盖原有的rdb快照文件**。
- save是同步命令，bgsave是异步命令，**bgsave会从redis主进程fork出一个子进程专门用来生成rdb二进制文件**(这个时候的阻塞只有fork 进程的时间)

#### RDB的优势与不足
##### 优势
- 速度快，体积小
- 因为RDB的持久化方式是可以在时间间隔内进行数据快照，所以RDB非常适合用于**灾难恢复**。例如设置每小时备份一次，或每天备份一次总的，从而方便数据的追溯和还原到不同版本。


##### 不足
- 不能做到实时持久化
> 特定时间下才进行一次持久化，所以易丢失数据；例如你设置30分钟备份一次数据，但是如果Redis服务器发生故障，那么就可能丢失好几分钟的数据没能备份。

- RDB 文件格式兼容问题
- 庞大数据时，保存时会出现性能问题(一次持久化的时间可能会很长)


### AOF 持久化
- AOF：先把命令追加到操作日志的尾部，保存所有历史操作。
- 记录每次对服务器写的操作,当服务器**重启的时候会重新执行这些命令来恢复原始的数据**。
- AOF 主要解决持久化实时性的问题

#### 持久化原理
- 每一个写命令都通过write函数追加到 appendonly.aof 中，当Redis出现故障重启时，将会读取 AOF 文件进行“重放”以恢复到 Redis 关闭前的状态。
-  AOF(默认关闭的，但是你使用 BGREWRITEAOF 这个命令，AOF文件依然可以写出)

```
# 是否开启aof
appendonly yes
# 文件名称
appendfilename "appendonly.aof"
# 同步方式
appendfsync everysec
# aof重写期间是否同步
no-appendfsync-on-rewrite no
# 重写触发配置
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 加载aof时如果有错如何处理
aof-load-truncated yes

# 文件重写策略
aof-rewrite-incremental-fsync yes
```
#### 自动触发
-  就是根据配置规则来触发，当然自动触发的整体时间还跟Redis的定时任务频率有关系。

#### 手动触发
- bgrewriteaof

#### appendfsync 的三种策略
- always：把每个写命令都立即同步到aof，很慢，但是很安全
- everysec：每秒同步一次，是折中方案
- no：redis不处理交给OS来处理，非常快，但是也最不安全

> 一般情况下都采用 everysec 配置，这样可以兼顾速度与安全，最多损失1s的数据。

#### AOF 写流程

#### AOF重写流程
![image-20201125212920713](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:29:21-image-20201125212920713.png)

- AOF的整个流程大体来看可以分为两步，一步是命令的实时写入（如果是 appendfsync everysec 配置，会有1s损耗），第二步是对aof文件的重写
- 对于增量追加到文件这一步主要的流程是：命令写入=>追加到aof_buf => 同步到aof磁盘,如果实时写入磁盘会带来非常高的磁盘IO，影响整体性能

##### 重写细节
- 在重写期间，由于主进程依然在响应命令，为了保证最终备份的完整性；因此它依然会写入旧的AOF file中，如果重写失败，能够保证数据不丢失。
- 为了把重写期间响应的写入信息也写入到新的文件中，因此也会为子进程保留一个buf，防止新写的file丢失数据。
- 重写是直接**把当前内存的数据生成对应命令，并不需要读取老的AOF文件进行分析、命令合并**(当前生成的肯定比现有的AOF小)
- AOF文件直接采用的文本协议，主要是兼容性好、追加方便、可读性高可认为修改修复

##### 重写的意义
- 减小文件的体积
- 提升文件被redis 加载的速度

##### 重写策略
- auto-aof-rewrite-percentage 100  是64M的一倍的时候重写（也就是128M）
- auto-aof-rewrite-min-size 64mb   当AOF文件多大时重写   默认64M

#### AOF 的优势与不足
1. 数据非常完整，故障恢复丢失数据少；可对历史操作进行处理。
2. 在备份相同的数据集时，AOF的文件体积大于RDB的文件体积；
3. 	5AOF使用fsync策略的话，AOF的速度可能会慢于RDB。

### 混合持久化

- 重启redis恢复数据集时，很少会使用rdb来恢复内存状态，因为会丢失大量数据。通常会使用aof日志恢复数据，但是重放aof日志性能相对rdb来说要慢很多，这样在redis实例很大的情况下，启动需要花费很长时间。Redis4.0为了解决这个问题，带来了新的持久化选项——混合持久化
> aof-use-rdb-preamble yes
- 如果开启了混合持久化，aof在重写时，不再是单纯将内存数据转换为RESP命令写入aof文件，而是将重写这一刻之前的内存做rdb快照处理，并且将rdb快照内容和增量的aof修改内存数据的命令存在一起，都写入新的aof文件，新的aof文件一开始不叫appendonly.aof，等到重写完成后，新的aof文件才会进行改名，原子的覆盖原有的aof文件，完成新旧两个aof文件的替换。
于是在redis重启的时候，可以先加载rdb文件，然后再重放增量的aof日志就可以完全替代之前的aof全量文件重放，因此重启效率大幅得到提高。



![image-20201125212950682](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:29:51-image-20201125212950682.png)

## 从持久化中恢复数据
- 启动时会先检查AOF文件是否存在，如果不存在就尝试加载RDB。因为AOF保存的数据更完整，通过上面的分析我们知道AOF基本上最多损失1s的数据。

  

  ![image-20201125213014078](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:30:14-image-20201125213014078.png)


## 性能与实践
> RDB的快照、AOF的重写都需要fork，这是一个重量级操作，会对Redis造成阻塞。因此为了不影响Redis主进程响应，我们需要尽可能降低阻塞

1. 降低fork的频率，比如可以手动来触发RDB生成快照、与AOF重
2. 控制Redis最大使用内存，防止fork耗时过长；
3. 合理配置Linux的内存分配策略，避免因为物理内存不足导致fork失败。

## 数据备份

- 获取 redis 目录可以使用 CONFIG 命令

![image-20201125213044221](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:30:44-image-20201125213044221.png)

### 备份
- SAVE 命令用于创建当前数据库的备份，该命令将在 redis 数据目录中创建dump.rdb文件。
- 创建 redis 备份文件也可以使用命令 BGSAVE，该命令在后台执行。

### 恢复数据
- 如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 的数据目录并启动服务即可。
- 可以做到不丢失任何一条数据(系统的缓存)
