[toc]
## 动态分区调整

- 动态分区属性：设置为true表示开启动态分区功能（默认为false）hive.exec.dynamic.partition=true;
- 动态分区属性：设置为nonstrict,表示允许所有分区都是动态的（默认为strict）设置为strict，表示必须保证至少有一个分区是静态的hive.exec.dynamic.partition.mode=strict;
- 动态分区属性：每个mapper或reducer可以创建的最大动态分区个数hive.exec.max.dynamic.partitions.pernode=100;
- 动态分区属性：一个动态分区创建语句可以创建的最大动态分区个数hive.exec.max.dynamic.partitions=1000;
- 动态分区属性：全局可以创建的最大文件个数hive.exec.max.created.files=100000;
- 控制DataNode一次可以打开的文件个数 这个参数必须设置在DataNode的$HADOOP_HOME/conf/hdfs-site.xml文件中
```
<property>
    <name>dfs.datanode.max.xcievers</name>
    <value>8192</value>
</property>
```
### 注意
> 在Hive中，动态分区会造成在插入数据过程中，生成过多零碎的小文件

## 动态分区插入

如果需要创建非常多的分区，用户就需要写非常多的条件查询sql把数据插入对应分区。好在`Hive`提供了动态分区功能，可以根据分区字段的取值自动创建分区。前面列出的开启动态分区`hive.exec.dynamic.partition`，并且`hive.exec.dynamic.partition.mode`需要为非严格模式，通常如果分区很多的话，`hive.exec.max.dynamic.partitions.pernode`也需要设置为一个较大的数，否则会有报错提醒。

现在有`sql`：

```sql
insert overwrite table employees partitions (country, state)
select ...,se.cnty, se.st
from staged_employees se;
```

可以看出，`Hive`根据`select`语句中最后两列来确定分区字段`country`和`state`的值，这里刻意使用了不同的命名，就是为了强调源表字段和输出分区值之间的关系是根据位置而不是根据命名来匹配的。

## 动静分区结合

也可以混合使用动态和静态分区。上面那个例子，我们可以指定国家这个分区值五为静态值`US`，而分区字段`state`是动态值：

```sql
insert overwrite table employees partitions (country = 'US', state)
select ...,se.cnty, se.st
from staged_employees se
where se.cnty = 'US';
```

**注意**：静态分区需要出现在动态分区字段之前。

动态分区功能默认情况下是没有开启的，默认是以`严格`模式执行，这种模式下要求至少有一列分区字段是静态的。这样做的好处是可以防止因设计或其它错误查询导致产生大量的分区，比如`sql boy`不小心使用了时间戳作为分区字段，那将是灾难。在日常导入一天的数据，通常是指定日期为静态分区，小时为动态分区，进行混合模式导入。

## 例子

建表

```sql
create table if not exists test.test
(
id string,
name string
)
partitioned by (dt string,hour string)
row format delimited fields terminated by '\t';

create table if not exists test.test2
(
id string,
name string
)
partitioned by (dt string,hour string)
row format delimited fields terminated by '\t'
stored as orc;
```

导入数据到`test.test`表

```sql
load data local inpath '/home/hadoop/data/test.txt' into table test.test partition(dt = '2019-09-10', hour = '02');

test.txt
001	keguang
002	kg
003	kk
004	ikeguang
```

利用动态分区插入

```sql
insert overwrite table test.test2 partition(dt, hour) select `(dt|hour)?+.+`,dt,hour from test.test;
```

这里，`(dt|hour)?+.+`表示查询出`test`表除了`dt`和`hour`这两个字段的其它所有字段。

