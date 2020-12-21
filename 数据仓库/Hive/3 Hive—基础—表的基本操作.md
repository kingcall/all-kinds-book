- [1. 创建表](#1----)
- [2. 拷贝表](#2----)
- [3. 查看表结构](#3------)
- [4. 删除表](#4----)
- [5. 修改表](#5----)
  * [5.1 表重命名](#51-----)
  * [5.2 增、修、删分区](#52--------)
  * [5.3 修改列信息](#53------)
  * [5.4 增加列](#54----)
  * [5.5 删除列](#55----)
  * [5.6 修改表的属性](#56-------)


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