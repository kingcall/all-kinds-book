[TOC]

## 1. 创建表

`create table`语句遵从`sql`语法习惯，只不过`Hive`的语法更灵活。例如，可以定义表的数据文件存储位置，使用的存储格式等。

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

没有指定`external`关键字，则为`管理表`，跟`mysql`一样，`if not exists`如果表存在则不做操作，否则则新建表。`comment`可以为其做注释，分区为`age`年龄，列之间分隔符是`\t`，存储格式为列式存储`orc`，存储位置为默认位置，即参数`hive.metastore.warehouse.dir`（默认：/user/hive/warehouse）指定的`hdfs`目录。

## 2. 拷贝表

使用`like`可以拷贝一张跟原表结构一样的空表，里面是没有数据的。

```sql
create table if not exists test.user2 like test.user1;
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


[TOC]



## Hive 表的基本操作

### DDL 操作



hive> CREATE TABLE pokes (foo INT, bar STRING);


**复杂一下如下：**
**创建外部表：**

CREATE EXTERNAL TABLE page_view(viewTime INT, userid BIGINT,
   page_url STRING, referrer_url STRING,
   ip STRING COMMENT 'IP Address of the User',
   country STRING COMMENT 'country of origination')
COMMENT 'This is the staging page view table'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\054'
STORED AS TEXTFILE
LOCATION '<hdfs_location>';

**建分区表:**

CREATE TABLE par_table(viewTime INT, userid BIGINT,
   page_url STRING, referrer_url STRING,
   ip STRING COMMENT 'IP Address of the User')
COMMENT 'This is the page view table'
PARTITIONED BY(date STRING, pos STRING)
ROW FORMAT DELIMITED ‘\t’
  FIELDS TERMINATED BY '\n'
STORED AS SEQUENCEFILE;

**建Bucket表**

CREATE TABLE par_table(viewTime INT, userid BIGINT,
   page_url STRING, referrer_url STRING,
   ip STRING COMMENT 'IP Address of the User')
COMMENT 'This is the page view table'
PARTITIONED BY(date STRING, pos STRING)
CLUSTERED BY(userid) SORTED BY(viewTime) INTO 32 BUCKETS
ROW FORMAT DELIMITED ‘\t’
  FIELDS TERMINATED BY '\n'
STORED AS SEQUENCEFILE;



**创建表并创建索引字段ds**
hive> CREATE TABLE invites (foo INT, bar STRING) PARTITIONED BY (ds STRING);

**复制一个空表**
CREATE TABLE empty_key_value_store
LIKE key_value_store;

**例子**
create table user_info (user_id int, cid string, ckid string, username string)
row format delimited
fields terminated by '\t'
lines terminated by '\n';
导入数据表的数据格式是：字段之间是tab键分割，行之间是断行。

及要我们的文件内容格式：

100636 100890 c5c86f4cddc15eb7     yyyvybtvt
100612 100865 97cc70d411c18b6f     gyvcycy
100078 100087 ecd6026a15ffddf5     qa000100


**显示所有表**
hive> SHOW TABLES;
按正条件（正则表达式）显示表，
hive> SHOW TABLES '.*s';



### 修改表

**表添加一列 ：**

```
hive> ALTER TABLE pokes ADD COLUMNS (new_col INT);复制代码
```


**添加一列并增加列字段注释**`hive> ALTER TABLE invites ADD COLUMNS (new_col2 INT COMMENT 'a comment');*复制代码*`




**更改表名：**

```
hive> ALTER TABLE events RENAME TO 3koobecaf;复制代码
```


**删除列：**hive> DROP TABLE pokes;




**增加、删除分区**


&#8226;增加

```
ALTER TABLE table_name ADD [IF NOT EXISTS] partition_spec [ LOCATION 'location1' ] partition_spec [ LOCATION 'location2' ] ...
      partition_spec:
  : PARTITION (partition_col = partition_col_value, partition_col = partiton_col_value, ...)复制代码
```


&#8226;删除

```
ALTER TABLE table_name DROP partition_spec, partition_spec,...复制代码
```


**重命名表**&#8226;
`ALTER TABLE table_name RENAME TO new_table_name *复制代码*`


**修改列的名字、类型、位置、注释：**
`ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name]*复制代码*`
&#8226;这个命令可以允许改变列名、数据类型、注释、列位置或者它们的任意组合


**表添加一列 ：**
`hive> ALTER TABLE pokes ADD COLUMNS (new_col INT);*复制代码*`
**添加一列并增加列字段注释**
`hive> ALTER TABLE invites ADD COLUMNS (new_col2 INT COMMENT 'a comment');*复制代码*`
**增加/更新列**
`ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...) *复制代码*`
&#8226; ADD是代表新增一字段，字段位置在所有列后面(partition列前)
   REPLACE则是表示替换表中所有字段。


**增加表的元数据信息**

```
ALTER TABLE table_name SET TBLPROPERTIES table_properties table_properties:
         :[property_name = property_value…..]复制代码
```

&#8226;用户可以用这个命令向表中增加metadata


**改变表文件格式与组织**&#8226;


`ALTER TABLE table_name SET FILEFORMAT file_format ALTER TABLE table_name CLUSTERED BY(userid) SORTED BY(viewTime) INTO num_buckets BUCKETS*复制代码*`这个命令修改了表的物理存储属性

**创建／删除视图**

```
CREATE VIEW [IF NOT EXISTS] view_name [ (column_name [COMMENT column_comment], ...) ][COMMENT view_comment][TBLPROPERTIES (property_name = property_value, ...)] AS SELECT*复制代码*
```

&#8226;增加视图
&#8226;如果没有提供表名，视图列的名字将由定义的SELECT表达式自动生成
&#8226;如果修改基本表的属性，视图中不会体现，无效查询将会失败
&#8226;视图是只读的，不能用LOAD/INSERT/ALTER
&#8226;DROP VIEW view_name
&#8226;删除视图


**创建数据库**
`CREATE DATABASE name*复制代码*`

**显示命令**

```
show tables;
show databases;
show partitions ;
show functions
describe extended table_name dot col_name复制代码
```



### DML

hive不支持用insert语句一条一条的进行插入操作，也不支持update操作。数据是以load的方式加载到建立好的表中。数据一旦导入就不可以修改。

DML包括：INSERT插入、UPDATE更新、DELETE删除

#### 加载数据

```
LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
LOAD DATA LOCAL INPATH './examples/files/kv2.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-15');
 LOAD DATA INPATH '/user/myname/kv2.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-15');
```



#### 查询插入

```
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 FROM from_statement
&#8226;多插入模式
FROM from_statement
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1
[INSERT OVERWRITE TABLE tablename2 [PARTITION ...] select_statement2] ...
&#8226;自动分区模式
INSERT OVERWRITE TABLE tablename PARTITION (partcol1[=val1], partcol2[=val2] ...) select_statement FROM from_statement
```



#### **查询结果写入HDFS文件系统**

```
INSERT OVERWRITE [LOCAL] DIRECTORY directory1 SELECT ... FROM ...
        FROM from_statement
        INSERT OVERWRITE [LOCAL] DIRECTORY directory1 select_statement1
     [INSERT OVERWRITE [LOCAL] DIRECTORY directory2 select_statement2]
```

