[toc]



## 数据过滤
### 行过滤
- 行处理：在分区剪裁中，**当使用外关联时，如果将副表的过滤条件写在Where后面，那么就会先全表关联，之后再过滤**，

### 列过滤 
- 列处理：在SELECT中，只拿需要的列，如果有，**尽量使用分区过滤，少用SELECT ***。

## map 个数

input目录的文件总个数，input的文件大小，集群设置的文件块大小(目前为128M, 可在hive中通过set dfs.block.size;命令查看到，该参数不能自定义修改)；

如果一个任务有很多小文件（远远小于块大小128m），则每个小文件也会被当做一个块，用一个map任务来完成，而一个map任务启动和初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费。而且，同时可执行的map数是受限的

是不是保证每个map处理接近128m的文件块，就高枕无忧了，答案也是不一定。比如有一个127m的文件，正常会用一个map去完成，但这个文件只有一个或者两个小字段，却有几千万的记录，如果map处理的逻辑比较复杂，用一个map任务去做，肯定也比较耗时

### map 个数计算公式
在 MapReduce 的编程案例中，我们得知，一个MR Job的 MapTask 数量是由输入分片 InputSplit 决定的。而输入分片是由 FileInputFormat.getSplit()决定的。一个输入分片对应一个 MapTask， 而输入分片是由三个参数决定的：
![image-20201206211403249](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:14:04-image-20201206211403249.png)

long splitSize = Math.max(minSize, Math.min(maxSize, blockSize))

默认情况下，输入分片大小和 HDFS集群默认数据块大小一致，也就是默认一个数据块，启用一个MapTask进行处理，这样做的好处是避免了服务器节点之间的数据传输，提高 job 处理效率。

### 例子
1. 假设input目录下有1个文件a,大小为780M,那么hadoop会将该文件a分隔成7个块（6个128m的块和1个12m的块），从而产生7个map数
2. 假设input目录下有3个文件a,b,c,大小分别为10m，20m，130m，那么hadoop会分隔成4个块（10m,20m,128m,2m）,从而产生4个map数
即，如果文件大于块大小(128m),那么会拆分，如果小于块大小，则把该文件当成一个块。

### 优化
#### 小文件太多
```
set mapred.max.split.size=100000000; -- 每个Map最大输入大小，决定合并后的文件数
set mapred.min.split.size.per.node=100000000;
set mapred.min.split.size.per.rack=100000000;
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
```
- set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;这个参数表示执行前进行小文件合并，
- 前面三个参数确定合并文件块的大小，大于文件块大小128m的，按照128m来分隔，小于128m,大于100m的，按照100m来分隔，把那些小于100m的（包括小文件和分隔大文件剩下的），进行合并

```
配置Map输入合并
-- 每个Map最大输入大小，决定合并后的文件数
set mapred.max.split.size=256000000;
-- 一个节点上split的至少的大小 ，决定了多个data node上的文件是否需要合并
set mapred.min.split.size.per.node=100000000;
-- 一个交换机下split的至少的大小，决定了多个交换机上的文件是否需要合并
set mapred.min.split.size.per.rack=100000000;
-- 执行Map前进行小文件合并； 在map执行前合并小文件，减少map数：CombineHiveInputFormat具有对小文件进行合并的功能（系统默认的格式）。HiveInputFormat没有对小文件合并功能。
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; 
 
配置Hive结果合并
我们可以通过一些配置项来使Hive在执行结束后对结果文件进行合并：
set hive.merge.mapfiles=true 在map-only job后合并文件，默认true，在map-only任务结束时合并小文件
set hive.merge.mapredfiles=true 在map-reduce job后合并文件，默认false，设为true  在map-reduce任务结束时合并小文件
set hive.merge.size.per.task=true 合并后每个文件的大小，默认268435456，合并文件的大小，默认256M
set hive.merge.smallfiles.avgsize=true 平均文件大小，是决定是否执行合并操作的阈值，默认16777216，当输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件merge

Hive在对结果文件进行合并时会执行一个额外的map-only脚本，mapper的数量是文件总大小除以size.per.task参数所得的值，触发合并的条件是：
　　根据查询类型不同，相应的mapfiles/mapredfiles参数需要打开；
　　结果文件的平均大小需要大于avgsize参数的值。
```

#### 增加map个数
- 当input的文件都很大，任务逻辑复杂，map执行非常慢的时候，可以考虑增加Map数，来使得每个map处理的数据量减少，从而提高任务的执行效率。
- 增加map的方法为：根据computeSliteSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=128M公式，调整maxSize最大值。让maxSize最大值低于blocksize就可以增加map的个数。
```
1．执行查询
hive (default)> select count(*) from emp;
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1

2．设置最大切片值为100个字节
hive (default)> set mapreduce.input.fileinputformat.split.maxsize=100;
hive (default)> select count(*) from emp;
Hadoop job information for Stage-1: number of mappers: 6; number of reducers: 1
```


### 结论
- 如果一个任务有很多小文件（远远小于块大小128m）,则每个小文件也会被当做一个块，用一个map任务来完成，
而一个map任务启动和初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费。而且，同时可执行的map数是受限的。
- 是不是保证每个map处理接近128m的文件块，就高枕无忧了？ 
答案也是不一定。比如有一个127m的文件，正常会用一个map去完成，但这个文件只有一个或者两个小字段，却有几千万的记录，
如果map处理的逻辑比较复杂，用一个map任务去做，肯定也比较耗时


## reduce 个数
- Reduce的个数对整个作业的运行性能有很大影响。如果Reduce设置的过大，那么将会产生很多小文件，对NameNode会产生一定的影响，而且整个作业的运行时间未必会减少；如果Reduce设置的过小，那么单个Reduce处理的数据将会加大，很可能会引起OOM异常
- reduce个数的设定极大影响任务执行效率，不指定reduce个数的情况下，Hive会猜测确定一个reduce个数，基于以下两个设定：
```
hive.exec.reducers.bytes.per.reducer（每个reduce任务处理的数据量，默认为1000^3=1G）
hive.exec.reducers.max（每个任务最大的reduce数，默认为999）
```

### reduce 计算公式
- 计算reducer数的公式很简单N=min(参数2，总输入数据量/参数1)即，如果reduce的输入（map的输出）总大小不超过1G,那么只会有一个

### 调整reduce个数方法
- 设置hive.exec.reducers.bytes.per.reducer=256000000
- set mapreduce.job.reduces = 15;

### 例子
- select pt,count(1) from popt_tbaccountcopy_mes where pt = '2012-07-04' group by pt; 
```
/group/p_sdo_data/p_sdo_data_etl/pt/popt_tbaccountcopy_mes/pt=2012-07-04 总大小为9G多，因此这句有10个reduce
```
### 结论
- reduce个数并不是越多越好；同map一样，启动和初始化reduce也会消耗时间和资源；
- 另外，有多少个reduce,就会有多少个输出文件，如果生成了很多个小文件，那么如果这些小文件作为下一个任务的输入，则也会出现小文件过多的问题


### 什么情况下只有一个reduce；
- 很多时候你会发现任务中不管数据量多大，不管你有没有设置调整reduce个数的参数，任务中一直都只有一个reduce任务；其实只有一个reduce任务的情况，除了数据量小于hive.exec.reducers.bytes.per.reducer参数值的情况外，还有以下原因：
#### 没有group by的汇总
- 比如把select pt,count(1) from popt_tbaccountcopy_mes where pt = '2012-07-04' group by pt; 写成 select count(1) from popt_tbaccountcopy_mes where pt = '2012-07-04';
这点非常常见，希望大家尽量改写。
#### 用了Order by
#### 有笛卡尔积
```
通常这些情况下，除了找办法来变通和避免，我暂时没有什么好的办法，因为这些操作都是全局的，所以hadoop不得不用一个reduce去完成；
```
```
同样的，在设置reduce个数的时候也需要考虑这两个原则：使大数据量利用合适的reduce数；使单个reduce任务处理合适的数据量；
```

## limit 优化
- 一般情况下，Limit语句还是需要执行整个查询语句，然后再返回部分结果。有一个配置属性可以开启，避免这种情况---对数据源进行抽样
    ```
    hive.limit.optimize.enable=true --- 开启对数据源进行采样的功能
    hive.limit.row.max.size --- 设置最小的采样容量
    hive.limit.optimize.limit.file --- 设置最大的采样样本数
    ```
- 缺点：有可能部分数据永远不会被处理到

## JOIN优化

### 将大表放后面(Map join)
- Hive假定查询中最后的一个表是大表。它会将其它表缓存起来，然后扫描最后那个表。
因此通常需要将小表放前面，或者标记哪张表是大表：/*streamtable(table_name) */
- hive.auto.convert.join=true,之后我们不必太关心表的顺序，hive 会自动帮我们优化
    - 可以 set hive.auto.convert.join = false; 进行效果测试    
- 一张表多大会被hive认为是小表呢？默认的是25000000字节，用户也可以通过下面参数自己设置
```
<property>
  <name>hive.mapjoin.smalltable.filesize</name>
  <value>25000000</value>
  <description>The threshold for the input file size of the small tables; if the file size is smaller than this threshold, it will try to convert the common join into map join</description>
</property>
```

#### map join 理论
![image-20201206211458408](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:14:59-image-20201206211458408.png)
- 首先是Task A，它是一个LocalTask（在客户端本地执行的Task），负责扫描小表b的数据，将其转换成一个HashTable的数据结构，并写入本地的文件中，之后将该文件加载到DistributeCache中。
- 接下来是Task B，该任务是一个没有Reduce的MR，启动MapTasks扫描大表a,在Map阶段，根据a的每一条记录去和DistributeCache中b表对应的HashTable关联，并直接输出结果。
- 由于MapJoin没有Reduce，**所以由Map直接输出结果文件，有多少个Map Task，就有多少个结果文件**

### 使用相同的连接键
- 当对3个或者更多个表进行join连接时，如果每个on子句都使用相同的连接键的话，那么只会产生一个MapReduce job。

### SMB join（sort merger bucket）
- A桶个数必须与B桶的个数相同，或者B桶的个数是A桶的个数的倍数
- 设置：hive.auto.convert.sortmerge.join=true


### 笛卡尔积
- 尽量避免笛卡尔积，join的时候不加on条件，或者无效的on条件，Hive只能使用1个reducer来完成笛卡尔积
- 当 Hive 设定为严格模式（hive.mapred.mode=strict）时，不允许在 HQL 语句中出现笛卡尔积,这实际说明了 Hive对笛卡尔积支持较弱。因为找不到 Join key，Hive 只能使用 1 个 reducer 来完成笛卡尔积。
- 当然也可以使用 limit 的办法来减少某个表参与 join 的数据量，但对于需要笛卡尔积语义的 需求来说，经常是一个大表和一个小表的Join操作，结果仍然很大（以至于无法用单机处理），这时MapJoin才是最好的解决办法。
- MapJoin，顾名思义，会在 Map 端完成 Join 操作。 这需要将 Join 操作的一个或多个表完全读入内存。
    - MapJoin 在子查询中可能出现未知 BUG。
- 在大表和小表做笛卡尔积时，规避笛卡尔积的 方法是，给 Join 添加一个 Join key，原理很简单：将小表扩充一列 join key，并将小表的条 目复制数倍，join key 各不相同；将大表扩充一列 join key 为随机数。
-  精髓就在于复制几倍，最后就有几个 reduce 来做，而且大表的数据是前面小表扩张 key值 范围里面随机出来的，所以复制了几倍n，就相当于这个随机范围就有多大n，那么相应的，大表的数据就被随机的分为了n份。并且最后处理所用的 reduce 数量也是 n，而且也不会 出现数据倾斜。


### 尽量尽早地过滤数据
- 减少每个阶段的数据量,对于分区表要加分区，同时只选择需要使用到的字段。
> join 优化前
select m.cid,u.id from order m join customer u on m.cid=u.id where m.dt='2013-12-12';

> join优化后
select m.cid,u.id from (select cid from order where dt='2013-12-12')m join customer u on m.cid=u.id;

### 尽量原子化操作
- 尽量避免一个SQL包含复杂逻辑，可以使用中间表来完成复杂的逻辑


## 优化 in/exists 语句
- 虽然经过测验，hive1.2.1 也支持 in/exists 操作，但还是推荐使用 hive 的一个高效替代方案：**left semi join**

### 例子
- select a.id, a.name from a where a.id in (select b.id from b);
- select a.id, a.name from a where exists (select id from b where a.id = b.id);
> 应该转换成：
select a.id, a.name from a left semi join b on a.id = b.id;

## 排序选择
- cluster by：对同一字段分桶并排序，不能和 sort by 连用
- distribute by + sort by：分桶，保证同一字段值只存在一个结果文件当中，结合 sort by 保证 每个 reduceTask 结果有序
- sort by：单机排序，单个 reduce 结果有序
- order by：全局排序，缺陷是只能使用一个 reduce 

## 合并 MapReduce操作
- Multi-group by 是 Hive 的一个非常好的特性，它使得 Hive 中利用中间结果变得非常方便。 例如：
```
FROM (SELECT a.status, b.school, b.gender FROM status_updates a JOIN profiles b ON (a.userid =
b.userid and a.ds='2009-03-20' ) ) subq1
INSERT OVERWRITE TABLE gender_summary PARTITION(ds='2009-03-20')
SELECT subq1.gender, COUNT(1) GROUP BY subq1.gender
INSERT OVERWRITE TABLE school_summary PARTITION(ds='2009-03-20')
SELECT subq1.school, COUNT(1) GROUP BY subq1.school

```
## 合理利用分桶：Bucketing 和 Sampling
- Bucket 是指将数据以指定列的值为 key 进行 hash，hash 到指定数目的桶中。这样就可以支持高效采样了。如下例就是以 - userid 这一列为 bucket 的依据，共设置 32 个 buckets
```
CREATE TABLE page_view(viewTime INT, userid BIGINT,
 page_url STRING, referrer_url STRING,
 ip STRING COMMENT 'IP Address of the User')
 COMMENT 'This is the page view table'
 PARTITIONED BY(dt STRING, country STRING)
 CLUSTERED BY(userid) SORTED BY(viewTime) INTO 32 BUCKETS
 ROW FORMAT DELIMITED
 FIELDS TERMINATED BY '1'
 COLLECTION ITEMS TERMINATED BY '2'
 MAP KEYS TERMINATED BY '3'
 STORED AS SEQUENCEFILE;
```





## 数据倾斜
![image-20201206211530341](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:15:30-image-20201206211530341.png)
- 任务进度长时间维持在99%（或100%），查看任务监控页面，发现只有少量（1个或几个）reduce子任务未完成。因为其处理的数据量和其他reduce差异过大。
- 单一reduce的记录数与平均记录数差异过大，通常可能达到3倍甚至更多。 最长时长远大于平均时长。
- 各种container报错OOM
- 读写的数据量极大，至少远远超过其它正常的reduce
- 伴随着数据倾斜，会出现任务被kill等各种诡异的表现。

### 倾斜的原理

- 在进行shuffle的时候，必须将****各个节点上相同的Key拉取到某个节点上的一个task来进行处理****，比如按照key进行聚合或者join操作。如果某个key对应的数据量特别大的话，会发生数据倾斜。

### 原因

- key分布不均匀
- 业务数据本身的特性
- 建表时考虑不周
- 某些SQL语句本身就有数据倾斜

<table>
<thead>
<tr><th><span style="font-size: 16px;">关键词</span></th><th><span style="font-size: 16px;">情形</span></th><th><span style="font-size: 16px;">后果</span></th></tr>
</thead>
<tbody>
<tr>
<td><span style="font-size: 16px;">join</span></td>
<td><span style="font-size: 16px;">其中一个表较小，但是key集中</span></td>
<td><span style="font-size: 16px;">分发到某一个或几个Reduce上的数据远高于平均值</span></td>
</tr>
<tr>
<td><span style="font-size: 16px;">join</span></td>
<td><span style="font-size: 16px;">大表与大表，但是分桶的判断字段0值或空值过多</span></td>
<td><span style="font-size: 16px;">这些空值都由一个reduce处理，灰常慢</span></td>
</tr>
<tr>
<td><span style="font-size: 16px;">group by</span></td>
<td><span style="font-size: 16px;">group by 维度过小，某值的数量过多</span></td>
<td><span style="font-size: 16px;">处理某值的reduce灰常耗时</span></td>
</tr>
<tr>
<td><span style="font-size: 16px;">count distinct</span></td>
<td><span style="font-size: 16px;">某特殊值过多</span></td>
<td><span style="font-size: 16px;">处理此特殊值reduce耗时</span></td>
</tr>
</tbody>
</table>

- 决定是否可以在 Map 端进行聚合操作 set hive.map.aggr=true; 
- 有数据倾斜的时候进行负载均衡 set hive.groupby.skewindata=true; 


### 空值产生的数据倾斜
#### 空值关联
##### 方案1 空值不参与关联
```
select * from log a join user b on a.user_id is not null and a.user_id = b.user_id
union all
select * from log c where c.user_id is null;
```

```
insert overwrite table jointable select n.* from nullidtable n left join ori o on o.id=n.id;
Time taken: 52.863 seconds
测试过滤空id
insert overwrite table jointable select n.* from (select * from nullidtable where id is not null) n left join ori o on n.id=o.id;
Time taken: 30.157 seconds
```
##### 方案2 赋予空值新值
- 有时虽然某个key为空对应的数据很多，但是相应的数据不是异常数据，必须要包含在join的结果中，此时我们可以表a中key为空的字段赋一个随机的值，使得数据随机均匀地分不到不同的reducer上。
- set mapreduce.job.reduces=-1 默认是-1，Hive会根据输入文件的大小估算出Reduce的个数。根据输入文件估算Reduce的个数可能未必很准确，因为Reduce的输入是Map的输出，而Map的输出可能会比输入要小，所以最准确的数根据Map的输出估算Reduce的个数。

```
不随机分布空null值：

hive (default)> set mapreduce.job.reduces=5; 
hive (default)> insert overwrite table jointable select n.* from nullidtable n left join ori o on o.id=n.id;
Time taken: 38.521 seconds
结果可以看出来，出现了数据倾斜，某些reducer的资源消耗远大于其他reducer。
```
![image-20201206211602242](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:16:02-image-20201206211602242.png)

```
随机分布空null值
hive (default)> insert overwrite table jointable select n.* from nullidtable n full join ori o on case when n.id is null then concat('hive', rand()) else n.id end = o.id;
Time taken: 55.743 seconds
结果可以看出来，消除了数据倾斜，负载均衡reducer的资源消耗
```
![image-20201206211624291](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:16:24-image-20201206211624291.png)

或者
```
select * from log a left outer join user b on
case when a.user_id is null then concat('hive',rand()) else a.user_id end = b.user_id
```
- 使本身为 null 的所有记录不会拥挤在同一个 reduceTask 了，会由于有替代的 随机字符串值，而分散到了多个 reduceTask 中了，由于 null 值关联不上，处理后并不影响最终结果。


#### 空值 distinct
- count distinct大量相同特殊值：count distinct时，将值为空的情况单独处理，如果是计算count distinct，**可以不用处理，直接过滤，在做后结果中加1**。如果还有其他计算，需要进行group by，可以先将值为空的记录单独处理，再和其他计算结果进行union.


### 不同数据类型关联产生数据倾斜
- 用户表中 user_id 字段为 int，log 表中 user_id 为既有 string 也有 int 的类型， 当按照两个表的 user_id 进行 join 操作的时候，默认的 hash 操作会按照 int 类型的 id 进 行分配，这样就会导致所有的 string 类型的 id 就被分到同一个 reducer 当中

#### 解决方案
```
select * from user a left outer join log b on b.user_id = cast(a.user_id as string)
```

### 大小表关联查询产生数据倾斜 
- 就是在Map阶段进行表之间的连接,而不需要进入到Reduce阶段才进行连接,这样就节省了在Shuffle阶段时要进行的大量数据传输,以及传输到某个节点上的数据量过大而导致的倾斜。从而起到了优化作业的作用。
- 将其中做连接的小表（全量数据）分发到所有MapTask端进行Join，从而避免了reduceTask，前提**要求是内存足以装下该全量数据**

#### mapjoin

![image-20201206211651406](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:16:52-image-20201206211651406.png)
- 以大表 a 和小表 b 为例，所有的 maptask 节点都装载小表 b 的所有数据，然后大表 a 的 一个数据块数据比如说是 a1 去跟 b 全量数据做链接，就省去了 reduce 做汇总的过程。 所以相对来说，在内存允许的条件下使用 map join 比直接使用 MapReduce 效率还高些， 当然这只限于做 join 查询的时候。

##### 写法1
- 在 hive 中，直接提供了能够在 HQL 语句指定该次查询使用 map join，map join 的用法是 在查询/子查询的SELECT关键字后面添加/*+ MAPJOIN(tablelist) */提示优化器转化为map join（早期的 Hive 版本的优化器是不能自动优化 map join 的）。其中 tablelist 可以是一个 表，或以逗号连接的表的列表。tablelist 中的表将会读入内存，通常应该是将小表写在这里。

```
select /* +mapjoin(a) */ a.id aid, name, age from a join b on a.id = b.id;
select /* +mapjoin(movies) */ a.title, b.rating from movies a join ratings b on a.movieid =
b.movieid;
```
##### 写法2
-  hive0.11 版本以后会自动开启 map join 优化，由两个参数控制
```
- set hive.auto.convert.join=true; //设置 MapJoin 优化自动开启
- set hive.mapjoin.smalltable.filesize=25000000 //设置小表不超过多大时开启 mapjoin 优化，即25M
```
##### 总计
- 普通的join是会走shuffle过程的，而一旦shuffle，就相当于会将相同key的数据拉取到一个shuffle read task中再进行join，此时就是reduce join。但是如果一个RDD是比较小的，则可以采用广播小RDD全量数据+map算子来实现与join同样的效果，也就是mao join ，而此时不会发生shuffle操作，也就不会发生数据倾斜。
- 对join操作导致的数据倾斜，效果非常好，因为根本就不会发生shuffle，也就根本不会发生数据倾斜。

### Count(Distinct) 
- 数据量小的时候无所谓，数据量大的情况下，由于COUNTDISTINCT操作需要用一个ReduceTask来完成，这一个Reduce需要处理的数据量太大，就会导致整个Job很难完成，一般COUNT DISTINCT使用先GROUP BY再COUNT的方式替换
> distinct会将b列所有的数据保存到内存中，形成一个类似hash的结构，速度是十分的块；但是在大数据背景下，因为b列所有的值都会形成以key值，**极有可能发生OOM**

```
select count(distinct user) from some_table;
由于必须去重，因此Hive将会把Map阶段的输出全部分布到一个ReduceTask上，
此时很容易引起性能问题，对于这种情况，可以通过先group by再count的方式优化
select  count(*) from (select user from some_table group by user) temp;
```

```
SELECT day, COUNT(DISTINCT user_id)
FROM T
GROUP BY day
优化之后
select
	day,COUNT( user_id)
from(
	SELECT day, user_id
	FROM T
	GROUP BY day,user_id
)
group by
	day
-- 但是上面的优化可能存在reduce过多的问题，或者分组太多的问题
SELECT day, SUM(cnt)
FROM (
    SELECT day, COUNT(DISTINCT user_id) as cnt
    FROM T
    GROUP BY day, MOD(HASH_CODE(user_id), 1024)
)
GROUP BY day
-- 如果不好理解的话，可以改造一下
SELECT day, SUM(cnt)
FROM (
    SELECT day,MOD(HASH_CODE(user_id), 1024) as key ,COUNT(DISTINCT user_id) as cnt
    FROM T
    GROUP BY day, MOD(HASH_CODE(user_id), 1024)
)
GROUP BY day
```
> 由于COUNT DISTINCT的全聚合操作，即使设定了reduce task个数，set mapred.reduce.tasks=100；hive也只会启动一个reducer 


### Group By
> group by引起的倾斜主要是输入数据行按照groupby列分别布均匀引起的，比如，假设按照供应商对销售明细事实表来统计订单数，那么部分大供应商的订单量显然非常大，而多数供应商的订单量就一般，

- 默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜了。**并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果**。

#### 参数优化设置

- 是否在Map端进行聚合，默认为True , set hive.map.aggr = true
- 在Map端进行聚合操作的条目数目 set hive.groupby.mapaggr.checkinterval = 100000
- 有数据倾斜的时候进行负载均衡（默认是false）set hive.groupby.skewindata = true

```
当选项设定为 true，生成的查询计划会有两个MRJob。第一个MRJob中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；
第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。
```
#### 聚合——两阶段聚合
- 这个方案的核心实现思路就是进行两阶段聚合。第一次是局部聚合，先给每个key都打上一个随机数，比如10以内的随机数，此时原先一样的key就变成不一样的了，比如(hello, 1) (hello, 1) (hello, 1) (hello, 1)，就会变成(1_hello, 1) (1_hello, 1) (2_hello, 1) (2_hello, 1)。接着对打上随机数后的数据，执行reduceByKey等聚合操作，进行局部聚合，那么局部聚合结果，就会变成了(1_hello, 2) (2_hello, 2)。然后将各个key的前缀给去掉，就会变成(hello,2)(hello,2)，再次进行全局聚合操作，就可以得到最终结果了，比如(hello, 4)

![image-20201206211718182](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:17:18-image-20201206211718182.png)

#### 例子
- 原始sql 
```
select behavior,sum(1) as cnt from user_behavior group by behavior order by cnt desc;pv 215662

pv	434349
cart 25824
fav	14396
buy	11161
```
- 改写之后的
```
select
    substr(behavior,3),
    sum(cnt) as cnt
from(
    select
        behavior,
        sum(1) as cnt
    from (
        select
            concat(floor(rand()*10),'_',behavior) as behavior ,
            user_id
        from
           user_behavior
    ) a
    group by
        behavior
)a
group by
    substr(behavior,3)
order by
    cnt desc


pv	434349
cart 25824
fav	14396
buy	11161
```

### 最后的大招(分开处理)
- 将倾斜的key 单独输出到一张表，然后对这些key 进行前缀随机N，然后将被关联的key,随机重复N倍，然后关联处理这些倾斜的key

- 对包含少数几个数据量过大的key的那个表，通过sample算子采样出一份样本来，然后统计一下每个key的数量，计算出来数据量最大的是哪几个key。
- 然后将这几个key对应的数据从原来的表中拆分出来，形成一个单独的表，并给每个key都打上n以内的随机数作为前缀，而不会导致倾斜的大部分key形成另外一个表
- 接着将需要join的另一个表，也过滤出来那几个倾斜key对应的数据并形成一个单独的表，将每条数据膨胀成n条数据，这n条数据都按顺序附加一个0~n的前缀，不会导致倾斜的大部分key也形成另外一个表。
- 再将附加了随机前缀的独立表与另一个膨胀n倍的独立表进行join，此时就可以将原先相同的key打散成n份，分散到多个task中去进行join了。
- 而另外两个普通的表就照常join即可。
最后将两次join的结果使用union算子合并起来即可，就是最终的join结果。

### 开启hive的优化参数
- hive.map.aggr=true和hive.groupby.skewindata=true
#### 原理

##### hive.map.aggr=true
- 开启在map 端的聚合,这个参数默认为true
> hive.groupby.mapaggr.checkinterval =100000 在 Map 端进行聚合操作的条目数目

> 在hive 中叫在map端聚合，在mr 中叫combiner,在flink 中叫localglobal

##### hive.groupby.skewindata=true
> 有数据倾斜的时候进行负载均衡，当选项设定为true,生成的查询计划会有两个MR Job。第一个MR Job中，Map的输出结果集合会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MR Job在根据预处理的数据结果按照 Group By Key 分布到Reduce中(这个过程可以保证相同的 Group By Key 被分布到同一个Reduce中)，最后完成最终的聚合操作。

- 当hive.groupby.skewindata=true时，hive不支持多列上的去重操作，并报错：
```
hive.groupby.skewindata=true
Error in semantic analysis: DISTINCT on different columns notsupported with skew in data.
```

##### 对有特殊值的数据倾斜处理 
- SET mapred.reduce.tasks=20; 
- SET hive.map.aggr=TRUE; 
- SET hive.groupby.skewindata=TRUE; 
- SET hive.optimize.skewjoin=TRUE;   


### 过滤产生倾斜的key
- 果发现导致倾斜的key就少数几个，而且对计算本身的影响并不大的话，那么很适合使用这种方案。比如99%的key就对应10条数据，但是只有一个key对应了100万数据，从而导致了数据倾斜。

#### 实现思路
- 如果我们判断那少数几个数据量特别多的key，对作业的执行和计算结果不是特别重要的话，那么干脆就直接过滤掉那少数几个key。比如，在Spark SQL中可以使用where子句过滤掉这些key或者在Spark Core中对RDD执行filter算子过滤掉这些key。如果需要每次作业执行时，动态判定哪些key的数据量最多然后再进行过滤，那么可以使用sample算子对RDD进行采样，然后计算出每个key的数量，取数据量最多的key过滤掉即可。

#### 实践经验
- 在项目中我们也采用过这种方案解决数据倾斜。有一次发现某一天Spark作业在运行的时候突然OOM了，追查之后发现，是Hive表中的某一个key在那天数据异常，导致数据量暴增。因此就采取每次执行前先进行采样，计算出样本中数据量最大的几个key之后，直接在程序中将那些key给过滤掉。
## HIVE 不常见的参数优化

### 推测执行
- 目的：是通过加快获取单个task的结果以及进行侦测将执行慢的TaskTracker加入到黑名单的方式来提高整体的任务执行效率

```
推测执行(Speculative Execution)是指在集群环境下运行MapReduce，可能是程序Bug，负载不均或者其他的一些问题，导致在一个JOB下的多个TASK速度不一致，比如有的任务已经完成，但是有些任务可能只跑了10%，根据木桶原理，这些任务将成为整个JOB的短板
如果集群启动了推测执行，这时为了最大限度的提高短板，Hadoop会为该task**启动备份任务**，让speculative task与原始task同时处理一份数据，哪个先运行完，则将谁的结果作为最终结果，并且在运行完成后Kill掉另外一个任务。
```
#### 总结
- 其实对于集群内有不同性能的机器开启这个功能是比较有用的，但是如果仅仅是数据倾斜问题可能用处就不是很大，因为即使换了机器执行，它的本质问题–数据量大并未解决，所以也有可能会执行很长的一段时间
- 推测执行(SpeculativeExecution)是通过利用更多的资源来换取时间的一种优化策略，但是在资源很紧张的情况下，推测执行也不一定能带来时间上的优化
- 所以是否启用推测执行，如果能根据资源情况来决定，如果在资源本身就不够的情况下，还要跑推测执行的任务，这样会导致后续启动的任务无法获取到资源，以导致无法执行。

### strict模式
- 对分区表进行查询，在where子句中没有加分区过滤的话，将禁止提交任务(默认：nonstrict)
set hive.mapred.mode=strict;

#### 禁止下面5种类型的查询
- 对于分区表，不加分区字段过滤条件，不能执行
- 对于order by语句，必须使用limit语句。
- 限制笛卡尔积的查询（join的时候不使用on，而使用where的）
- bigint 数据类型和string 类型的比较
- bigint 数据类型和double 类型的比较


### 并行执行
- hive会将一个查询转化为一个或多个阶段，包括：MapReduce阶段、抽样阶段、合并阶段、limit阶段等。默认情况下，一次只执行一个阶段。不过，如果某些阶段不是互相依赖，是可以并行执行的。这样可能使得整个job的执行时间缩短。不过，如果有更多的阶段可以并行执行，那么job可能就越快完成。
- set hive.exec.parallel=true,可以开启并发执行。
- set hive.exec.parallel.thread.number=16;//同一个sql允许最大并行度，默认为8。会比较耗系统资源。

#### 执行计划
- 根据执行计划，可以画出DAG,然后从DAG 中可以方便的看到，到底哪些stage 是可以并行执行的
![image-20201206211745590](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:17:46-image-20201206211745590.png)

#### map 和 reduce 的日志输出
- 理论上reduce 需要等到 所有的map 完成之后才能开始的 

#### 扩展
- 对于比较重要的任务开启并行执行，保证它的按时产出 

### JVM重用

```
众所周知，hadoop生态系统底层是java开发的，也就是说它基于JVM运行，因此我们可以设置JVM重用来优化hadoop和hive。
hadoop默认配置是派生JVM来执行map和reduce任务的，
这时JVM的启动可能会造成很大的开销，尤其是执行的job包含上千个task的时候，
JVM重用可以使得JVM实例在同一个job中重新使用N次。
N的值可以在Hadoop的mapred-site.xml文件中进行配置。通常在10-20之间，具体多少需要根据具体业务场景测试得出
```

- 用于避免小文件的场景或者task特别多的场景，这类场景大多数执行时间都很短，因为hive调起mapreduce任务，JVM的启动过程会造成很大的开销，尤其是job有成千上万个task任务时，JVM重用可以使得JVM实例在同一个job中重新使用N次
- set mapred.job.reuse.jvm.num.tasks=10; --10为重用个数
- 这个参数有一个缺点，开启JVM重用将会一直占用使用到的task插槽，以便进行重用，直到执行结束才会释放，如果某个job中有几个task执行的时间比其他task时间多很多，那么它们会一直保留插槽让其空闲无法被其他job使用，因此浪费了资源。

### 不足
- 这个功能的缺点是，开启JVM重用将一直占用使用到的task插槽，以便进行重用，直到任务完成后才能释放。
- 如果某个"不平衡的"job中有某几个reducetask执行的时间要比其他Reducetask消耗的时间多的多的话，那么保留的插槽就会一直空闲着却无法被其他的job使用，直到所有的task都结束了才会释放

### 本地模式
```
Hive 在集群上查询时，默认是在集群上N台机器上运行，需要多个机器进行协调运行，这个方式很好地解决了大数据量的查询问题。
但是当 Hive 查询处理的数据量比较小时，其实没有必要启动分布式模式去执行，因为以分布式方式执行就涉及到跨网络传输、多节点协调 等，并且消耗资源。这个时间可以只使用本地模式来执行 mapreduce job，只在一台机器上执行，速度会很快
```
- 有时hive的输入数据量是非常小的。在这种情况下，为查询出发执行任务的时间消耗可能会比实际job的执行时间要多的多。对于大多数这种情况，hive可以通过本地模式在单台机器上处理所有的任务。对于小数据集，执行时间会明显被缩短

- set hive.exec.mode.local.auto=true;
- 当一个job满足如下条件才能真正使用本地模式：
```
1.job的输入数据大小必须小于参数：hive.exec.mode.local.auto.inputbytes.max(默认128MB)
2.job的map数必须小于参数：hive.exec.mode.local.auto.tasks.max(默认4) 这个其实就是限制了文件数目个数
3.job的reduce数必须为0或者1
```
- 可用参数hive.mapred.local.mem(默认0)控制child jvm使用的最大内存数。 

### Fetch 避免一些不必要的MR作业

```
hive 0.10.0为了执行效率考虑，简单的查询，就是只是select，不带count,sum,groupby这样的，都不走map/reduce，直接读取hdfs文件进行filter过滤。
这样做的好处就是不新开mr任务，执行效率要提高不少，但是不好的地方就是用户界面不友好，有时候数据量大还是要等很长时间，但是又没有任何返回。
```
- 前面提到过在hive中有些查询会转换为MR作业，但有的不会，在hive中，我们可以进行优化，让一些语句避免执行MR作业，从而加快效率，这些参数就是hive.fetch.task.conversion，在hive-site.xml中可以设置下面属性：或者 set hive.fetch.task.conversion=more

```
0. none : disable hive.fetch.task.conversion
1. minimal : SELECT STAR, FILTER on partition columns, LIMIT only
2. more    : SELECT, FILTER, LIMIT only (support TABLESAMPLE and virtual columns)
```

### 减少MapTask
- set mapred.max.split.size
- set mapred.min.split.size
- set mapred.min.split.size.per.node
- set mapred.min.split.size.per.rack
- set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat

### 增加MapTask
- 当input的文件都很大，任务逻辑复杂，map执行非常慢的时候，可以考虑增加Map数，来使得每个map处理的数据量减少，从而提高任务的执行效率文件，或者问价大小不大，但包含几千万的记录，如果用1个map去完成这个任务，肯定是比较耗时的，这种情况下，我们要考虑将这一个文件合理的拆分成多个，这样就可以用多个map任务去完成
> select data_desc, count(1), count(distinct id),sum(case when …),sum(case when ...),sum(…) from a group by data_desc

- set mapred.reduce.tasks=10;
- create table a_1 as select * from a distribute by rand(123);


### reduce数目设置
- hive.exec.reducers.bytes.per.reducer=1G：每个reduce任务处理的数据量
- hive.exec.reducers.max=999(0.95*TaskTracker数)：每个任务最大的reduce数目
- reducer数=min(参数2,总输入数据量/参数1)
- set mapred.reduce.tasks：每个任务默认的reduce数目。典型为0.99*reduce槽数，hive将其设置为-1，自动确定reduce数目。

## 压缩存储
### 合理利用文件存储格式 
- 创建表时，尽量使用 orc、parquet这些列式存储格式，因为列式存储的表，每一列的数据在物理上是存储在一起的，Hive查询时会只遍历需要列数据，大大减少处理的数据量。

### 压缩的原因
- Hive 最终是转为 MapReduce 程序来执行的，而MapReduce的性能瓶颈在于网络IO和磁盘IO，要解决性能瓶颈，最主要的是减少数据量，对数据进行压缩是个好的方式。压缩虽然是减少了数据量，但是压缩过程要消耗CPU的，但是在Hadoop中， 往往性能瓶颈不在于CPU，CPU压力并不大，所以压缩充分利用了比较空闲的 CPU

### 常见压缩格式
![image-20201206211806672](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:18:07-image-20201206211806672.png)

### 压缩方式的选择
- 压缩比率
- 压缩解压缩速度
- 是否支持 Split

### 压缩使用
- Job 输出文件按照 block 以 GZip 的方式进行压缩：
    - set mapreduce.output.fileoutputformat.compress=true // 默认值是 false
    - set mapreduce.output.fileoutputformat.compress.type=BLOCK // 默认值是 Record
    - set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.GzipCodec // 默认值是 org.apache.hadoop.io.compress.DefaultCodec

- Map 输出结果也以 Gzip 进行压缩：
    - set hive.exec.compress.intermediate=true;  开启hive中间传输数据压缩功能 
    - set mapred.map.output.compress=true; 开启mapreduce中map输出压缩功能
    - set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.GzipCodec //设置mapreduce中map输出数据的压缩方式 默认值是 org.apache.hadoop.io.compress.DefaultCodec 
- 对 Hive 输出结果和中间都进行压缩：
    - set hive.exec.compress.output=true // 默认值是 false，不压缩
    - set hive.exec.compress.intermediate=true // 默认值是 false，为 true 时 MR 设置的压缩才启用
- 开启Reduce输出阶段压缩
    - set hive.exec.compress.output=true;  开启hive最终输出数据压缩功能
    - set mapreduce.output.fileoutputformat.compress=true;  开启mapreduce最终输出数据压缩
    - set mapreduce.output.fileoutputformat.codec=org.apache.hadoop.io.compress.SnappyCodec;  设置mapreduce最终数据输出压缩方式
    - set mapreduce.output.fileoutputformat.type=BLOCK;  设置mapreduce最终数据输出压缩为块压缩

## OOM 优化
- 适用于那些由于内存超限内务被kill掉的场景。通过加大内存起码能让任务跑起来，不至于被杀掉。该参数不一定会降低任务执行时间。
- set mapreduce.reduce.memory.mb=5120 ;
- set mapreduce.reduce.Java.opts=-Xmx5000M -XX:MaxPermSize=128m;

## 扩展
### 只有一个reduce 的情况
- 使用了Order by （Order By是会进行全局排序）
- 直接COUNT(1),没有加GROUP BY，比如：SELECT COUNT(1) FROM tbl WHERE pt=’201909’
- 有笛卡尔积操作
