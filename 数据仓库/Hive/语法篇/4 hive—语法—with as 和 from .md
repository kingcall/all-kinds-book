[TOC]

## with as 和 from

### with as

在我们介绍hive 的时候我们说到了hive 不止实现了标准的SQL语法，还扩展了很多其特有的语法，还允许用户自定义函数，今天我们就来学习一个hive 的一个扩展语法，with...as也叫做子查询部分,语句允许hive定义一个sql片段,供整个sql使用，这里的使用不仅仅指的是像视图一样简化你的SQL 书写，而且还会将这个片段产生的结果集保存在内存中,后续的sql均可以访问这个结果集,左右有点像物化视图或者是临时表

hive 可以通过with查询来提高查询性能，因为先通过with语法将数据查询到内存，然后后面其它查询可以直接使用

```sql
with q1 as (select * from src where key= '5'),
q2 as (select * from srcs2 where key = '4')
select * from q1 union all select * from q2;
```

其实这个语法它好处比较多 



#### 语法规则

```sql
with temp as (
    select * from table
)
select * from temp1;
```

当然你也可以同时定义几个，然后使用逗号分隔就可以

```sql
with temp1 as (
    select * from xxx
),temp2 as (
    select * from xxx
)
select * from temp1,temp2;
```

`with...as`还支持嵌套

```sql
with temp2 as (
    with temp1 as (
        select * from xxx
    )
    select * from temp1
)
select * from temp2;
```

当然这种用的也不多，主要还是前面第一种和第二种

`with...as`只能在一条sql中使用使用

```
with temp1 as (
    select * from xxx
)
select * from temp1;
select xxx from temp1; 
```

这样会报表找不到。

#### 案例

我有一张表，表结构如下

```
+--------------+------------+----------+
|   col_name   | data_type  | comment  |
+--------------+------------+----------+
| user_id      | bigint     |          |
| item_id      | bigint     |          |
| category_id  | bigint     |          |
| behavior     | string     |          |
| ts           | int        |          |
+--------------+------------+----------+
```

behavior 代表的实用户行为(buy,cart,fav,pv)， ts 是时间戳

我想要计算buy的用户数， cart 的记录数目，我们看一下普通的写法

```sql
select behavior,count(distinct user_id) as userCnt from  user_behavior where behavior='buy' group by behavior
union all 
select behavior,count(1) as userCnt from user_behavior where behavior='cart' group by behavior;
```

![image-20201231195805493](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201231195805493.png)

```sql
with tmp as (select user_id,behavior from user_behavior where behavior='buy' or  behavior='cart')
select behavior,count(distinct user_id) as userCnt from  tmp where behavior='buy' group by behavior
union all 
select behavior,count(1) as userCnt from tmp where behavior='cart' group by behavior;
```

![image-20201231201705400](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201231201705400.png)

### from

hive 可以通过with查询来提高查询性能，因为先通过with语法将数据查询到内存，然后后面其它查询可以直接使用,from 也有类似的用法，比起with 更加简洁,但是后面的子的SQL 必须是insert 才可以，就像下面这样

```sql
from 
	(select user_id,behavior from user_behavior where behavior='buy' or  behavior='cart')
tmp
INSERT OVERWRITE TABLE ods_dest_data select behavior,count(distinct user_id) as userCnt where behavior='buy' group by behavior
INSERT OVERWRITE TABLE ods_dest_data2 select behavior,count(1) as userCnt  where behavior='cart' group by behavior;
```

![image-20210101080725419](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210101080725419.png)

所以说如果我们的场景就是一次查询多次插入的话，还是很有用的

```sql
from 
	ods.user_behavior
tmp
INSERT OVERWRITE TABLE ods_dest_data select behavior,count(distinct user_id) as userCnt where behavior='buy' group by behavior
INSERT OVERWRITE TABLE ods_dest_data2 select behavior,count(1) as userCnt  where behavior='cart' group by behavior;
```

你甚至可以没有from 后面的子查询，直接从一张已经存在的表里或者视图里数据也是可以的

```sql
 FROM src
  INSERT OVERWRITE TABLE dest1 SELECT src.* WHERE src.key < 100
  INSERT OVERWRITE TABLE dest2 SELECT src.key, src.value WHERE src.key >= 100 and src.key < 200
  INSERT OVERWRITE TABLE dest3 PARTITION(ds='2008-04-08', hr='12') SELECT src.key WHERE src.key >= 200 and src.key < 300
  INSERT OVERWRITE LOCAL DIRECTORY '/tmp/dest4.out' SELECT src.value WHERE src.key >= 300;
```

设置你可以在一句SQL同时插入数据到表里和导出数据

```sql
from
    (select behavior,count(distinct user_id) as userCnt from user_behavior group by behavior) a
inner join
    (select behavior,count(distinct user_id) as userCnt from user_behavior group by behavior) b
on
    a.behavior=b.behavior
INSERT OVERWRITE TABLE ods_dest_data
    select a.behavior,a.userCnt where a.behavior='buy'
INSERT OVERWRITE TABLE ods_dest_data
    select b.behavior,b.userCnt where b.behavior='cart'
;
```

你也可以写出在from 语句里实现join 的逻辑，然后下面使用join 出来的结果集合

```sql
from
    (select behavior,count(distinct user_id) as userCnt from user_behavior group by behavior) a,
    (select behavior,count(distinct user_id) as userCnt from user_behavior group by behavior) b
INSERT OVERWRITE TABLE ods_dest_data
    select a.behavior,a.userCnt where a.behavior='buy'
INSERT OVERWRITE TABLE ods_dest_data2
    select b.behavior,b.userCnt where b.behavior='cart'
;
```

也可以写出上面这种多个from 并列的子句

## 总结

1. 可以简化SQL的写法和视图的作用有点像

2. 可以提高性能——减少表的扫描次数(缓存结果集的方式)

3. 当一个查询结果需要多次使用的时候我们就可以使用with as 这种方式,再特殊一点的场景我们也可以尝试使用from 开头的这种写法

4. with as 可以替代from 的写法，但是from 开头的这种写法只适合插入的场景

   

   