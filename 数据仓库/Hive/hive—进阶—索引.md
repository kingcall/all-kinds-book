[toc]
## 索引
- Hive从0.7.0版本开始加入了索引，目的是提高Hive表指定列的查询速度。没有索引的时候，Hive在执行查询时需要加载整个表或者整个分区，然后处理所有的数据，但当在指定列上存在索引，再通过指定列查询时，那么只会加载和处理部分文件。在可以预见到分区数据非常庞大的情况下，索引常常是优于分区的。
- 此外，同传统关系型数据库一样，增加索引在提升查询速度的同时，会额外消耗资源去创建索引和需要更多的磁盘空间存储索引。
- Hive 0.8.0版本中增加了bitmap索引。

### 机制和原理
- Hive的索引其实是一张索引表（Hive的物理表），在表里面存储索引列的值，该值对应的HDFS的文件路径和该值在数据文件中的偏移量。
- 当Hive通过索引列执行查询时，首先通过一个MRJob去查询索引表，**根据索引列的过滤条件，查询出该索引列值对应的HDFS文件目录及偏移量，并且把这些数据输出到HDFS的一个文件中，然后再根据这个文件中去筛选原文件，作为查询Job的输入**

### 优点
- 可以避免全表扫描和资源浪费

### 创建索引
```
create index test_index on table test(id)
as 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler'
with deferred rebuild
in table test;
```
### 生成索引数据
刚创建完的Hive索引表是没有数据的，需要生成索引数据
```
alter index test_index on test rebuild;
```

### 使用索引
```
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
SET hive.optimize.index.filter=true;
SET hive.optimize.index.filter.compact.minsize=0;
```

### 删除索引
```
drop index test_index on test;
```

### 缺点
- 使用过程繁琐
- 需用额外Job扫描索引表
- 不会自动刷新，如果表有数据变动，索引表需要手动刷新