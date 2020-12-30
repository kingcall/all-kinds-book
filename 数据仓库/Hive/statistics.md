## 背景
- 统计信息，例如一个表有多少行，多少个分区，列的直方图等重要的信息。统计信息的关键作用是查询优化。把统计信息作为输入，通过成本优化函数，可以方便的比较不同的查询方案，并且从中进行最优的执行计划。统计数据有时可以直接满足用户的查询目的。比如他们只是查询一些基础数据，而不需要通过运行执行计划.





https://cwiki.apache.org/confluence/display/Hive/Streaming+Data+Ingest

https://cwiki.apache.org/confluence/display/Hive/Streaming+Data+Ingest+V2





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

