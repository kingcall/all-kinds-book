Lag和Lead分析函数可以在同一次查询中取出同一字段的后N行的数据(Lag)和前N行的数据(Lead)作为独立的列。

这种操作可以代替表的自联接，并且LAG和LEAD有更高的效率，其中over()表示当前查询的结果集对象，括号里面的语句则表示对这个结果集进行处理。

## Lag

LAG(col,n,DEFAULT) 用于统计窗口内往上第n行值第一个参数为列名，第二个参数为往上第n行（可选，默认为1），第三个参数为默认值（当往上第n行为NULL时候，取默认值，如不指定，则为NULL）可以用来做一些时间的维护，如上一次登录时间。

##　场景

用户Peter在浏览网页，在某个时刻，Peter点进了某个页面，过一段时间后，Peter又进入了另外一个页面，如此反复，那怎么去统计Peter在某个特定网页的停留时间呢，又或是怎么统计某个网页用户停留的总时间呢？

```sql
create table test.user_log(
    userid string,
    time string,
    url string
) row format delimited fields terminated by '\t';
```

使用`load`命令将如下测试数据导入：

```
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

数据说明：Peter	2015-10-12 01:10:00	url1   ，表示Peter在`2015-10-12 01:10:00`进入了网页`url2`，即记录的是进入网页的时间。

```sql
select userid,
time etime,
lag(time, 1, '1970-01-01 00:00:00') over(partition by userid order by time) stime,
url 
from test.user_log;
```

这里`etime`是结束时间，`stime`是开始时间，结果：

```
Marry	2015-11-12 01:10:00	1970-01-01 00:00:00	url1
Marry	2015-11-12 01:15:10	2015-11-12 01:10:00	url2
Marry	2015-11-12 01:16:40	2015-11-12 01:15:10	url3
Marry	2015-11-12 02:13:00	2015-11-12 01:16:40	url4
Marry	2015-11-12 03:14:30	2015-11-12 02:13:00	url5
Peter	2015-10-12 01:10:00	1970-01-01 00:00:00	url1
Peter	2015-10-12 01:15:10	2015-10-12 01:10:00	url2
Peter	2015-10-12 01:16:40	2015-10-12 01:15:10	url3
Peter	2015-10-12 02:13:00	2015-10-12 01:16:40	url4
Peter	2015-10-12 03:14:30	2015-10-12 02:13:00	url5
```

计算总时间，只需要用结束时间 - 开始时间，然后分组累加即可。

```sql
select userid,
UNIX_TIMESTAMP(time, 'yyyy-MM-dd HH:mm:ss') - 
UNIX_TIMESTAMP(lag(time, 1, '1970-01-01 00:00:00') over(partition by userid order by time), 'yyyy-MM-dd HH:mm:ss'),
url 
from test.user_log;
```

结果

```
Marry	1447290600	url1
Marry	310	url2
Marry	90	url3
Marry	3380	url4
Marry	3690	url5
Peter	1444612200	url1
Peter	310	url2
Peter	90	url3
Peter	3380	url4
Peter	3690	url5
```

因为有两个我将默认值置为了`1970-01-01`，所以算出来比较大，实际工作中需要按照实际情况处理。