除了使用础的数据类型`string`等，Hive中的列支持使用struct, map, array集合数据类型。

| 数据类型 | 描述                                                         | 语法示例                            |
| -------- | ------------------------------------------------------------ | ----------------------------------- |
| STRUCT   | 和C语言中的struct或者"对象"类似，都可以通过"点"符号访问元素内容。 | struct{'John', 'Doe'}               |
| MAP      | MAP是一组键-值对元素集合，使用key可以访问元素。              | map('fisrt', 'John', 'last', 'Doe') |
| ARRAY    | 数组是一组具有相同数据类型和名称的变量的集合。               | Array('John', 'Doe')                |

## 1. Array的使用
创建数据库表，以array作为数据类型

```sql
create table  person(name string,work_locations array<string>)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
COLLECTION ITEMS TERMINATED BY ',';
```

数据

```
biansutao beijing,shanghai,tianjin,hangzhou
linan changchu,chengdu,wuhan
```

入库数据

```sql
LOAD DATA LOCAL INPATH '/home/hadoop/person.txt' OVERWRITE INTO TABLE person;
```

查询

```
hive> select * from person;
biansutao       ["beijing","shanghai","tianjin","hangzhou"]
linan   ["changchu","chengdu","wuhan"]
Time taken: 0.355 seconds
hive> select name from person;
linan
biansutao
Time taken: 12.397 seconds
hive> select work_locations[0] from person;
changchu
beijing
Time taken: 13.214 seconds
hive> select work_locations from person;   
["changchu","chengdu","wuhan"]
["beijing","shanghai","tianjin","hangzhou"]
Time taken: 13.755 seconds
hive> select work_locations[3] from person;
NULL
hangzhou
Time taken: 12.722 seconds
hive> select work_locations[4] from person;
NULL
NULL
Time taken: 15.958 seconds
```


## 2. Map 的使用
创建数据库表

```sql
create table score(name string, score map<string,int>)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':';
```

要入库的数据

```
biansutao '数学':80,'语文':89,'英语':95
jobs '语文':60,'数学':80,'英语':99
```

入库数据

```sql
LOAD DATA LOCAL INPATH '/home/hadoop/score.txt' OVERWRITE INTO TABLE score;
```

查询

```
hive> select * from score;
biansutao       {"数学":80,"语文":89,"英语":95}
jobs    {"语文":60,"数学":80,"英语":99}
Time taken: 0.665 seconds
hive> select name from score;
jobs
biansutao
Time taken: 19.778 seconds
hive> select t.score from score t;
{"语文":60,"数学":80,"英语":99}
{"数学":80,"语文":89,"英语":95}
Time taken: 19.353 seconds
hive> select t.score['语文'] from score t;
60
89
Time taken: 13.054 seconds
hive> select t.score['英语'] from score t;
99
95
Time taken: 13.769 seconds
```

修改map字段的分隔符

```
Storage Desc Params:	 	 
	colelction.delim    	##                  
	field.delim         	\t                  
	mapkey.delim        	=                   
	serialization.format	\t                  
```

可以通过`desc formatted tableName`查看表的属性。
hive-2.1.1中，可以看出`colelction.delim`，这里是colelction而不是collection，hive里面这个单词写错了，所以还是要按照错误的来。

```sql
alter table t8 set serdepropertyes('colelction.delim'=',');
```

## 3. Struct 的使用
创建数据表

```sql
CREATE TABLE test(id int,course struct<course:string,score:int>)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
COLLECTION ITEMS TERMINATED BY ',';
```

数据

```
1 english,80
2 math,89
3 chinese,95
```

入库

```sql
LOAD DATA LOCAL INPATH '/home/hadoop/test.txt' OVERWRITE INTO TABLE test;
```

查询

```
hive> select * from test;
OK
1       {"course":"english","score":80}
2       {"course":"math","score":89}
3       {"course":"chinese","score":95}
Time taken: 0.275 seconds
hive> select course from test;
{"course":"english","score":80}
{"course":"math","score":89}
{"course":"chinese","score":95}
Time taken: 44.968 seconds
select t.course.course from test t; 
english
math
chinese
Time taken: 15.827 seconds
hive> select t.course.score from test t;
80
89
95
Time taken: 13.235 seconds
```

## 4. 不支持组合的复杂数据类型

我们有时候可能想建一个复杂的数据集合类型，比如下面的a字段，本身是一个Map，它的key是string类型的，value是Array集合类型的。

建表

```sql
create table test1(id int,a MAP<STRING,ARRAY<STRING>>)
row format delimited fields terminated by '\t' 
collection items terminated by ','
MAP KEYS TERMINATED BY ':';
```

导入数据

```
1 english:80,90,70
2 math:89,78,86
3 chinese:99,100,82

LOAD DATA LOCAL INPATH '/home/hadoop/test1.txt' OVERWRITE INTO TABLE test1;
```

这里查询出数据：

```sql
hive> select * from test1;
OK
1	{"english":["80"],"90":null,"70":null}
2	{"math":["89"],"78":null,"86":null}
3	{"chinese":["99"],"100":null,"82":null}
```

可以看到，已经出问题了，我们意图是想"english":["80", "90", "70"]，实际上把90和70也当作Map的key了，value值都是空的。分析一下我们的建表语句，`collection items terminated by ','`制定了集合类型（map, struct, array）数据元素之间分隔符是", "，实际上map也是属于集合的，那么也会按照逗号分出3个key-value对；由于`MAP KEYS TERMINATED BY ':'`定义了map中key-value的分隔符是":"，第一个“english”可以准确识别，后面的直接把value置为"null"了。