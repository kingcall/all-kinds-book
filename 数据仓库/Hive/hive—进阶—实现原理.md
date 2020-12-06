[toc]
## 概论
-  目前，Hive除了支持MapReduce计算引擎，还支持Spark和Tez这两中分布式计算引擎。
-  HDFS中最关键的一点就是，数据存储HDFS上是没有schema的概念的(schema:相当于表里面有列、字段、字段名称、字段与字段之间的分隔符等，这些就是schema信息)然而HDFS上的仅仅只是一个纯的文本文件而已，那么，没有schema，就没办法使用sql进行查询了啊。。。因此，在这种背景下，就有问题产生：如何为HDFS上的文件添加Schema信息？如果加上去，是否就可以通过SQL的方式进行处理了呢？于是强大的Hive出现了

## SQL转化为MapReduce的过程
了解了MapReduce实现SQL基本操作之后，我们来看看Hive是如何将SQL转化为MapReduce任务的，整个编译过程分为六个阶段：

- Antlr定义SQL的语法规则，完成SQL词法，语法解析，将SQL转化为抽象语法树AST Tree
- 遍历AST Tree，抽象出查询的基本组成单元QueryBlock
- 遍历QueryBlock，**翻译为执行操作树OperatorTree**
- **逻辑层优化器进行OperatorTree变换**，合并不必要的ReduceSinkOperator，减少shuffle数据量
- 遍历OperatorTree，翻译为MapReduce任务
- 物理层优化器进行MapReduce任务的变换，生成最终的执行计划

## 行式存储vs列式存储
行式数据库存储在hdfs上式按行进行存储的，一个block存储一或多行数据。而列式数据库在hdfs上则是按照列进行存储，一个block可能有一列或多列数据。

### 数据压缩
- 对于行式数据库，必然按行压缩，当一行中有多个字段，各个字段对应的数据类型可能不一致，压缩性能压缩比就比较差。
对于列式数据库，必然按列压缩，每一列对应的是相同数据类型的数据，故列式数据库的压缩性能要强于行式数据库。
### 数据查询
假设执行的查询操作是：select id,name from table_emp;

- 对于行式数据库，它要遍历一整张表将每一行中的id,name字段拼接再展现出来，这样需要查询的数据量就比较大，效率低。
对于列式数据库，它只需找到对应的id,name字段的列展现出来即可，需要查询的数据量小，效率高。
假设执行的查询操作是：select * from table_emp;

## join 实现原理
- select u.name, o.orderid from order o join user u on o.uid = u.uid;
```
在map的输出value中为不同表的数据打上tag标记，在reduce阶段根据tag判断数据来源
```
![image-20201206211916602](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:19:17-image-20201206211916602.png)


### reduce-join(common join也叫做shuffle join)
> 这种情况下生再两个table的大小相当，但是又不是很大的情况下使用的
- 在map阶段，map函数同时读取两个文件File1和File2，为了区分两种来源的key/value数据对，对每条数据打一个标签（tag）,比如：tag=0表示来自文件File1，tag=2表示来自文件File2。**即：map阶段的主要任务是对不同文件中的数据打标签**。
- 在reduce阶段，reduce函数获取key相同的来自File1和File2文件的value list，然后对于同一个key，对File1和File2中的数据进行join（笛卡尔乘积）。**即：reduce阶段进行实际的连接操作**。

### map-join
- 之所以存在reduce side join，是因为在map阶段不能获取所有需要的join字段，即：同一个key对应的字段可能位于不同map中。Reduce side join是非常低效的，因为shuffle阶段要进行大量的数据传输。
- Map side join是针对以下场景进行的优化：两个待连接表中，有一个表非常大，**而另一个表非常小，以至于小表可以直接存放到内存中**。这样，我们可以将小表复制多份，让每个maptask内存中存在一份（比如存放到hashtable中），然后只扫描大表：对于大表中的每一条记录key/value，在hash table中查找是否有相同的key的记录，**如果有，则连接后输出即可**
- 为了支持文件的复制，Hadoop提供了一个类DistributedCache
```
1. 用户使用静态方法DistributedCache.addCacheFile()指定要复制的文件，它的参数是文件的URI（如果是HDFS上的文件，可以这样：hdfs://namenode:9000/home/XXX/file，其中9000是自己配置的NameNode端口号）。JobTracker在作业启动之前会获取这个URI列表，并将相应的文件拷贝到各个TaskTracker的本地磁盘上。
2. 用户使用DistributedCache.getLocalCacheFiles()方法获取文件目录，并使用标准的文件读写API读取相应的文件。
```

#### map-join 内存溢出
- 可以关闭map-join

### Semi Join
- Semi Join，也叫半连接，是从分布式数据库中借鉴过来的方法。它的产生动机是：对于reduce side join，跨机器的数据传输量非常大，这成了join操作的一个瓶颈，**如果能够在map端过滤掉不会参加join操作的数据，则可以大大节省网络IO**
- 实现方法很简单：选取一个小表，假设是File1，**将其参与join的key抽取出来**，保存到文件File3中，File3文件一般很小，可以放到内存中。在map阶段，使用DistributedCache将File3复制到各个TaskTracker上，然后将File2中不在File3中的key对应的记录过滤掉，剩下的reduce阶段的工作与reduce side join相同。

###  reduce side join + BloomFilter
- 在某些情况下，SemiJoin抽取出来的小表的key集合在内存中仍然存放不下，这时候可以使用BloomFiler以节省空间。
- BloomFilter最常见的作用是：判断某个元素是否在一个集合里面。它最重要的两个方法是：add()和contains()。最大的特点是不会存在 false negative，即：如果contains()返回false，则该元素一定不在集合中，但会存在一定的 false positive，即：如果contains()返回true，则该元素一定可能在集合中。
- 因而可将小表中的key保存到BloomFilter中，在map阶段过滤大表，可能有一些不在小表中的记录没有过滤掉（但是在小表中的记录一定不会过滤掉），这没关系，只不过增加了少量的网络IO而已。

### Sort-Merge-Bucket-Join
```
set hive.auto.convert.sortmerge.join=true;
set hive.optimize.bucketmapjoin = true;
set hive.optimize.bucketmapjoin.sortedmerge = true;
set hive.auto.convert.sortmerge.join.noconditionaltask=true;
```
- 对于map端连接的情况，两个表以相同方式划分桶。处理左边表内某个桶的mapper知道右边表内相匹配的行在对应的桶内。因此，mapper只需要获取那个桶 (这只是右边表内存储数据的一小部分)即可进行连接。这一优化方法并不一定要求两个表必须桶的个数相同，两个表的桶个数是倍数关系也可以
- 桶中的数据可以根据一个或多个列另外进行排序。由于这样对每个桶的连接变成了高效的归并排序(merge-sort), 因此可以进一步提升map端连接的效率
```
CREATE TABLE bucketed_users (id INT, name STRING)
CLUSTERED BY (id) SORTED BY (id ASC) INTO 4 BUCKETS;
```


## group by 实现原理
- select rank, isonline, count(*) from city group by rank, isonline;
```
将GroupBy的字段组合为map的输出key值，利用MapReduce的排序，在reduce阶段保存LastKey区分不同的key。
```
![image-20201206211942349](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:19:42-image-20201206211942349.png)

## distinct 实现原理
- select dealid, count(distinct uid) num from order group by dealid;
- 当只有一个distinct字段时，如果不考虑Map阶段的Hash GroupBy，只需要将GroupBy字段和Distinct字段组合为map输出key，利用mapreduce的排序，同时将GroupBy字段作为reduce的key，在reduce阶段保存LastKey即可完成去重

- 该 SQL 语句会按照 age 和 pageid 预先分组，进行 distinct 操作。然后会再按 照 age 进行分组，再进行一次 distinct 操作

![image-20201206212008661](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:20:09-image-20201206212008661.png)



