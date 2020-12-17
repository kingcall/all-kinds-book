[TOC]
# hive知识点
工作中`hive`常用知识点。
## Hive简介
hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。
## 创建hive表
```sql
# 新建个数据库test
create database test;
create external table if not exists test.test(id string, name string);
```
这里创建了一个名为`test`的`hive`外部表，外部表与普通表的区别：
1. 在导入数据到外部表，数据并没有移动到自己的数据仓库目录下，也就是说外部表中的数据并不是由它自己来管理的！而表则不一样；
1. 在删除表的时候，Hive将会把属于表的元数据和数据全部删掉；而删除外部表的时候，Hive仅仅删除外部表的元数据，数据是不会删除的！那么，应该如何选择使用哪种表呢？在大多数情况没有太多的区别，因此选择只是个人喜好的问题。但是作为一个经验，如果所有处理都需要由Hive完成，那么你应该创建表，否则使用外部表！

### 创建分区表
通过`partition`关键字指定分区字段，分区表方便`hive`快速查询索引数据。
```sql
create table if not exists test.test
(
id string,
name string
)
partitioned by (dt string,hour string)
row format delimited fields terminated by '\t';
```
这里指定了两个分区：`dt`和`hour`，对应`hdfs`的2级目录`dt,hour`：
```
[hadoop@qcloud-test-hadoop01 ~]$ hdfs dfs -ls -R /hive/warehouse/test.db/test
drwxr-xr-x   - hadoop supergroup          0 2019-09-10 19:07 /hive/warehouse/test.db/test/dt=2019-09-10
drwxr-xr-x   - hadoop supergroup          0 2019-09-10 17:59 /hive/warehouse/test.db/test/dt=2019-09-10/hour=02
-rwxr-xr-x   2 hadoop supergroup         39 2019-09-10 17:59 /hive/warehouse/test.db/test/dt=2019-09-10/hour=02/test.txt
drwxr-xr-x   - hadoop supergroup          0 2019-09-10 19:07 /hive/warehouse/test.db/test/dt=2019-09-10/hour=03
-rwxr-xr-x   2 hadoop supergroup         39 2019-09-10 19:07 /hive/warehouse/test.db/test/dt=2019-09-10/hour=03/test.txt
```

### 删除分区
```sql
ALTER TABLE table_name DROP IF EXISTS PARTITION(year = 2015, month = 10, day = 1);
```

## 创建orc存储格式表
`hive`创建`orc`格式表不能像`textfile`格式一样直接`load`数据到表中，一般需要创建临时`textfile`表，然后通过`insert into` 或者`insert overwrite`到`orc`存储格式表中。

### 1) 临时表testfile存储格式
```sql
create table if not exists test.test
(
id string,
name string
)
partitioned by (dt string,hour string)
row format delimited fields terminated by '\t';
```
test.txt
```
001	keguang
002	kg
003	kk
004	ikeguang
```
load data local inpath '/home/hadoop/data/test.txt' into table test.test partition(dt = '2019-09-10', hour = '02');
### 2) 导入数据到orc表
```sql
create table if not exists test.test2
(
id string,
name string
)
partitioned by (dt string,hour string)
row format delimited fields terminated by '\t'
stored as orc;
```
`insert select` 导入数据
```
insert overwrite table test.test2 partition(dt, hour) select `(dt|hour)?+.+`,dt,hour from test.test;
```
这里`(dt|hour)?+.+`表示排除`dt,hour`两个字段，由于动态分区`partition(dt, hour)`是按照`select`出来的最后2个字段作为分区字段的。其实这里```select * ``` 也是可以的，因为分区表查询结果，最后两个字段就是分区字段。
```
select * from test.test2;

结果
001	keguang	2019-09-10	02
002	kg	2019-09-10	02
003	kk	2019-09-10	02
004	ikeguang	2019-09-10	02
```
所以说，非textfile存储格式表导入数据步骤：
1. 导入数据到textfile
1. 查询数据插入orc格式表

### select 排除字段
选择`tableName`表中除了`name、id、pwd`之外的所有字段
```
set hive.support.quoted.identifiers=None;
select `(name|id|pwd)?+.+` from tableName;
```

## UDF用法
添加临时函数
```sql
add jar /home/hadoop/codejar/flash_format.jar;
create temporary function gamelabel as 'com.js.dataclean.hive.udf.hm2.GameLabel';
```
删除临时函数
```sql
drop temporary function 数据库名.函数名;
```
添加永久函数
```sql
create function hm2.gamelabel as 'com.js.dataclean.hive.udf.hm2.GameLabel' using jar 'hdfsJarPath'
```
==注意==：1). 需要指定数据库.函数名，即`hm2.gamelabel`，否则默认在`default`数据库下面：`default.gamelabel`;2). `hdfsJarPath`即该`jar`包需要上传到`hdfs`目录；

删除永久函数：
```sql
drop function 数据库名.函数名字;
```
如果客户端通过`hiveserver2`连接`hive`，为了正常使用自定义的永久`udf`，需要执行`reload function`；

### 查看函数用法

查month 相关的函数
```
show functions like '*month*'
```
查看 add_months 函数的用法
```
desc function add_months;
```
查看 add_months 函数的详细说明并举例
```
desc function extended add_months;
```

### UDAF用法
关于UDAF开发注意点：

1.需要`import org.apache.hadoop.hive.ql.exec.UDAF`以及`org.apache.hadoop.hive.ql.exec.UDAFEvaluator`,这两个包都是必须的

2.函数类需要继承`UDAF`类，内部类Evaluator实现UDAFEvaluator接口

3.Evaluator需要实现 init、iterate、terminatePartial、merge、terminate这几个函数

    1）init函数类似于构造函数，用于UDAF的初始化
    
    2）iterate接收传入的参数，并进行内部的轮转。其返回类型为boolean
    
    3）terminatePartial无参数，其为iterate函数轮转结束后，返回乱转数据，iterate和terminatePartial类似于hadoop的Combiner
    
    4）merge接收terminatePartial的返回结果，进行数据merge操作，其返回类型为boolean
    
    5）terminate返回最终的聚集函数结果  
---------------------

## hive hbase 关联
```sql
create 'flash_people','info','label'

create external table if not exists hm2.flash_people(
guid string comment "people guid",
firsttime string comment "首次入库时间",
ip string comment "用户ip",
jstimestamp bigint comment "json时间戳"
)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping" = ":key,info:firsttime,info:ip,:timestamp")
tblproperties("hbase.table.name" = "hm2:flash_people");
```
## hive -e 用法
`hive -e`主要用来在命令行执行`sql`
```bash
hive -e 'set mapred.reduce.tasks = 30;insert into hm2.flash_people select guid,dt,remote_addr,(32523145869-ngx_timestamp) from hm2.data where dt = "2018-07-01" and length(guid) = 38 and ngx_timestamp is not null and ngx_timestamp != '' and ngx_timestamp is regexp '\\d{8}' and remote_addr is not null and remote_addr != '';'
set mapred.reduce.tasks = 30;insert into hm2.flash_people select guid,dt,remote_addr,(32523145869-ngx_timestamp) from hm2.data where dt = "2018-07-01" and length(guid) = 38 and ngx_timestamp is not null and ngx_timestamp != '' and ngx_timestamp rlike'^\\d+$' and remote_addr is not null and remote_addr != '';
```

hive一些优化参数
```sql
set hive.auto.convert.join = false;
set hive.ignore.mapjoin.hint=false;
set hive.exec.parallel=true;
set mapred.reduce.tasks = 60;
```

## 字段变更
### 添加字段
```
alter table hm2.helper add columns(country string, province string, city string);
```
>hive添加字段后，前面数据会有空值，就算将前面数据hdfs文件删除，重新导入，仍然查询出来是 NULL，这个问题有待解决。

- [ ] 解决
- [x] 未解决

添加 map 复杂类型字段
```
alter table hm2.helper add columns(data_map map<string, string>);

hive> desc formatted hm2.helper;
OK
# col_name            	data_type           	comment             
	 	 
time                	string              	                    
uuid                	string              	                    
country             	string              	                    
province            	string              	                    
city                	string              	                    
data_map            	map<string,string>  	                    
	 	 
# Partition Information	 	 
# col_name            	data_type           	comment             
	 	 
dt                  	string              	                    
hour                	string              	                    
msgtype             	string              	                    
	 	 
# Detailed Table Information	 	 
Database:           	hm2                 	 
Owner:              	hadoop              	 
CreateTime:         	Wed Apr 24 10:12:30 CST 2019	 
LastAccessTime:     	UNKNOWN             	 
Protect Mode:       	None                	 
Retention:          	0                   	 
Location:           	hdfs://nameser/hive/warehouse/hm2.db/helper	 
Table Type:         	EXTERNAL_TABLE      	 
Table Parameters:	 	 
	EXTERNAL            	TRUE                
	last_modified_by    	hadoop              
	last_modified_time  	1556072221          
	transient_lastDdlTime	1556072221          
	 	 
# Storage Information	 	 
SerDe Library:      	org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe	 
InputFormat:        	org.apache.hadoop.mapred.TextInputFormat	 
OutputFormat:       	org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat	 
Compressed:         	No                  	 
Num Buckets:        	-1                  	 
Bucket Columns:     	[]                  	 
Sort Columns:       	[]                  	 
Storage Desc Params:	 	 
	field.delim         	\t                  
	serialization.format	\t                  
Time taken: 0.105 seconds, Fetched: 59 row(s)
```
仿照一张在创建表时定义了map类型字段的表的属性描述
```
Storage Desc Params:	 	 
	colelction.delim    	,                   
	field.delim         	\t                  
	mapkey.delim        	:                   
	serialization.format	\t                  
```
只需要将map属性修改为：
```
hive> alter table hm2.helper set serdeproperties('colelction.delim' = ',', 'mapkey.delim' = ':');
OK
Time taken: 0.132 seconds
```
那么
```
hive> desc formatted hm2.helper;
OK
# col_name            	data_type           	comment             
	 	 
time                	string              	                    
uuid                	string              	                    
country             	string              	                    
province            	string              	                    
city                	string              	                    
data_map            	map<string,string>  	                    
	 	 
# Partition Information	 	 
# col_name            	data_type           	comment             
	 	 
dt                  	string              	                    
hour                	string              	                    
msgtype             	string              	                    
	 	 
# Detailed Table Information	 	 
Database:           	hm2                 	 
Owner:              	hadoop              	 
CreateTime:         	Wed Apr 24 10:12:30 CST 2019	 
LastAccessTime:     	UNKNOWN             	 
Protect Mode:       	None                	 
Retention:          	0                   	 
Location:           	hdfs://nameser/hive/warehouse/hm2.db/helper	 
Table Type:         	EXTERNAL_TABLE      	 
Table Parameters:	 	 
	EXTERNAL            	TRUE                
	last_modified_by    	hadoop              
	last_modified_time  	1556072669          
	transient_lastDdlTime	1556072669          
	 	 
# Storage Information	 	 
SerDe Library:      	org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe	 
InputFormat:        	org.apache.hadoop.mapred.TextInputFormat	 
OutputFormat:       	org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat	 
Compressed:         	No                  	 
Num Buckets:        	-1                  	 
Bucket Columns:     	[]                  	 
Sort Columns:       	[]                  	 
Storage Desc Params:	 	 
	colelction.delim    	,                   
	field.delim         	\t                  
	mapkey.delim        	:                   
	serialization.format	\t                  
Time taken: 0.079 seconds, Fetched: 61 row(s)
```
即可

### 删除字段
```sql
CREATE TABLE test (
creatingTs BIGINT,
a STRING,
b BIGINT,
c STRING,
d STRING,
e BIGINT,
f BIGINT
);
```
如果需要删除 column f 列，可以使用以下语句：
```sql
ALTER TABLE test REPLACE COLUMNS (
creatingTs BIGINT,
a STRING,
b BIGINT,
c STRING,
d STRING,
e BIGINT
);
```

增加列：
```sql
alter table of_test columns (judegment int)
```

## hive-1.2.1 支持`insert,update,delete`的配置
`hive-site.xml`中添加配置
```xml
<property>
    <name>hive.support.concurrency</name>
    <value>true</value>
</property>
<property>
    <name>hive.exec.dynamic.partition.mode</name>
    <value>nonstrict</value>
</property>
<property>
    <name>hive.txn.manager</name>
    <value>org.apache.hadoop.hive.ql.lockmgr.DbTxnManager</value>
</property>
<property>
    <name>hive.compactor.initiator.on</name>
    <value>true</value>
</property>
<property>
    <name>hive.compactor.worker.threads</name>
    <value>1</value>
</property>
<property>
  <name>hive.enforce.bucketing</name>
  <value>true</value>
</property>
```
建表语句
```sql
create external table if not exists hm2.history_helper
(
guid string,
starttime string,
endtime string,
num int
)
clustered by(guid) into 50 buckets
stored as orc TBLPROPERTIES ('transactional'='true');
```
==说明：建表语句必须带有`into buckets`子句和`stored as orc TBLPROPERTIES ('transactional'='true')`子句，并且不能带有`sorted by`子句。==

这样，这个表就可以就行`insert,update,delete`操作了。

==注意：== 上面规则在 hive-1.2.1 是可以的，在 hive-2.1.1 中需要将`external`关键字去掉，即高版本不支持外部表update了。

## hive表中的锁

**场景：**

在执行`insert into`或insert overwrite任务时，中途手动将程序停掉，会出现卡死情况（无法提交MapReduce），只能执行查询操作，而`drop insert`操作均不可操作，无论执行多久，都会保持卡死状态

临时解决办法是……把表名换一个…… 

根本原因是：hive表被锁或者某个分区被锁，需要解锁
```
show locks 表名：
```
可以查看表被锁的情况

**解锁**
```
unlock table 表名;  -- 解锁表
unlock table 表名 partition(dt='2014-04-01');  -- 解锁某个分区
```
**注**意

表锁和分区锁是两个不同的锁，对表解锁，对分区是无效的，分区需要单独解锁


高版本hive默认插入数据时，不能查询，因为有锁
```
hive> show locks;
OK
test@helper	EXCLUSIVE
```
解决办法：关闭锁机制
```sql
set hive.txn.manager=org.apache.hadoop.hive.ql.lockmgr.DbTxnManager; // 这是默认值
set hive.support.concurrency=false; 默认为true
```

## 基本知识
查看表结构信息
```
 desc formatted table_name;
 desc table_name;
```

## 导入数据到hive表

### load命令
```
hive -e 'load data inpath "/data/MROutput/hm2_data_gamelabel_output2/part-r-*" into table hm2.game_label partition(dt="2018-10-11");'
```
==注意==：inpath 后接的hdfs路径需要引号

python中的hive命令字符串示例：
```
cmd = 'hive -e \'load data inpath "%s/part-r-*" into table hm2.game_label partition(dt=%s);\''%(outpath, formatDate(day))
```

### orc格式表
- hive创建orc格式表不能像`textfile`格式一样直接load数据到表中，一般需要load创建临时textfile表，然后通过insert into 或者`insert overwrite`到orc存储格式表中。
- 或者将现有orc文件cp到`hive`对应表的目录
## map,reduce知识
什么情况下只有一个reduce？ 

很多时候你会发现任务中不管数据量多大，不管你有没有设置调整reduce个数的参数，任务中一直都只有一个reduce任务；其实只有一个reduce任务的情况，除了数据量小于```hive.exec.reducers.bytes.per.reducer```参数值的情况外，还有以下原因：
1. 没有group by的汇总，比如把select pt,count(1) from popt_tbaccountcopy_mes where pt = '2012-07-04' group by pt; 写成 select count(1) from popt_tbaccountcopy_mes where pt = '2012-07-04'; 
这点非常常见，希望大家尽量改写。 
1. 用了Order by 
1. 有笛卡尔积 
通常这些情况下，除了找办法来变通和避免，我暂时没有什么好的办法，因为这些操作都是全局的，所以hadoop不得不用一个reduce去完成； 
同样的，在设置reduce个数的时候也需要考虑这两个原则：使大数据量利用合适的reduce数；使单个reduce任务处理合适的数据量。


## hive 优化
1. hive mapreduce参数优化
设置map,reduce任务分配的资源
```
set mapreduce.map.memory.mb = 4096 ;
set mapreduce.reduce.memory.mb = 4096 ; 
set mapreduce.map.java.opts=-Xmx3686m;
set mapreduce.reduce.java.opts=-Xmx3428m;
```
2. hive.exec.parallel参数控制在同一个sql中的不同的job是否可以同时运行,默认为false.

下面是对于该参数的测试过程:

测试sql:
```sql
select r1.a 
from 
(select t.a from sunwg_10 t join sunwg_10000000 s on t.a=s.b) r1 
join 
(select s.b from sunwg_100000 t join sunwg_10 s on t.a=s.b) r2 
on (r1.a=r2.b);
```
- 当参数为false的时候,三个job是顺序的执行
```
set hive.exec.parallel=false;
```
- 但是可以看出来其实两个子查询中的sql并无关系,可以并行的跑
```
set hive.exec.parallel=true;
```
3. 设置reduce个数
```
set mapred.reduce.tasks = 15;
```
==总结==:
在资源充足的时候hive.exec.parallel会让那些存在并发job的sql运行得更快,但同时消耗更多的资源
可以评估下hive.exec.parallel对我们的刷新任务是否有帮助.

4. 参数设置
```
set mapred.max.split.size=256000000;        -- 决定每个map处理的最大的文件大小，单位为B
```

5. orc 小文件合并
```sql
set hive.execution.engine = mr; # 不是必须的
alter table hm3.hm3_format_log partition (dt="2019-09-17", hour="16", msgtype="web", action="image") concatenate;
```

6. msck repair修复大量分区
```sql
set hive.msck.path.validation=ignore;
msck repair table tbName;
```

## hive on spark 知识
cdh 6.0.1 下通过设置：
```
set hive.execution.engine=spark;
```
也可以将默认的`application`执行引擎切换为`spark`；
[apache hadoop 下配置 hive on spark](https://www.cnblogs.com/xinfang520/p/7684605.html)


### 参数调优
了解完了Spark作业运行的基本原理之后，对资源相关的参数就容易理解了。所谓的Spark资源参数调优，其实主要就是对Spark运行过程中各个使用资源的地方，通过调节各种参数，来优化资源使用的效率，从而提升Spark作业的执行性能。以下参数就是Spark中主要的资源参数，每个参数都对应着作业运行原理中的某个部分。

**num-executors/spark.executor.instances**

- 参数说明：该参数用于设置Spark作业总共要用多少个Executor进程来执行。Driver在向YARN集群管理器申请资源时，YARN集群管理器会尽可能按照你的设置来在集群的各个工作节点上，启动相应数量的Executor进程。这个参数非常之重要，如果不设置的话，默认只会给你启动少量的Executor进程，此时你的Spark作业的运行速度是非常慢的。
- 参数调优建议：每个Spark作业的运行一般设置50~100个左右的Executor进程比较合适，设置太少或太多的Executor进程都不好。设置的太少，无法充分利用集群资源；设置的太多的话，大部分队列可能无法给予充分的资源。

**executor-memory/spark.executor.memory**

- 参数说明：该参数用于设置每个Executor进程的内存。Executor内存的大小，很多时候直接决定了Spark作业的性能，而且跟常见的JVM OOM异常，也有直接的关联。
- 参数调优建议：每个Executor进程的内存设置4G8G较为合适。但是这只是一个参考值，具体的设置还是得根据不同部门的资源队列来定。可以看看自己团队的资源队列的最大内存限制是多少，num-executors乘以executor-memory，是不能超过队列的最大内存量的。此外，如果你是跟团队里其他人共享这个资源队列，那么申请的内存量最好不要超过资源队列最大总内存的1/31/2，避免你自己的Spark作业占用了队列所有的资源，导致别的同学的作业无法运行。

**executor-cores/spark.executor.cores**

- 参数说明：该参数用于设置每个Executor进程的CPU core数量。这个参数决定了每个Executor进程并行执行task线程的能力。因为每个CPU core同一时间只能执行一个task线程，因此每个Executor进程的CPU core数量越多，越能够快速地执行完分配给自己的所有task线程。
- 参数调优建议：Executor的CPU core数量设置为2~4个较为合适。同样得根据不同部门的资源队列来定，可以看看自己的资源队列的最大CPU core限制是多少，再依据设置的Executor数量，来决定每个Executor进程可以分配到几个CPU core。同样建议，如果是跟他人共享这个队列，那么num-executors * executor-cores不要超过队列总CPU core的1/3~1/2左右比较合适，也是避免影响其他同学的作业运行。

**driver-memory**

- 参数说明：该参数用于设置Driver进程的内存。
- 参数调优建议：Driver的内存通常来说不设置，或者设置1G左右应该就够了。唯一需要注意的一点是，如果需要使用collect算子将RDD的数据全部拉取到Driver上进行处理，那么必须确保Driver的内存足够大，否则会出现OOM内存溢出的问题。

**spark.default.parallelism**

- 参数说明：该参数用于设置每个stage的默认task数量。这个参数极为重要，如果不设置可能会直接影响你的Spark作业性能。
- 参数调优建议：Spark作业的默认task数量为500~1000个较为合适。很多同学常犯的一个错误就是不去设置这个参数，那么此时就会导致Spark自己根据底层HDFS的block数量来设置task的数量，默认是一个HDFS block对应一个task。通常来说，Spark默认设置的数量是偏少的（比如就几十个task），如果task数量偏少的话，就会导致你前面设置好的Executor的参数都前功尽弃。试想一下，无论你的Executor进程有多少个，内存和CPU有多大，但是task只有1个或者10个，那么90%的Executor进程可能根本就没有task执行，也就是白白浪费了资源！因此Spark官网建议的设置原则是，设置该参数为num-executors * executor-cores的2~3倍较为合适，比如Executor的总CPU core数量为300个，那么设置1000个task是可以的，此时可以充分地利用Spark集群的资源。

**spark.storage.memoryFraction**

- 参数说明：该参数用于设置RDD持久化数据在Executor内存中能占的比例，默认是0.6。也就是说，默认Executor 60%的内存，可以用来保存持久化的RDD数据。根据你选择的不同的持久化策略，如果内存不够时，可能数据就不会持久化，或者数据会写入磁盘。
- 参数调优建议：如果Spark作业中，有较多的RDD持久化操作，该参数的值可以适当提高一些，保证持久化的数据能够容纳在内存中。避免内存不够缓存所有的数据，导致数据只能写入磁盘中，降低了性能。但是如果Spark作业中的shuffle类操作比较多，而持久化操作比较少，那么这个参数的值适当降低一些比较合适。此外，如果发现作业由于频繁的gc导致运行缓慢（通过spark web ui可以观察到作业的gc耗时），意味着task执行用户代码的内存不够用，那么同样建议调低这个参数的值。

**spark.shuffle.memoryFraction**
- 参数说明：该参数用于设置shuffle过程中一个task拉取到上个stage的task的输出后，进行聚合操作时能够使用的Executor内存的比例，默认是0.2。也就是说，Executor默认只有20%的内存用来进行该操作。shuffle操作在进行聚合时，如果发现使用的内存超出了这个20%的限制，那么多余的数据就会溢写到磁盘文件中去，此时就会极大地降低性能。
- 参数调优建议：如果Spark作业中的RDD持久化操作较少，shuffle操作较多时，建议降低持久化操作的内存占比，提高shuffle操作的内存占比比例，避免shuffle过程中数据过多时内存不够用，必须溢写到磁盘上，降低了性能。此外，如果发现作业由于频繁的gc导致运行缓慢，意味着task执行用户代码的内存不够用，那么同样建议调低这个参数的值。


[原文链接](https://www.jianshu.com/p/577a3601d7ef)

---

**Spark On Yarn执行中executor内存限制问题**

解决Spark On Yarn执行中executor内存限制问题

集群版本 Spark 2.2.0 + Hadoop 3.0-CDH6.0.1

hive on saprk , 设置：
```
hive> set hive.execution.engine=spark;
hive> set spark.executor.memory=31.5g;
hive> set spark.executor.cores=11;
hive> set spark.serializer=org.apache.spark.serializer.KryoSerializer;
```

提示内存不足

```
Failed to execute spark task, with exception 'org.apache.hadoop.hive.ql.metadata.HiveException(Failed to create Spark client for Spark session 50288c8b-96aa-44ad-9eea-3cb4abb1ae5b)'
FAILED: Execution Error, return code 30041 from org.apache.hadoop.hive.ql.exec.spark.SparkTask. Failed to create Spark client for Spark session 50288c8b-96aa-44ad-9eea-3cb4abb1ae5b
```

解决方案，修改`Yarn`的配置文件：

1、```yarn.nodemanager.resource.memory-mb``` 容器内存

设置为 至少 : ```executor-memory(15g) + driver(512m)```的内存，如上例可配置为 16g

2、```yarn.scheduler.maximum-allocation-mb``` 最大容器内存

设置为 至少 : ```executor-memory(15g) + driver(512m)```的内存，如上例可配置为 16g

第一个参数为```NodeManager```的配置 ，第二个参数为 ```ResourceManager```的配置。

## 字符串处理
1 字符串连接：
```
concat(str, str2, str3,...) 字符串连接
concat_ws(separator, str, str2, str3, ...) 将字符串用separator作为间隔连接起来
```
2 字符串截取
```
substr(s, 0, 1) 截取第一个字符
substr(s, -1) 截取最后一个字符
```
3 字符串urldecode
```sql
reflect("java.net.URLDecoder", "decode", trim(originalfilename), "UTF-8")
```

## hive 中 join

`mapjoin`的优化在于，在`mapreduce task`开始之前，创建一个`local task`,小表以`hshtable`的形式加载到内存，然后序列化到磁盘，把内存的`hashtable`压缩为`tar`文件。然后把文件分发到 `Hadoop Distributed Cache`，然后传输给每一个`mapper`，`mapper`在本地反序列化文件并加载进内存在做`join`

sql
```sql
select workflow,count(workflow) from (select guid, substr(workflow, -1) workflow from hm2.workflow_list) m right join hm2.helper helper on m.guid = helper.guid and helper.dt = "2018-10-21" group by workflow;
```
内存溢出解决办法:
```
set hive.auto.convert.join = false;
set hive.ignore.mapjoin.hint=false;
set hive.exec.parallel=true;
```

### Hive中Join的原理和机制
笼统的说，Hive中的Join可分为Common Join（Reduce阶段完成join）和Map Join（Map阶段完成join）。本文简单介绍一下两种join的原理和机制。
### Hive Common Join
如果不指定MapJoin或者不符合MapJoin的条件，那么Hive解析器会将Join操作转换成Common Join,即：在Reduce阶段完成join.
整个过程包含Map、Shuffle、Reduce阶段。

- Map阶段<br>

读取源表的数据，Map输出时候以Join on条件中的列为key，如果Join有多个关联键，则以这些关联键的组合作为key;
Map输出的value为join之后所关心的(select或者where中需要用到的)列；同时在value中还会包含表的Tag信息，用于标明此value对应哪个表；
按照key进行排序

- Shuffle阶段<br>
根据key的值进行hash,并将key/value按照hash值推送至不同的reduce中，这样确保两个表中相同的key位于同一个reduce中

- Reduce阶段<br>
根据key的值完成join操作，期间通过Tag来识别不同表中的数据。

以下面的HQL为例，图解其过程：
```
SELECT 
 a.id,a.dept,b.age 
FROM a join b 
ON (a.id = b.id);
```
### Hive Map Join
MapJoin通常用于一个很小的表和一个大表进行join的场景，具体小表有多小，由参数hive.mapjoin.smalltable.filesize来决定，该参数表示小表的总大小，默认值为25000000字节，即25M。
Hive0.7之前，需要使用hint提示 /*+ mapjoin(table) */才会执行MapJoin,否则执行Common Join，但在0.7版本之后，默认自动会转换Map Join，由参数hive.auto.convert.join来控制，默认为true.
仍然以9.1中的HQL来说吧，假设a表为一张大表，b为小表，并且hive.auto.convert.join=true,那么Hive在执行时候会自动转化为MapJoin。

如图中的流程，首先是Task A，它是一个Local Task（在客户端本地执行的Task），负责扫描小表b的数据，将其转换成一个HashTable的数据结构，并写入本地的文件中，之后将该文件加载到DistributeCache中，该HashTable的数据结构可以抽象为：

| key  | value |
| ---- | ----- |
| 1    | 26    |
| 2    | 34    |
- 接下来是Task B，该任务是一个没有Reduce的MR，启动MapTasks扫描大表a,在Map阶段，根据a的每一条记录去和DistributeCache中b表对应的HashTable关联，并直接输出结果。
- 由于MapJoin没有Reduce，所以由Map直接输出结果文件，有多少个Map Task，就有多少个结果文件。

[原文链接](http://lxw1234.com/archives/2015/06/313.htm)

## 转义字符
```
hive> select split('a:1||b:2||c:3','\\|\\|') from hm2.test;
OK
["a:1","b:2","c:3"]
["a:1","b:2","c:3"]
["a:1","b:2","c:3"]
["a:1","b:2","c:3"]
```
其它转义字符还有`{`, `[`

## insert table select from
```sql
insert into tbName select * from tbName2;
insert overwrite table tbName select * from tbName2;
```
insert overwrite例子
```sql
insert overwrite table hm2.helper partition(dt = '2018-06-22', hour = '09',msgtype = 'helper') select time,source,remote_addr,remote_user,body_bytes_sent,request_time,status,host,request_method,http_referrer,http_x_forwarded_for,http_user_agent,upstream_response_time,upstream_addr,guid,helperversion,osversion,ngx_timestamp,get_type,split(ip2area(http_x_forwarded_for,remote_addr), "\t")[0] country,split(ip2area(http_x_forwarded_for,remote_addr), "\t")[1] province,split(ip2area(http_x_forwarded_for,remote_addr), "\t")[2] city from hm2.helper where dt = '2018-06-22' and hour = '09' and msgtype = 'helper';
```
插入分区表，不用指定分区，可以自动识别
```
INSERT overwrite TABLE test.dis_helper PARTITION (dt,hour,msgtype) select `(num)?+.+` from (select *,row_number() over (partition by guid order by time asc) num from hm2.helper where dt ='2018-09-06'and hour between '00' and '23' and msgtype='helper') t where t.num=1;
```
这里把数据去重，插入分区表test.dis_helper中，自动根据dt,hour,msgtype字段的取值进入分区表，并且````(num)?+.+````表示除了```num```这个字段。

## explain 查看执行计划
对于这么一个插叙sql
```
explain
select
workflow,count(workflow) cou from (select guid, split(split(workflow,"\\|\\|")[0], ":")[1] workflow
from 
hm2.workflow_list) m
inner join
hm2.flash flash
on m.guid = flash.guid and flash.dt = "2018-11-04"
group by workflow order by cou;
```
可以打印出执行计划
```
STAGE DEPENDENCIES:
  Stage-1 is a root stage
  Stage-2 depends on stages: Stage-1
  Stage-3 depends on stages: Stage-2
  Stage-0 depends on stages: Stage-3

STAGE PLANS:
  Stage: Stage-1
    Map Reduce
      Map Operator Tree:
          TableScan
            alias: workflow_list
            Statistics: Num rows: 1 Data size: 0 Basic stats: PARTIAL Column stats: NONE
            Filter Operator
              predicate: guid is not null (type: boolean)
              Statistics: Num rows: 1 Data size: 0 Basic stats: PARTIAL Column stats: NONE
              Select Operator
                expressions: guid (type: string), split(split(workflow, '\|\|')[0], ':')[1] (type: string)
                outputColumnNames: _col0, _col1
                Statistics: Num rows: 1 Data size: 0 Basic stats: PARTIAL Column stats: NONE
                Reduce Output Operator
                  key expressions: _col0 (type: string)
                  sort order: +
                  Map-reduce partition columns: _col0 (type: string)
                  Statistics: Num rows: 1 Data size: 0 Basic stats: PARTIAL Column stats: NONE
                  value expressions: _col1 (type: string)
          TableScan
            alias: flash
            Statistics: Num rows: 489153811 Data size: 48915382339 Basic stats: COMPLETE Column stats: NONE
            Filter Operator
              predicate: guid is not null (type: boolean)
              Statistics: Num rows: 244576906 Data size: 24457691219 Basic stats: COMPLETE Column stats: NONE
              Reduce Output Operator
                key expressions: guid (type: string)
                sort order: +
                Map-reduce partition columns: guid (type: string)
                Statistics: Num rows: 244576906 Data size: 24457691219 Basic stats: COMPLETE Column stats: NONE
      Reduce Operator Tree:
        Join Operator
          condition map:
               Inner Join 0 to 1
          keys:
            0 _col0 (type: string)
            1 guid (type: string)
          outputColumnNames: _col1
          Statistics: Num rows: 269034602 Data size: 26903460924 Basic stats: COMPLETE Column stats: NONE
          Group By Operator
            aggregations: count(_col1)
            keys: _col1 (type: string)
            mode: hash
            outputColumnNames: _col0, _col1
            Statistics: Num rows: 269034602 Data size: 26903460924 Basic stats: COMPLETE Column stats: NONE
            File Output Operator
              compressed: false
              table:
                  input format: org.apache.hadoop.mapred.SequenceFileInputFormat
                  output format: org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat
                  serde: org.apache.hadoop.hive.serde2.lazybinary.LazyBinarySerDe

  Stage: Stage-2
    Map Reduce
      Map Operator Tree:
          TableScan
            Reduce Output Operator
              key expressions: _col0 (type: string)
              sort order: +
              Map-reduce partition columns: _col0 (type: string)
              Statistics: Num rows: 269034602 Data size: 26903460924 Basic stats: COMPLETE Column stats: NONE
              value expressions: _col1 (type: bigint)
      Reduce Operator Tree:
        Group By Operator
          aggregations: count(VALUE._col0)
          keys: KEY._col0 (type: string)
          mode: mergepartial
          outputColumnNames: _col0, _col1
          Statistics: Num rows: 134517301 Data size: 13451730462 Basic stats: COMPLETE Column stats: NONE
          File Output Operator
            compressed: false
            table:
                input format: org.apache.hadoop.mapred.SequenceFileInputFormat
                output format: org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat
                serde: org.apache.hadoop.hive.serde2.lazybinary.LazyBinarySerDe

  Stage: Stage-3
    Map Reduce
      Map Operator Tree:
          TableScan
            Reduce Output Operator
              key expressions: _col1 (type: bigint)
              sort order: +
              Statistics: Num rows: 134517301 Data size: 13451730462 Basic stats: COMPLETE Column stats: NONE
              value expressions: _col0 (type: string)
      Reduce Operator Tree:
        Select Operator
          expressions: VALUE._col0 (type: string), KEY.reducesinkkey0 (type: bigint)
          outputColumnNames: _col0, _col1
          Statistics: Num rows: 134517301 Data size: 13451730462 Basic stats: COMPLETE Column stats: NONE
          File Output Operator
            compressed: false
            Statistics: Num rows: 134517301 Data size: 13451730462 Basic stats: COMPLETE Column stats: NONE
            table:
                input format: org.apache.hadoop.mapred.TextInputFormat
                output format: org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat
                serde: org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe

  Stage: Stage-0
    Fetch Operator
      limit: -1
      Processor Tree:
        ListSink
```

## 常用 sql 查询
常用SQL查询。
#### explode 一行变多行
`explode`主要是将一行数据按照某个字段变为多行。
```sql
select mt.impress,count(mt.guid) from (select (case when t.impressed = '' or t.impressed is null then 'null' else t.impressed end) impress,t.guid from (select guid,impressed from hm2.install lateral view explode(split(replace(replace(offer_impressed,'[',''),']',''), ',')) test_alias as impressed where dt='2018-12-17') t) mt group by mt.impress;
```
#### 分组后取 top N
`row_number() over(partition by)`

表`hm4.ffbrowser_domain_url_2019_07_17`数据如下：
```
www.baidu.com	https://www.baidu.com/link?url=fCBMjU_THQzoAdo0RTAQLQIfVWIlPlgzpEM5Pk5qChKiahWFfMzFo6ckRjd9OFc7w8cj5h8esZXKBab5WqeLPgDYipewXUdz9LFPf-oxOfK&wd=&eqid=d2c80e8e0033424b000000045d2f14e9	1
www.baidu.com	https://www.baidu.com/link?url=3jlP1HLS4aYSnN1L5CG3pPr0zXnqBpDdG8mlFQm47w1RcHFwHiBU0t8Hi0UrMD37lSJvQkGWQ3iBtNpc0AJhEei-v8MdGKgRnVqy62tuCA_&wd=&eqid=e3d5845b002790f7000000045d2f0af8	1
www.baidu.com	http://www.baidu.com/	1
www.baidu.com	https://www.baidu.com/link?url=8S_ziMFwpClJww3C15iXu__wqMrMOxnYuDnZDpQWnbs1PTTqx_wwjIY7QsrFfaKT&wd=&eqid=eb8f38f70039cf51000000045d2e8716	1
www.baidu.com	https://www.baidu.com/link?url=AZvluWbTjZjpaT5lnIpkB-gTIdyX_nZdtoLX_pkbM5i&wd=&eqid=8b09c549000038e7000000035930ca9a	1
www.baidu.com	https://www.baidu.com/link?url=IjStquL7c4YwVDk1zQJFYkwBiGY20Kv2PQsXuTQTHH0BhAPLjUaz-XhLLp5Zfe3fE4hU_KNfEs6JxyESkwGlea&wd=&eqid=e7e404c9000012b1000000045d2ee845	1
www.baidu.com	https://www.baidu.com/link?url=qRaLKHc_ZZIskkWF_f6azkmHqRlfgmuRQZcrzRovBC5MEBR5yTIG20FiR3O__8Jz&wd=&eqid=e13f05290018e7fb000000045d2eed7d	2
www.baidu.com	https://www.baidu.com/s?tn=50000201_hao_pg&usm=3&wd=%E7%AB%AF%E7%81%AB%E9%94%85%E6%B3%BC%E5%A6%BB%E5%AD%90%E5%90%8C%E5%AD%A6&ie=utf-8&rsv_cq=%E4%BA%BA%E7%B1%BB%E7%99%BB%E6%9C%8850%E5%91%A8%E5%B9%B4&rsv_dl=0_right_fyb_pchot_20811_01&rsf=531e82477396136261c6275e8afa58b1_1_10_8&rqid=e3d5845b002790f7	1
www.baidu.com	https://www.baidu.com/link?url=NHmzZVrcbQ1tf6JnR4MJlHXJZFy-4RMgKwjNeDvskMyl17vpdi_8XgVCdRvGFtU2WJpNpHQf4VbwIeQi5qDHskDTrDUK5KMUkrkfKcWYxhy&wd=&eqid=e3d5845b002790f7000000045d2f0af8	1
www.baidu.com	https://www.baidu.com/s?wd=%E5%BE%AE%E5%8D%9A&ie=utf-8	1
```
sql查询语句
```sql
select a.domain,a.url,a.cou from (select domain,url,cou,row_number() over(partition by domain order by cou desc) as n from hm4.ffbrowser_domain_url_2019_07_17)a where a.n <= 2;
```
结果
```
www.baidu.com	https://www.baidu.com/	69
www.baidu.com	https://www.baidu.com/link?url=r6riiF-vxG9OX70KBVx86FuywJYXHu-TpTTSEst9ggK78xIjVvkI_QoS9tEDBAqq&wd=&eqid=ba409e160014f8c9000000045eec97	3
```
**相似问题**：每门课程成绩的前N名

#### case when 用法
```sql
select reginlistlength, softname, sum(cou) from (select (case when reginlistlength > '1' then 'more' else reginlistlength end) as reginlistlength, softname, count(l.guid) cou from (select reginlistlength, softname, guid from hm2.lnk where dt = '2019-05-24' and softtype = '1' group by guid,reginlistlength, softname, guid)l where l.guid in (select guid from hm2.helper where dt = '2019-05-24' group by guid) group by reginlistlength, softname)m group by reginlistlength, softname;
```
```sql
select (case when reginlistlength > '1' then 'more' else reginlistlength end) as reginlistlength, softname, count(l.guid) cou from (select reginlistlength, softname, guid from hm2.lnk where dt = '2019-05-24' and softtype = '1' group by guid,reginlistlength, softname, guid)l where l.guid in (select guid from hm2.helper where dt = '2019-05-24' group by guid) group by reginlistlength, softname;
```