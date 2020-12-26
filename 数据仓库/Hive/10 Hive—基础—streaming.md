

## streaming

HIVE是通过利用或扩展Hadoop的组件功能来运行的，常见的抽象有InputFormat、OutputFormat、Mapper、Reducer，还包含一些自己的抽象接口，例如SerializerDeserializer(SerDe)、用户自定义函数（UDF）和StorageHandlers。这些组件都是java组件，不过hive将这些复杂的底层实现隐藏起来了，而提供给用户通过SQL语句执行的方式，而不是使用java代码

Streaming 提供了另一种处理数据的方式。在streaming job中，Hadoop StreamingAPI会为外部进程开启一个I/O管道。然后数据会被传给这个进程，然后数据从标准输入中读取数据，然后通过标准输出来写结果数据，最后返回到Streaming API job。尽管HIVE并没有直接使用Hadoop的StreamingAPI，不过他们的工作方式是一致的。这种管道计算模型对于Unix操作系统以及其衍生系统，如Linux和Mac OS  X的用户来说是非常熟悉的。

**Streaming的执行效率通常会比对应的编写UDF或改写InputFormat对象的方式要低。管道中的序列化和反序列化数据通常是低效的。而且以通常的方式很难调试整个程序。不过，对于快速原型设计和支持非java编写的已有的代码是非常有用的。对于那些不想写java代码的HIVE用户来说，这也是一个高效的方式**

HIVE中提供了多个语法来使用Streaming，包括：MAP()、REDUCE()、TRANSFORM()，你可以在sql 中使用一些脚本去处理一些任务，脚本分为mapper 端的脚本和reducer 端的脚本，MAP()实际上并非可以强制在map阶段执行Streaming，同样REDUCE()实际上并非可以强制在reduce阶段执行Streaming，所以推荐使用TRANSFORM()，这样可以避免误导读者对查询语句产生疑惑。



这里我们演示一个简单的功能就是判断一个时间戳是周几，也就是将时间戳转化为日期

创建新表，准备数据

```sql
CREATE TABLE ods.u_data (
  userid INT,
  movieid INT,
  rating INT,
  unixtime STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
```

```shell
wget http://files.grouplens.org/datasets/movielens/ml-100k.zip
unzip ml-100k.zip
```

```
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/ml-100k/u.data' OVERWRITE INTO TABLE ods.u_data;
```

创建脚本

```python
import sys
import datetime

for line in sys.stdin:
  line = line.strip()
  userid, movieid, rating, unixtime = line.split('\t')
  weekday = datetime.datetime.fromtimestamp(float(unixtime)).isoweekday()
  print('\t'.join([userid, movieid, rating, str(weekday)]))
```

创建新表、加载脚本、执行sql 使用脚本

```sql
CREATE TABLE u_data_new (
  userid INT,
  movieid INT,
  rating INT,
  weekday INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t';

add file /Users/liuwenqiang/weekday_mapper.py;

INSERT OVERWRITE TABLE u_data_new
SELECT
  TRANSFORM (userid, movieid, rating, unixtime)
  USING 'python weekday_mapper.py'
  AS (userid, movieid, rating, weekday)
FROM u_data;
```



这里为了帮助大家理解我再提供一个例子，但是这个例子没啥意义，那就是**恒等变换**

```
select transform (id,name) using '/bin/cat' as (id int,name varchar) from test;
```

```
select transform(id,name) using 'cut -f1' as (id) from test
```

![image-20201226121732115](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/26/12:17:32-image-20201226121732115.png)

例如我也可以使用其他命令或者是自定义的脚本,下面这个脚本就是为了获取服务器的当前时间

```sql
#!/bin/bash
time=`date +'%Y/%m/%d %H:%M:%S'`
echo $time
```

需要注意的实如果Streaming使用的都是UNIX系统或其衍生系统自带的如cat或sed这样的系统脚本程序，则不需要其他操作

但是当一个查询所需要的文件没有在每个TaskTracker上事先安装好时，用户需要使用分布式缓存将数据或者程序传输到集群中，然后在job完成后会清理掉这些缓存的数据和文件。（Hadoop的分布式缓存可以对缓存内的文件按照LRU原则进行删除，因此并非是job一结束就立即删除）。这个功能非常有用，因为在大规模集群上安装或者卸载大量的小组件会成为一件很有负担的事情。同时，缓存中会独立保存每个job的缓存文件，而不会相互干扰。

所以我们会需要使用add 命令将我们需要的脚本或者程序添加到分布式缓存中去` add file /Users/liuwenqiang/date.sh;` 然后使用 ` select TRANSFORM('a') USING 'sh date.sh' as (ntime);`去调用

![image-20201226120101601](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/26/12:09:18-12:01:02-image-20201226120101601.png)

## 总结

1. hiveserver2 为我们提供了JDBC 这样方便的调用方式。
2. streaming 为我们提供可一个接口，可以让我们方便的在自己SQL里调用自定义的脚本，至于Mapper 端的脚本还是Reducer 端的脚本，完全取决你在Mapper 阶段调用脚本还是在Reducer 阶段调用,而不是取决于你使用哪个方法调用脚本

