[TOC]

## 复合数据类型

除了使用`string`之外的基础数据类型，Hive中的还支持使用struct, map, array，uniontype 等复合数据类型。一般在企业中使用 arraye和map会比较多一些。

| 数据类型 | 描述                                                         | 语法示例                            |
| -------- | ------------------------------------------------------------ | ----------------------------------- |
| STRUCT   | 和C语言中的struct或者"对象"类似，都可以通过"点"符号访问元素内容。 | struct{'John', 'Doe'}               |
| MAP      | MAP是一组键-值对元素集合，使用key可以访问元素。              | map('fisrt', 'John', 'last', 'Doe') |
| ARRAY    | 数组是一组具有相同数据类型和名称的变量的集合。               | Array('John', 'Doe')                |

### 1. Array类型

ARRAY类型是由一系列相同数据类型的元素组成，这些元素可以通过下标来访问。比如有一个ARRAY类型的变量fruits，它是由['apple','orange','mango']组成，那么我们可以通过fruits[1]来访问元素orange，因为ARRAY类型的下标是从0开始的；



创建数据库表，以array作为数据类型

```sql
create table person(name string,work_locations array<string>)
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

```sql
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

Hive 中的数组的访问方式和Java 的一样，字段名[index]，不同的是Hive中不会有下标越界的异常，如果发生了下标越界，只会返回NULL 而已，除了这样通过下标的方式去访问，Hive 还给我们提供了一个函数array_contains 来判断数组里面是否包含特定的值。

```sql
select * from person where array_contains(work_locations,'beijing');
```

![image-20201225213000661](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/25/21:30:01-image-20201225213000661.png)



### 2. Map 类型

MAP包含key->value键值对，可以通过key来访问元素。比如"userlist"是一个map类型，其中username是key，password是value；那么我们可以通过userlist['username']来得到这个用户对应的password；

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

```sql
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

### 3. Struct 类型

STRUCT可以包含不同数据类型的元素。这些元素可以通过"点语法"的方式来得到所需要的元素，比如user是一个STRUCT类型，那么可以通过user.address得到这个用户的地址。有点类似Java 中的对象

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

```sql
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

**size(Map)函数：**可得map的长度。返回值类型：int 

**map_keys(Map)函数：**可得map中所有的key; 返回值类型: array

**map_values(Map)函数：**可得map中所有的value; 返回值类型: array

**判断map中是否包含某个key值：** array_contains(map_keys(t.params),'k0');



### 4. uniontype 类型

uniontype可以理解为泛型，同一时刻联合体中的一个元素生效，uniontype中的元素共享内存,可以通过create_union内置函数创建uniontype：create_union(tag, val1, val2) tag是数字，0开始，必须小于后面参数的个数,怎么理解呢，uniontype 类型的意思就是这个字段可以存储你定义的类型中的每一种都可以，例如我有一个字段是 `uni uniontype<int, double, string>` 这就是说我uni 这个字段可以存储`<int, double, string>` 三种类型中的任意一种，但是同时只能存一个，不能说我同时存了一个字段是`<int, string>` 因为这都不满足第一范式了。

开始之前我们先介绍一个函数create_union,你可以执行`desc function extended create_union;`来查看一下这个函数的定义是什么

```
+--------------------------------------------------------------------------------------------+
|                      tab_name                                                              |
+--------------------------------------------------------------------------------------------+
| create_union(tag, obj1, obj2, obj3, ...) - Creates a union with the object for given tag   |
| Example:                                                                                   |
|   > SELECT create_union(1, 1, "one") FROM src LIMIT 1;                                     |
|   one                                                                                      |
| Function class:org.apache.hadoop.hive.ql.udf.generic.GenericUDFUnion                       |
| Function type:BUILTIN                                                                      |
+--------------------------------------------------------------------------------------------+
```

这个函数就是返回一个uniontype 类型的值，就像date 函数返回一个date 类型的值一样。那这个函数是怎么用的呢，第一个参数你可以认为是数组的下标，后面的全部参数你可以认为是数组，返回的uniontype值就是数组指定下标处的值，只不过这个返回值

还会带一个下标

```sql
select  create_union(0,10,10.0, '10.1');
```

![image-20201225223309798](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/25/22:33:11-image-20201225223309798.png)

此时返回值是一个类似map的结构，其中0是下标，10 是下标为0 处对应的值。

下面我们来一个完整的演示

```sql
create table uniontab(
    arr array<string>,
    mp MAP<INT, string>,
    stru struct<name:string, age:int>,
    uni uniontype<int, decimal(2,0), string>
)
row format 
  delimited fields terminated by ','
  collection items terminated by '#'
  map keys terminated by ':'
  lines terminated by '\n';
```

接下来我们插入一条数据,就像下面一样，你就会看到报错了

```
insert into uniontab select array('d','e','f'), map(1,'zjx',2,'wly'), named_struct('name', 'wly', 'age', 17), create_union(0,10.0, '10.1');
```

![image-20201225223620836](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/25/22:36:21-image-20201225223620836.png)

`Cannot convert column 3 from uniontype<int,string> to uniontype<int,decimal(2,0),string>. (state=42000,code=10044)`

什么意思呢，第三列` uniontype<int,string> ` 不能转换成 ` uniontype<int,decimal(2,0),string>` ，啥意思呢我们查询出来的，也就是create_union(0,10.0, '10.1')的类型是` uniontype<int,string> ` 它不能转换成我们表定义的 ` uniontype<int,decimal(2,0),string>` ，我们查询出来的类型为什么是` uniontype<int,string> `呢这是和我们的create_union 函数有关，就是我们create_union 函数的的第二个参数数组中的数据个数和类型的顺序必须和我们表定义的时候的类型的顺序一致

```sql
insert into uniontab select array('d','e','f'), map(1,'zjx',2,'wly'), named_struct('name', 'wly', 'age', 17), create_union(0,2,10.0, '101.1');
```

上面这样就对了

### 5. 不支持组合的复杂数据类型

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

## 总结

复杂类型一般用的不多，但是有时候还是会用，因为可能和历史有关或者是数仓的建设者有关，因为复杂类型已经违背我们对数据库的定义，虽然数仓和数据库不一样，但是三范式在表的定义上的影响长远，大家在建表的时候可以思考一下自己真的需要这种复杂数据类型吗

| 类型      | 名称       | 创建函数                               | 访问方式 | 引入版本 |
| --------- | ---------- | -------------------------------------- | -------- | -------- |
| Array     | 数组类型   | array('d','e','f')                     | 下标     | 0.14     |
| Map       | k-v 类型   | map(1,'zjx',2,'wly')                   | 字段名称 | 0.14     |
| Struct    | 结构体类型 | named_struct('name', 'wly', 'age', 17) | .        | 0.0      |
| Uniontype | 联合类型   | create_union(1,10.0, '10.1');          |          | 0.7.0    |

Uniontype 这个类型不太好理解，网上资料也不多，所以我重点讲解了一下，大家可以多体会体会。

针对复合数据类型也衍生出来了很多函数，可以在需要的时候可以自行查阅，后续我也会慢慢添加到这篇文章中去，不断完善，大家如果觉得有什么地方还需要完善可以指出来