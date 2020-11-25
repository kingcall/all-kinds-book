[toc]

## 查看内存使用情况
![image-20201125213518402](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:35:18-image-20201125213518402.png)
### info memory
- used_memory:已经使用了的内存大小，包括redis进程内部开销和你的cache的数据所占用的内存，单位byte,包括使用的虚拟内存（即swap)

- used_memory_human:以可读格式返回的used_memory。
- used_memory_rss:从操作系统角度显示的redis(redis 进程)占用的物理内存，与top及ps命令看到的值是一致的，**除了分配器分配的内存之外，used_memory_rss还包括进程运行本身需要的内存、内存碎片等，但是不包括虚拟内存**

- used_memory_rss_human:以可读格式返回的used_memory。
- used_memory_peak:内存的最大使用值，表示used_memory的峰值。
- used_memory_peak_human:
- used_memory_memory_lua: Lua引擎所消耗的内存大小
- mem_fragmentation_ratio: - used_memory_rss/used_memory的比值，表示内存碎片率
- used_allocator:Redis所使用的内存分配器

![image-20201125215027765](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:50:28-image-20201125215027765.png)

#### 总结
- used_memory和used_memory_rss，**前者是从Redis角度得到的量，后者是从操作系统角度得到的量**。二者之所以有所不同，一方面是因为内存碎片和Redis进程运行需要占用内存，使得前者可能比后者小，另一方面虚拟内存的存在，使得前者可能比后者大。
-  由于虚拟内存不一定使用，但是内存碎片一定会有，所以mem_fragmentation_ratio往往会大于等于1
-  mem_fragmentation_ratio<1，说明Redis使用了虚拟内存，由于虚拟内存的媒介是磁盘，比内存速度要慢很多，当这种情况出现时，应该及时排查，如果内存不足应该及时处理，如增加Redis节点、增加Redis服务器的内存、优化应用等
-  mem_fragmentation_ratio**值很大**，是因为还没有向Redis中存入数据，Redis进程本身运行的内存使得used_memory_rss 比used_memory大得多。


used_memory=实际内存+虚拟内存(内存充足的时候不适用)

used_memory_rss=实际内存+内存碎片+进程所需(很小)



### 内存消耗(内存划分)
#### 自身内存(进程所需内存)
- redis自身运行所消耗的内存，一般很小。

Redis主进程本身运行肯定需要占用内存，如代码、常量池等等；这部分内存大约几兆，在大多数生产环境中与Redis数据占用的内存相比可以忽略。这部分内存不是由jemalloc分配，因此不会统计在used_memory中。除了主进程外，Redis创建的子进程运行也会占用内存，如Redis执行AOF、RDB重写时创建的子进程。当然，这部分内存不属于Redis进程，也不会统计在used_memory和used_memory_rss中


#### 对象内存
- 这是redis消耗内存最大的一块，存储着用户所有的数据，每次存入key-value的键值对，暂时可以简单的理解为sizeof(key+value)的内存大小消耗。

#### 缓冲内存
- 缓冲内存主要包括：客户端缓冲、复制积压缓冲区、AOF缓冲区。

缓冲内存包括客户端缓冲区、复制积压缓冲区、AOF缓冲区等；其中，客户端缓冲存储客户端连接的输入输出缓冲；复制积压缓冲用于部分复制功能；AOF缓冲区用于在进行AOF重写时，保存最近的写入命令。在了解相应功能之前，不需要知道这些缓冲的细节；这部分内存由jemalloc分配，因此会统计在used_memory中。



##### 客户端缓冲
- 是指客户端连接redis之后，输入或者输出数据的缓冲区，其中输出缓冲可以通过配置参数参数client-output-buffer-limit控制。

###### 普通客户端的连接
- 普通客户端默认并没有对输出缓冲区做限制。
- 但是如果当有大量的慢连接客户端接入时，这部分消耗就不能忽略了，因为消费的很慢，在成输出缓冲区数据积压。
- 所以可以设置maxclients做限制。

###### 从客户端
- client-output-buffer-limit slave 256mb 64mb 60
- 主节点会每一个从节点单独建立一条连接用于命令复制。当主节点网络延迟较高或主节点挂载大量的从节点时，这部分内存消耗将占用很大一部分，**建议主节点挂载从节点最好不要超过2个**。

######  订阅客户端
- client-output-buffer-limit pubsub 32mb 8mb 60 
- 当生产消息的速度快于消费的速度时，输出缓冲区容易积压消息

##### 复制积压缓冲区
- 一个可重用的固定大小缓冲区用于实现部分复制功能，根据repl-backlog-size参数控制，默认1MB。对
于复制积压缓冲区整个主节点只有一个，所有的从节点共享此缓冲区，因此可以设置较大的缓冲区空间，如100MB，这部分内存投入是有价值的，可以有效避免全量复制。

##### AOF缓冲区
- 这部分空间用于在Redis重写期间保存最近的写入命令，AOF缓冲区空间消耗用户无法控制，消耗的内存取决于AOF重写时间和写入命令量，这部分空间占用通常很小。

#### 内存碎片
- 这是所有内存分配器无法避免的通病，但是可以优化。
- 内存碎片是Redis在分配、回收物理内存过程中产生的。例如，如果对数据的更改频繁，而且数据之间的大小相差很大，可能导致redis释放的空间在物理内存中并没有释放，但redis又无法有效利用，这就形成了内存碎片。内存碎片不会统计在used_memory中。

##### 配置

```
# 碎片整理总开关
# activedefrag yes
# 内存碎片达到多少的时候开启整理(小于100M 的时候忽略)
active-defrag-ignore-bytes 100mb
# Minimum percentage of fragmentation to start active defrag
# 碎片率达到百分之多少开启整理
active-defrag-threshold-lower 10
# Maximum percentage of fragmentation at which we use maximum effort
# 碎片率小余多少百分比开启整理
active-defrag-threshold-upper 100
# Minimal effort for defrag in CPU percentage
active-defrag-cycle-min 25
# Maximal effort for defrag in CPU percentage
active-defrag-cycle-max 75

```
### 时间设置指令 
- EXPIRE  指令	EXPIRE key seconds    
- SETEX key seconds value   设置超时时间
- PERSIST key    取消超时时间，将key变成持久 键 
- TTL key 查看key剩余的存活时间 

## 内存优化
### 优化内存碎片
- 尽量数据对齐，视业务情况而定
- 安全重启：重启可以做到内存碎片重新整理。
- redis4.0以上可以使用新增指令来手动回收内存碎片。

#### 内存分配器

- used_allocator参数的值是libc，表示当前redis使用的内存分配器是glibc

##### jemalloc
- 另一个redis的内存分配器jemalloc。jemalloc针对碎片化问题专门做了优化，一般不会存在过度碎片化的问题，使用jemalloc正常的碎片率（mem_fragmentation_ratio）在1.03左右
- jemalloc内存分配器是为了**更好的管理和重复利用内存**，分配内存策略一般采用**固定范围的内存块进行内存分配**。
- 简单地说jemalloc将内存空间划分为三个部分：Small class、Large class、Hugeclass，每个部分又划分为很多小的内存块单位
```
- Small class: [8byte], [16byte, 32byte, … 128byte], [192byte, 256byte, … 512byte], [768byte, 1024byte, … 3840byte] 
- Large class: [4kb, 8kb, 12kb, … 4072kb] 
- Huge class: [4mb, 8mb, 12mb …]
```

- 简单来说就是采用不同大小的内存块来存储不同大小的内存对象，比如说一个5kb的对象就会存到8kb的内存块中，那么剩下的3kb内存就变成了内存碎片，不能再被分配给其他对象。
- 通常在对key做频繁更新操作和大量过期key被删除的时候会导致碎片率上升，可以**考虑重启节点的方式重新整理碎片**

#### 内存碎片率
- 当used_memory < used_memory_rss时，说明used_memory_rss中多余出来的内存没有被用来存储对象。如果两个值的差值很大，说明碎片率很高。

- used_memory > used_memory_rss时，说明操作系统采用了内存交换，把Redis内存交换（swap）到硬盘。内存交换（swap）对于Redis来说是致命的，Redis能保证高性能的一个重要前提就是读写都基于内存。如果操作系统把Redis使用的部分内存交换到硬盘，由于硬盘的读写效率比内存低上百倍，会导致Redis性能急剧下降，特别容易引起Redis阻塞。
- 如果Redis服务器中的内存碎片已经很大，可以通过安全重启的方式减小内存碎片：因为重启之后，Redis重新从备份文件中读取数据，在内存中进行重排，为每个数据重新选择合适的内存单元，减小内存碎片。


### 不要频繁的对已存在的键做append、setrange等更新操作
- 例如一个8byte的字符串正好存储在一个8byte的内存块上，此时内存碎片率为0，如果对改字段进行了append操作，新增了一个8byte长度的字符串，那么redis会为新字符串分配一个预留的内存片，大概是16byte。
- 即新的字符串长度16byte，却用了32byte的内存来存储，碎片率到达了200%。append之后预留一倍的内存来是由redis的预分配机制来确定的，并非一定是翻倍的内存，具体可以查询redis字符串SDS的预分配机制。
- 所以对redis键的更新，最好是取出键值，删除之后再重新覆盖，这样就避免了redis内部的预分配

### 缩减键值对象
- 降低Redis内存使用最直接的方式就是缩减key和value的长度
- key的设计：越短越好，如user:{userid}:friends:notify:{fid}，可以简化为u:{uid}:fs:nt:{fid}
- value： 值对象缩减比较复杂，常见的需求是把业务对象序列化放入Redis。

## 内存管理
### 最大内存
- Redis通过maxmemory参数限制最大可用内存。限制内存目的主要有用于缓存场景，当超出内存上限maxmemory时候使用LRU等删除策略释放空间，防止所用内存超过服务器物理内存。
- 如果不限制最大内存，当内存使用完了就会大量使用Swap，导致redis 的速度很慢，从而产生大量阻塞

### 动态调整内存上限
- 可以通过命令config set maxmemory8GB

### 内存回收策略
#### 惰性删除
- Redis采用惰性删除和定时任务删除机制实现过期key的内存回收
- 用于当客户端读取带有超时属性的key的时候，**如果已经超过设置的过期时间，会执行删除操作并返回空**。但是有一个问题，**当过期键一直没有访问将无法得到及时删除，从而导致内存不能及时释放**

#### 定时任务删除
- Redis内部维护一个定时任务，默认每秒运行10次，通过配置hz属性控制
-  每秒10次的执行如下操作：随机选取100个key校验是否过期，如果有25个以上的key过期了，立刻额外随机选取下100个key(不计算在10次之内)。可见，如果过期的key不多，它最多每秒回收200条左右，如果有超过25%的key过期了，它就会做得更多，但只要key不被主动get，它占用的内存什么时候最终被清理掉只有天知道。

##### 分散过期时间
- 为了保证过期扫描不会出现循环过度，导致线程卡死现象，算法还增加了扫描时间的上限，默认不会超过 25ms
- 扫描期间客户端请求可能发生阻塞，慢查询无法追溯（慢查询指逻辑处理慢，不包含等待时间）
- 开发人员一定要注意过期时间，如果有大批量的 key 过期，要给过期时间设置一个随机范围，而不宜全部在同一时间过期，分散过期处理的压力。

##### 从库的过期策略
- 过期删除操作仅在master实例上执行; 
- 从库不会进行过期扫描，从库对过期的处理是被动的。主库在 key 到期时，会在 AOF 文件里增加一条 del 指令，同步到所有的从库，从库通过执行这条 del 指令来删除过期的 key。
- 因为指令同步是异步进行的，所以主库过期的 key 的 del 指令没有及时同步到从库的话，会出现主从数据的不一致，主库没有的数据在从库里还存在


### 内存溢出的控制策略
- 当Redis所用内存达到maxmeory上限时，会触发相应的溢出控制策略。
- 即按照配置的Policy进行处理，默认策略为volatile-lru，对设置了expire time的key进行LRU清除(不是按实际expiretime)。
- 如果沒有数据设置了expiretime或者policy为noeviction，则直接报错，但此时系统仍支持get之类的读操作。
- 具体策略受maxmeory-policy参数控制，Redis支持6种策略：
- 后面的 lru、ttl 以及 random 是三种不同的淘汰策略，再加上一种 no-enviction 永不回收的策略

```
volatile-lru ：根据LRU算法删除设置了超时属性的键，直到腾出足够空间为止

 allkeys-lru： 根据LRU算法删除键，不管有没有设置超时属性，直到腾出足够空间为止

volatile-random： 随即删除过期键，直到腾出足够空间为止

allkeys-random ：随即删除所有键，直到腾出足够空间为止

volatile-ttl：根据ttl属性，删除最近将要过期的数据，如果没有回退到noeviction策略

noeviction： 不会删除任何数据，拒绝所有写入操作，并返回错误信息，此时只是响应读
```
- 当使用volatile-lru、volatile-random、volatile-ttl这三种策略时，如果没有key可以被淘汰，则和noeviction一样返回错误 
- LRU 其核心思想是：如果一个数据在最近一段时间没有被用到，那么将来被使用到的可能性也很小，所以就可以被淘汰掉。

####  allkeys-lru
- allkeys-lru：如果我们的应用对缓存的访问符合幂律分布（也就是存在相对热点数据），或者我们不太清楚我们应用的缓存访问分布状况，我们可以选择allkeys-lru策略。

#### allkeys-random
- allkeys-random：如果我们的应用对于缓存key的访问概率相等，则可以使用这个策略。

#### volatile-ttl
- volatile-ttl：这种策略使得我们可以向Redis提示哪些key更适合被eviction。

## LRU在Redis中的实现
- 近似LRU算法 Redis使用的是近似LRU算法，它跟常规的LRU算法还不太一样。近似LRU算法通过随机采样法淘汰数据，每次随机出5（默认）个key，从里面淘汰掉最近最少使用的key。
- 可以通过maxmemory-samples参数修改采样数量：例：maxmemory-samples 10 **maxmenory-samples配置的越大，淘汰的结果越接近于严格的LRU算法**
- Redis为了实现近似LRU算法，给每个key增加了一个额外增加了一个24bit的字段，用来存储该key最后一次被访问的时间。
- Redis3.0对近似LRU的优化 Redis3.0对近似LRU算法进行了一些优化。新算法会维护一个候选池（大小为16），池中的数据根据访问时间进行排序，第一次随机选取的key都会放入池中，随后每次随机选取的key只有在访问时间小于池中最小的时间才会放入池中，直到候选池被放满。当放满后，如果有新的key需要放入，则将池中最后访问时间最大（最近被访问）的移除。



## 内存分配器

- Redis在编译时便会指定内存分配器；内存分配器可以是 libc 、jemalloc或者tcmalloc，默认是jemalloc。

### jemalloc

jemalloc作为Redis的默认内存分配器，在减小内存碎片方面做的相对比较好。jemalloc在64位系统中，将内存空间划分为小、大、巨大三个范围；每个范围内又划分了许多小的内存块单位；当Redis存储数据时，会选择大小最合适的内存块进行存储。

![image-20201125215145100](/Users/liuwenqiang/Library/Application%20Support/typora-user-images/image-20201125215145100.png)