[toc]

## 常见的数据格式
### 背景
- 当今的数据处理大致可分为两大类，联机事务处理 OLTP(on-line transaction processing)联机分析处理 OLAP(On-Line Analytical Processing)
> OLTP 是传统关系型数据库的主要应用用来执行一些基本的、日常的事务处理比如数据库记录的增、删、改、查等等而OLAP则是分布式数据库的主要应用它对实时性要求不高，但处理的数据量大通常应用于复杂的动态报表系统上

![image-20201206214745123](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:47:45-image-20201206214745123.png)


#### 行存储的特点
- 查询满足条件的一整行数据的时，只需要找到其中一个值，其余的值都在相邻地方，所以此时行存储查询的速度更快。
- 传统的关系型数据库，如 Oracle、DB2、MySQL、SQL SERVER 等采用行式存储法(Row-based)，在基于行式存储的数据库中， **数据是按照行数据为基础逻辑存储单元进行存储的， 一行中的数据在存储介质中以连续存储形式存在**。
- - 行存储的特点：查询满足条件的一整行数据的时，只需要找到其中一个值，其余的值都在相邻地方，所以此时行存储查询的速度更快
- TEXTFILE和SEQUENCEFILE的存储格式都是基于行存储的；


#### 列存储的特点
- 因为每个字段的数据聚集存储，在查询只需要少数几个字段的时候，能大大减少读取的数据量；每个字段的数据类型一定是相同的，列式存储可以针对性的设计更好的设计压缩算法。TEXTFILE和SEQUENCEFILE的存储格式都是基于行存储的；ORC和PARQUET是基于列式存储的。
- 列式存储(Column-based)是相对于行式存储来说的，新兴的Hbase、HPVertica、EMCGreenplum等分布式数据库均采用列式存储。在基于列式存储的数据库中， **数据是按照列为基础逻辑存储单元进行存储的，一列中的数据在存储介质中以连续存储形式存在**。

![image-20201206214802357](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:48:02-image-20201206214802357.png)

- 列存储的特点：因为每个字段的数据聚集存储，在查询只需要少数几个字段的时候，能**大大减少读取的数据量**；每个字段的数据类型一定是相同的，列式存储可以针对性的设计更好的设计压缩算法。ORC和PARQUET是基于列式存储的。

### TextFile
- 默认格式，存储方式为行存储，数据不做压缩，磁盘开销大，数据解析开销大。 
- 可结合Gzip、Bzip2使用(系统自动检查，执行查询时自动解压)，但使用这种方式，压缩后的文件不支持split，Hive不会对数据进行切分，从而无法对数据进行并行操作。
- 并且在反序列化过程中，必须逐个字符判断是不是分隔符和行结束符，因此反序列化开销会比SequenceFile高几十倍。
```
create table log_text(track_time string, url string, session_id string, referer string, ip string, end_user_id string, city_id string)
row format delimited fields terminated by '\t'
stored as textfile;
```

### SequenceFile
- SequenceFile是Hadoop API提供的一种二进制文件支持，，存储方式为行存储，其具有使用方便、可分割、可压缩的特点。SequenceFile支持三种压缩选择：NONE，RECORD，BLOCK。Record压缩率低，一般建议使用BLOCK压缩。优势是文件和hadoop api中的MapFile是相互兼容的。

### RCfile：
存储方式：数据按行分块，每块按列存储。结合了行存储和列存储的优点：首先，RCFile 保证同一行的数据位于同一节点，因此元组重构的开销很低；其次，像列存储一样，RCFile 能够利用列维度的数据压缩，并且能跳过不必要的列读取；

### ORCfile
- 存储方式：**数据按行分块 每块按照列存储**(不是真正意义上的列存储，可以理解为分段列存储)。
> 假设一个表有10亿行数据，按照列式存储的定义，应该先将某个字段的10亿条数据存储完之后，再存储其他字段。
- 压缩快 快速列存取。效率比rcfile高,是rcfile的改良版本。
- 是以二进制方式存储的，所以是不可以直接读取，ORC文件也是自解析的，它包含许多的元数据，这些元数据都是同构ProtoBuffer进行序列化的。

```
create table log_orc(track_time string, url string, session_id string, referer string, ip string, end_user_id string, ciry_id string)
row format delimited fields terminated by '/t'
stored as orc;
```
- 默认创建的ORC存储方式，导入数据后的大小为:2.8 M  /user/hive/warehouse/log_orc/000000
比Snappy压缩的还小。原因是orc存储文件默认采用ZLIB压缩，ZLIB采用的是deflate压缩算法。比snappy压缩的小。

#### 格式
- 每个Orc文件由1个或多个stripe组成，**每个stripe一般为HDFS的块大小**，每一个stripe包含多条记录，**这些记录按照列进行独立存*储*对应到Parquet中的row group的概念。每个Stripe里有三部分组成，分别是Index Data，Row Data，Stripe Footer；
- Stripe对应到Parquet中的row group的概念。

![image-20201206214822743](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:48:23-image-20201206214822743.png)
- 每次读取文件是以行组为单位的，一般为HDFS的块大小，保存了每一列的索引和数据。


#### 数据访问
- **读取ORC文件是从尾部开始的**，第一次读取16KB的大小，尽可能的将Postscript和Footer数据都读入内存。文件的最后一个字节保存着PostScript的长度，它的长度不会超过256字节，PostScript中保存着整个文件的元数据信息，它包括文件的压缩格式、文件内部每一个压缩块的最大长度(每次分配内存的大小)、Footer长度，以及一些版本信息。在Postscript和Footer之间存储着整个文件的统计信息(上图中未画出)，这部分的统计信息包括每一个stripe中每一列的信息，主要统计成员数、最大值、最小值、是否有空值等


### Parquet
- Parquet文件是**以二进制方式存储的，所以是不可以直接读取的**，文件中包括该文件的数据和元数据，因此Parquet格式文件是自解析的。
```
create table log_parquet(track_time string, url string, session_id string, referer string, ipstring,end_user_idstring, city_id string)
row format delimited fields terminated by '/t'
stored as parquet;
```


## 存储与压缩

## 存储与索引
- 分区与分桶可以看做文件块上的索引可以快速定位到数据块，那么底层的存储格式Parquet和Orc 可以看做是文件里面的索引，可以快速定位到数据