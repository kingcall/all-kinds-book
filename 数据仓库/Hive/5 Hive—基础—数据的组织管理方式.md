[TOC]

## Hive 中的数据组织管理方式

### Database和 Table

这两个概念其实很常见，Database 又叫命名空间，其实主要是为了组织管理和命名冲突，例如表名冲突，视图冲突，组织管理含义就很多了

![image-20201223150849662](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/15:09:03-15:08:50-image-20201223150849662.png)

这里我们有两个Database其实可以看到Database 在hive(HDFS) 上的物理存储就是两个文件夹，只不过这两个文件夹都有了自己的后缀 .db，如果你去命令行里查看的话是没有这个后缀的

![image-20201223151112406](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/15:11:12-image-20201223151112406.png)

我们随便选一个文件夹进行，发现表的物理存储或者是组织管理方式也是文件夹，其实到这里我们就差不多理解了，Database和 Table 都是数据组织的方式，而且都是文件夹，后面当你学习到分区的时候，你还会看到分区也是这样的

注意这里的文件夹名字就是表名字，没有了后缀

![image-20201223151132321](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/15:11:32-image-20201223151132321.png)

### 分区和分桶

我们知道传统的DBMS系统一般都具有表分区的功能，**通过表分区能够在特定的区域检索数据**，减少扫描成本，在一定程度上提高查询效率，当然我们还可以通过进一步在分区上建立索引进一步提升查询效率。在此就不赘述了。例如mysql 中也是有分区表的，但是我们用的比较少，所以很多了忽略了这个东西。

在Hive数仓中也有分区分桶的概念，在逻辑上分区表与未分区表没有区别，在物理上分区表会将数据按照分区键的列值存储在表目录的子目录中，目录名=“分区键=键值”。其中需要注意的是分区键的值不一定要基于表的某一列（字段），它可以指定任意值，只要查询的时候指定相应的分区键来查询即可。我们可以对分区进行添加、删除、重命名、清空等操作。因为分区在特定的区域（子目录）下检索数据，它作用同DNMS分区一样，都是为了减少扫描成本。

分桶则是指定分桶表的某一列，让该列数据按照哈希取模的方式随机、均匀地分发到各个桶文件中。因为分桶操作需要根据某一列具体数据来进行哈希取模操作，故指定的分桶列必须基于表中的某一列（字段）。因为分桶改变了数据的存储方式，它会把哈希取模相同或者在某一区间的数据行放在同一个桶文件中。如此一来便可提高查询效率，如：我们要对两张在同一列上进行了分桶操作的表进行JOIN操作的时候，只需要对保存相同列值的桶进行JOIN操作即可。同时分桶也能让取样（Sampling）更高效。

**其实一句话，分区分桶就是为了提高查询效率的**，而原理就是减少数据的扫描



#### 分区

是指按照数据表的某列或某些列分为多个区，**区从形式上可以理解为文件夹**，比如我们要收集某个大型网站的日志数据，一个网站每天的日志数据存在同一张表上，由于每天会生成大量的日志，导致数据表的内容巨大，在查询时进行全表扫描耗费的资源非常多。那其实这个情况下，我们可以按照日期对数据表进行分区，不同日期的数据存放在不同的分区，在查询时只要指定分区字段的值就可以直接从该分区查找。

创建分区表的时候，通过关键字 partitioned by (name string)声明该表是分区表，并且是按照字段name进行分区，name值相同的所有记录存放在一个分区中，分区属性name的类型是string类型。当然，可以依据多个列进行分区，即对某个分区的数据按照某些列继续分区。

其次，向分区表导入数据的时候，要通过关键字partition（name=“jack”）显示声明数据要导入到表的哪个分区，这里表示要将数据导入到分区为name=jack的分区。

所谓分区，这是将满足某些条件的记录打包，放在同一个文件夹里，在查询时提高效率，相当于按文件夹对文件进行分类，文件夹名可类比分区字段。

这个分区字段形式上存在于数据表中，在查询时会显示到客户端上，但并不真正在存储在数据表文件中，是所谓伪列。但是这里有另外一个概念有点与之相悖，那就是动态分区，动态分区分分区列往往是表中的一列或者是多列

```
CREATE TABLE ods.u_data (
  userid INT,
  movieid INT,
  rating INT,
  unixtime STRING)
 partitioned by(year string,month string ,day string) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
```

下面我多load 几次看一下效果

```sql
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/ml-100k/u.data' OVERWRITE INTO TABLE ods.u_data partition(year='2020',month='2020-12',day='2020-12-21');
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/ml-100k/u.data' OVERWRITE INTO TABLE ods.u_data partition(year='2020',month='2020-12',day='2020-12-22');
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/ml-100k/u.data' OVERWRITE INTO TABLE ods.u_data partition(year='2020',month='2020-12',day='2020-12-23');
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/ml-100k/u.data' OVERWRITE INTO TABLE ods.u_data partition(year='2021',month='2020-01',day='2021-01-01');
```

首先我们看到分区的物理实现方式就是文件夹，分区的名字`分区名称='分区值'`

![image-20201223152518512](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/15:25:19-image-20201223152518512.png)

多字段分区的管理方式就是文件嵌套，直到最小分区

![image-20201223152634350](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/15:26:34-image-20201223152634350.png)

其实上面按照时间分区的方式是我们最常见的，大家注意和下面这种方式进行区别一下，可以考虑一下为什么,又不懂的评论区留言

![image-20201223160816281](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/16:08:16-image-20201223160816281.png)

#### 分桶

**分桶是相对分区进行更细粒度的划分**。分桶将整个数据内容按照某列属性值的hash值进行区分，如要安装name属性分为3个桶，就是对name属性值的hash值对3取摸，按照取模结果对数据分桶。如取模结果为0的数据记录存放到一个文件，取模为1的数据存放到一个文件，取模为2的数据存放到一个文件。

其实关于这个粒度更细，说实话我一直不敢苟同，为什么呢？说之前我先说一下，大家为什么说分桶是更细的组织方式啊，因为我们很多时候分桶都是配合分区使用的，也就是说我们一般是分区完了，在某一个分区里面进行分桶的，所以我们说是更细的粒度，但是呢，你从另外一个角度去看问题，如果我按照表的某个字段做动态分区，那进入同一个分区的是不是这个字段值相同的数据呢，那你要是对这个字段做分桶呢，这个时候取决于你的桶多少，如果你的桶比较少，那么同一个桶里的数据分桶字段值不一定相等，只是对桶的个数取余是相等的。

分桶对查询的性能提高体现在join 的时候以及方便我们对数据进行抽样

所以这里引出了一个问题那就是你如果在没有分区的情况下，去使用分桶倒不先考虑使用分桶

```java
CREATE TABLE ods.u_data (
  userid INT,
  movieid INT,
  rating INT,
  unixtime STRING)
 partitioned by(year string,month string ,day string) 
 CLUSTERED BY (`movieid` ) INTO 3 BUCKETS      
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/ml-100k/u.data' OVERWRITE INTO TABLE ods.u_data partition(year='2020',month='2020-12',day='2020-12-21');
```

下面是数据的组织管理方式，我们看到数据按照movieid字段分到了三个文件里面

![image-20201223155633740](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/15:56:34-image-20201223155633740.png)

## 总结

1. 不论是Database和 Table还是分区分桶都是数据组织管理的方式，目的都是为了提高效率
2. 分区不一定基于列，是可以指定的任意值，也可以基于列做动态分区，保存为Hdfs里面就是一个目录
3. 分桶是要基于列的值做hash的，桶的数据就是hdfs文件的数量，两张分桶相同的表，可以直接在map端做join，并且这个笛卡尔积也是大大减小的。如果分桶字段不存在你会得到`SemanticException [Error 10002]: Invalid column reference  'XXX'` 的异常
4. 分区是以文件夹形式管理的，分桶是以文件形式管理的