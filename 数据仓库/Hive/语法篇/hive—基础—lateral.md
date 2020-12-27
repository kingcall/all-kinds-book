explode与lateral view在关系型数据库中本身是不该出现的，因为他的出现本身就是在操作不满足第一范式的数据（每个属性都不可再分），本身已经违背了数据库的设计原理（不论是业务系统还是数据仓库系统），不过大数据技术普及后，很多类似pv，uv的数据，在业务系统中是存贮在非关系型数据库中，用json存储的概率比较大，直接导入hive为基础的数仓系统中，就需要经过ETL过程解析这类数据，explode与lateral view在这种场景下大显身手。

```sql
hive> select * from test.user_log;
OK
user_log.userid	user_log.time	user_log.url
Peter	2015-10-12 01:10:00	url1
Peter	2015-10-12 01:15:10	url2
Peter	2015-10-12 01:16:40	url3
Peter	2015-10-12 02:13:00	url4
Peter	2015-10-12 03:14:30	url5
Marry	2015-11-12 01:10:00	url1
Marry	2015-11-12 01:15:10	url2
Marry	2015-11-12 01:16:40	url3
Marry	2015-11-12 02:13:00	url4
Marry	2015-11-12 03:14:30	url5
```

当使用UDTF函数的时候,hive只允许对拆分字段进行访问的。

```sql
hive> select userid, explode(split(time, '-')) from test.user_log;
FAILED: SemanticException [Error 10081]: UDTF's are not supported outside the SELECT clause, nor nested in expressions
```

这样写是可以的：

```sql
select explode(split(time, '-')) from test.user_log;
```

但是实际中经常要拆某个字段,然后一起与别的字段一起出.例如上面的id和拆分的array元素是对应的.我们应该如何进行连接呢?我们知道直接select id,explode()是不行的.这个时候就需要`lateral view`出场了。

lateral view为侧视图,意义是为了配合UDTF来使用,把某一行数据拆分成多行数据.不加lateral view的UDTF只能提取单个字段拆分,并不能塞会原来数据表中.加上lateral view就可以将拆分的单个字段数据与原始表数据关联上。

在使用lateral view的时候需要指定视图别名和生成的新列别名。

```sql
select userid, a  from test.user_log lateral view explode(split(time, '-')) time_view as a;
```

`time_view`为视图别名，`a`为指定新列别名。

结果：

```
userid	a
Peter	2015
Peter	10
Peter	12 01:10:00
Peter	2015
Peter	10
Peter	12 01:15:10
Peter	2015
Peter	10
...省略...
```

lateral view explode 相当于一个拆分数组的虚表，然后根据userid将其与原表进行笛卡尔积关联.

我们也可以多次使用lateral view explode

```sql
select userid, a, a2  from test.user_log 
lateral view explode(split(time, '-')) time_view as a 
lateral view explode(split(time, '-')) time_view2 as a2;
```

可想而知，做了笛卡尔集，由于explode里面拆分出来的数组长度是3，lateral view使用了2次，原始是10条数据，结果就是3*3，翻了9倍，一共90条：

```
userid	a	a2
Peter	2015	2015
Peter	2015	10
Peter	2015	12 01:10:00
Peter	10	2015
Peter	10	10
Peter	10	12 01:10:00
Peter	12 01:10:00	2015
Peter	12 01:10:00	10
Peter	12 01:10:00	12 01:10:00
Peter	2015	2015
Peter	2015	10
Peter	2015	12 01:15:10
Peter	10	2015
...省略...
```

