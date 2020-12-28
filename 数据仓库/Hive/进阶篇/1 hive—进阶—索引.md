[toc]
## 索引
- Hive从0.7.0版本开始引入了索引，目的是提高Hive表指定列的查询速度。没有索引的时候，Hive在执行查询时需要加载整个表或者整个分区(分区表并提供了相关过滤条件)，即使你加了带有谓词的查询（如'WHERE table.column = 10'）依然会加载整个表或分区并处理所有行。**但当在指定列上存在索引，再通过指定列查询时，那么只会加载和处理部分文件**。在可以预见到分区数据非常庞大的情况下，索引常常是优于分区的。
- 此外，同传统关系型数据库一样，增加索引在提升查询速度的同时，会额外消耗资源去创建索引和需要更多的磁盘空间存储索引,但是Hive只有有限的索引功能，没有普通关系型数据库中键的概念，而且索引也不会自动更新
- Hive 0.8.0版本中增加了bitmap索引。

### 1. 机制和原理

- Hive的索引其实是一张索引表（Hive的物理表），里面的字段包括：索引列的值、该值对应的 HDFS 文件路径、该值在文件中的偏移量。在查询涉及到索引字段时，首先到索引表查找索引列值对应的 HDFS 文件路径及偏移量，这样就避免了全表扫描。
- 当Hive通过索引列执行查询时，首先通过一个MRJob去查询索引表，**根据索引列的过滤条件，查询出该索引列值对应的HDFS文件目录及偏移量，并且把这些数据输出到HDFS的一个文件中，然后再根据这个文件中去筛选原文件，作为查询Job的输入**

在指定列上建立索引，会产生一张索引表（表结构如下），

```properties
+--------------+----------------+----------+--+
|   col_name   |   data_type    | comment     |
+--------------+----------------+----------+--+
| empno        | int            |  建立索引的列  |   
| _bucketname  | string         |  HDFS 文件路径  |
| _offsets     | array<bigint>  |  偏移量       |
+--------------+----------------+----------+--+
```



### 2. 创建索引

```sql
CREATE INDEX index_name     --索引名称
  ON TABLE base_table_name (col_name, ...)  --建立索引的列
  AS index_type    --索引类型
  [WITH DEFERRED REBUILD]    --重建索引
  [IDXPROPERTIES (property_name=property_value, ...)]  --索引额外属性
  [IN TABLE index_table_name]    --索引表的名字
  [
     [ ROW FORMAT ...] STORED AS ...  
     | STORED BY ...
  ]   --索引表行分隔符 、 存储格式
  [LOCATION hdfs_path]  --索引表存储位置
  [TBLPROPERTIES (...)]   --索引表表属性
  [PARTITIONED BY(...)]     -- 索引的分区 
  [COMMENT "index comment"];  --索引注释
```
1. AS index_type 语句指定了索引处理器，例如CompactIndexHandler是其中的一个实现，当然还有我们上面提到的BITMAP
2. 如果省略partition by语句的话，那么索引将会包含原始表所有分区。

```sql
CREATE TABLE employees(
    name string,
    salary float,
    subordinates ARRAY<string>,
    deductions MAP<string,float>,
    address struct<street:STRING, city:STRING, state:STRING, zip:INT>
)
partitioned by(country string,state string);

CREATE INDEX employees_index
    on table employees(country)
    AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler'
    WITH deferred rebuild
    idxproperties('create ='kingcall','create_at'='2020-12-20')
    in table employees_index_table
    partitioned by(country,state)
    comment 'employees indexed by country and state';
```

### 3. 查看索引

```sql
--显示表上所有列的索引
SHOW FORMATTED INDEX ON table_name;
```

### 4. 删除索引

删除索引会删除对应的索引表。

```sql
DROP INDEX [IF EXISTS] index_name ON table_name;
```

如果存在索引的表被删除了，其对应的索引和索引表都会被删除。如果被索引表的某个分区被删除了，那么分区对应的分区索引也会被删除。

### 5. 重建索引

1. 刚创建完的Hive索引表是没有数据的，需要生成索引数据 
2. 如果原表数据更新了，索引数据是要重建的,Hive 会启动 MapReduce 作业去建立索引

```sql
alter index test_index on test rebuild;
```

例如对于上面的例子,如果我只更新了某一个分区的数据我也可以只更新这个分区的索引

```sql
ALTER　INDEX employees_index
on TABLE employees
Partition(country='US')
rebuild;
```



### 6. 使用索引

默认情况下，虽然建立了索引，但是 Hive 在查询时候是不会自动去使用索引的，需要开启相关配置。开启配置后，涉及到索引列的查询就会使用索引功能去优化查询。

```sql
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
SET hive.optimize.index.filter=true;
SET hive.optimize.index.filter.compact.minsize=0;
```

## 总结

索引表的优点是可以减少扫描数据，索引表的缺点也很多，最主要的一个在于：索引表无法自动 rebuild，这也就意味着如果表中有数据新增或删除，则必须手动 rebuild，重新执行 MapReduce 作业，生成索引表数据。

维护索引需要额外的存储空间，同时创建索引也需要消耗计算量，所以适用于不更新或者更新不频繁的静态字段，以免总是重建索引数据。

同时按照[官方文档](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Indexing) 的说明，Hive 会从 3.0 开始移除索引功能，主要基于以下两个原因：

- 具有自动重写的物化视图 (Materialized View) 可以产生与索引相似的效果（Hive 2.3.0 增加了对物化视图的支持，在 3.0 之后正式引入）
- 使用列式存储文件格式（Parquet，ORC）进行存储时，这些格式支持选择性扫描，可以跳过不需要的文件或块。

