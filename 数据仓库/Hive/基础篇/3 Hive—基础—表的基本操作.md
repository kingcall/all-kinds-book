[TOC]

## 1. 创建表

`create table`语句遵从`sql`语法习惯，只不过`Hive`的语法更灵活。例如，可以定义表的数据文件存储位置，使用的存储格式等。

简单一点就是这样的`CREATE TABLE pokes (foo INT, bar STRING);` 除了必要的信息外，其他的都可以使用默认的信息

**分区表**

```sql
create table if not exists test.user1(
name string comment 'name',
salary float comment 'salary',
address struct<country:string, city:string> comment 'home address'
)
comment 'description of the table'
partitioned by (age int)
row format delimited fields terminated by '\t'
stored as orc;
```

没有指定`external`关键字，则为`管理表`，跟`mysql`一样，`if not exists`如果表存在则不做操作，否则则新建表。`comment`可以为其做注释，分区列为`age`年龄，列之间分隔符是`\t`，存储格式为列式存储`orc`，存储位置为默认位置，即参数`hive.metastore.warehouse.dir`（默认：/user/hive/warehouse）指定的`hdfs`目录。

**分桶表**

其实我们可以大致的将表分为，普通表、分区表、和分桶表，下面我们演示一下分桶表，至于分区表和分桶表之间的区别我们后面会单独讲,下面我们创建一个**分桶表**

```sql
CREATE TABLE par_table(
  viewTime INT,
  userid BIGINT,
  page_url STRING, referrer_url STRING,
  ip STRING COMMENT 'IP Address of the User'
)
COMMENT 'This is the page view table'
PARTITIONED BY(date STRING, pos STRING)
CLUSTERED BY(userid) SORTED BY(viewTime) INTO 32 BUCKETS
ROW FORMAT DELIMITED ‘\t’
  FIELDS TERMINATED BY '\n'
STORED AS SEQUENCEFILE;
```



## 2. 拷贝表

使用`like`可以拷贝一张跟原表结构一样的空表，**里面是没有数据的**。

```sql
create table if not exists test.user2 like test.user1;
```

当然你也可以根据查询出来的数据创建表

```sql
create table user_behavior_copy as select * from user_behavior limit 10;
```

## 3. 查看表结构

通过`desc [可选参数] tableName`命令查看表结构，可以看出拷贝的表`test.user1`与原表`test.user1`的表结构是一样的。

```sql
hive> desc test.user2;
OK
name                	string              	name                
salary              	float               	salary              
address             	struct<country:string,city:string>	home address        
age                 	int                 	                    
	 	 
# Partition Information	 	 
# col_name            	data_type           	comment             
	 	 
age                 	int                 	                    
```

也可以加`formatted`，可以看到更加详细和冗长的输出信息。

```sql
hive> desc formatted test.user2;
OK
# col_name            	data_type           	comment             
	 	 
name                	string              	name                
salary              	float               	salary              
address             	struct<country:string,city:string>	home address        
	 	 
# Partition Information	 	 
# col_name            	data_type           	comment             
	 	 
age                 	int                 	                    
	 	 
# Detailed Table Information	 	 
Database:           	test                	 
Owner:              	hdfs                	 
CreateTime:         	Mon Dec 21 16:37:57 CST 2020	 
LastAccessTime:     	UNKNOWN             	 
Retention:          	0                   	 
Location:           	hdfs://nameservice2/user/hive/warehouse/test.db/user2	 
Table Type:         	MANAGED_TABLE       	 
Table Parameters:	 	 
	COLUMN_STATS_ACCURATE	{\"BASIC_STATS\":\"true\"}
	numFiles            	0                   
	numPartitions       	0                   
	numRows             	0                   
	rawDataSize         	0                   
	totalSize           	0                   
	transient_lastDdlTime	1608539877          
	 	 
# Storage Information	 	 
SerDe Library:      	org.apache.hadoop.hive.ql.io.orc.OrcSerde	 
InputFormat:        	org.apache.hadoop.hive.ql.io.orc.OrcInputFormat	 
OutputFormat:       	org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat	 
Compressed:         	No                  	 
Num Buckets:        	-1                  	 
Bucket Columns:     	[]                  	 
Sort Columns:       	[]                  	 
Storage Desc Params:	 	 
	field.delim         	\t                  
	serialization.format	\t                  
```

## 4. 删除表

这跟`sql`中删除命令`drop table`是一样的：

```sql
drop table if exists table_name;
```

对于管理表（内部表），直接把表彻底删除了；对于外部表，还需要删除对应的`hdfs`文件才会彻底将这张表删除掉，为了安全，通常`hadoop`集群是开启回收站功能的，删除外表表的数据就在回收站，后面如果想恢复也是可以恢复的，直接从回收站`mv`到`hive`对应目录即可。

## 5. 修改表

大多数表属性可以通过`alter table`来修改。

### 5.1 表重命名

```sql
alter table test.user1 rename to test.user3;
```

### 5.2 增、修、删分区

**增加分区**使用命令`alter table table_name add partition(...) location hdfs_path`

```sql
alter table test.user2 add if not exists
partition (age = 101) location '/user/hive/warehouse/test.db/user2/part-0000101'
partition (age = 102) location '/user/hive/warehouse/test.db/user2/part-0000102'
```

**修改分区**也是使用`alter table ... set ...`命令

```sql
alter table test.user2 partition (age = 101) set location '/user/hive/warehouse/test.db/user2/part-0000110'
```

**删除分区**命令格式是`alter table tableName drop if exists partition(...)`

```sql
alter table test.user2 drop if exists partition(age = 101)
```

### 5.3 修改列信息

可以对某个字段进行重命名，并修改位置、类型或者注释：

修改前：

```sql
hive> desc user_log;
OK
userid              	string              	                    
time                	string              	                    
url                 	string              	                    
```

修改列名`time`为`times`，并且使用`after`把位置放到`url`之后，本来是在之前的。

```sql
alter table test.user_log
change column time times string
comment 'salaries'
after url;
```

再来看表结构：

```sql
hive> desc user_log;
OK
userid              	string              	                    
url                 	string              	                    
times               	string              	salaries            
```

`time -> times`，位置在`url`之后。

### 5.4 增加列

`hive`也是可以添加列的：

```sql
alter table test.user2 add columns (
birth date comment '生日',
hobby string comment '爱好'
);
```

### 5.5 删除列

删除列不是指定列删除，需要把原有所有列写一遍，要删除的列排除掉即可：

```sql
hive> desc test.user3;
OK
name                	string              	name                
salary              	float               	salary              
address             	struct<country:string,city:string>	home address        
age                 	int                 	                    
	 	 
# Partition Information	 	 
# col_name            	data_type           	comment             
	 	 
age                 	int                 	                    
```

如果要删除列`salary`，只需要这样写：

```sql
alter table test.user3 replace columns(
name string,
address struct<country:string,city:string>
);
```

这里会报错：

```
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. Replacing columns cannot drop columns for table test.user3. SerDe may be incompatible
```

这张`test.user3`表是`orc`格式的，不支持删除，如果是`textfile`格式，上面这种`replace`写法是可以删除列的。通常情况下不会轻易去删除列的，增加列倒是常见。

### 5.6 修改表的属性

可以增加附加的表属性，或者修改属性，但是无法删除属性：

```sql
alter table tableName set tblproperties(
    'key' = 'value'
);
```

举例：这里新建一张表：

```sql
create table t8(time string,country string,province string,city string)
row format delimited fields terminated by '#' 
lines terminated by '\n' 
stored as textfile;
```

这条语句将t8表中的字段分隔符'#'修改成'\t';

```sql
alter table t8 set serdepropertyes('field.delim'='\t');
```

### 5.7  修改表的文件存储格式组织方式

下面这两个命令都修改了表的物理存储属性。

1. ALTER TABLE table_name SET FILEFORMAT file_format 修改表的存储格式
2. ALTER TABLE table_name CLUSTERED BY (col_name, col_name, ...) [SORTEDBY(col_name, ...)] INTO num_buckets BUCKETS 修改表的分桶情况
   

## 6 其他操作

### 6.1. 查看表

查看当前数据库下的全部标

```sql
show tables；
```

查看特定库下的全部表

```sql
 show tables in ods;
```

按正条件（正则表达式）显示表

```sql
 SHOW TABLES '.*s';
```

### 6.2 导入数据

这个一般用于我们加载一下其他地方的数据到hive 表中，例如外部的文本数据、或者是埋点数据

```java
create table ods.user_info (
    user_id int,
    cid string,
    ckid string,
    username string)
    row format delimited
    fields terminated by '\t'
    lines terminated by '\n';
```

**导入本地数据**

`load data local inpath '/Users/liuwenqiang/workspace/hive/user.txt' overwrite into table ods.user_info`

下面是数据，导入数据表的数据格式是：字段之间是tab键分割，行之间是换行。

```
100636	100890	c5c86f4cddc15eb7	王1
100612	100865	97cc70d411c18b6f	王2
100078	100087	ecd6026a15ffddf5	王
```

**导入HDFS数据**

去掉local 关键字就可以了，但是注意导入HDFS 上的数据是move 操作，而本地是copy 操作

`load data inpath '/Users/liuwenqiang/workspace/hive/user.txt' overwrite into table ods.user_info`

其实很多时候你不用load 命令也可以，只要你将准备好的数据放到对应的表目录下就可以了，对于本地文件可以使用 `hdfd dfs -put` 对于集群上的文件使用`hdfs dfs -mv`

### 6.3 导出数据

上面我们介绍使用load 命令可以导入本地数据或者HDFS上的数据到数仓，下面我们看一下如何将查询的结果导出，其实除了下面导出的方式我们还可以直接在HDFS 上的表数据直接进行导出，使用`hdfs dfs -get /user/hive/warehouse/ods.db/person/person.txt`

#### 导出到HDFS

```
INSERT OVERWRITE DIRECTORY '/tmp/hdfs_out' SELECT a.* FROM invites a WHERE a.ds='2008-08-15';
```

#### 导出到本地

```
 INSERT OVERWRITE LOCAL DIRECTORY '/tmp/local_out' SELECT a.* FROM pokes a;
```

