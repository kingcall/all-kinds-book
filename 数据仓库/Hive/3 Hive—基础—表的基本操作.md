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

