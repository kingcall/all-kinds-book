Lag和Lead分析函数可以在同一次查询中取出同一字段的后N行的数据(Lag)和前N行的数据(Lead)作为独立的列。

这种操作可以代替表的自联接，并且LAG和LEAD有更高的效率，其中over()表示当前查询的结果集对象，括号里面的语句则表示对这个结果集进行处理。



## LEAD

与LAG相反，LEAD(col,n,DEFAULT) 用于统计窗口内往下第n行值

参数1为列名，参数2为往下第n行（可选，默认为1），参数3为默认值（当往下第n行为NULL时候，取默认值，如不指定，则为NULL）

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

## 分析

要计算`Peter`在页面`url1`停留的时间，需要用进入页面`url2`的时间，减去进入`url1`的时间，即`2015-10-12 01:15:10`这个时间既是离开页面`url1`的时间，也是开始进入页面`url2`的时间。

获取用户在某个页面停留的起始与结束时间：

```sql
select userid,
time stime,
lead(time) over(partition by userid order by time) etime,
url 
from test.user_log;
```

stime就是进入页面时间，etime就是离开页面时间，结果是这样的：

```
Marry	2015-11-12 01:10:00	2015-11-12 01:15:10	url1
Marry	2015-11-12 01:15:10	2015-11-12 01:16:40	url2
Marry	2015-11-12 01:16:40	2015-11-12 02:13:00	url3
Marry	2015-11-12 02:13:00	2015-11-12 03:14:30	url4
Marry	2015-11-12 03:14:30	NULL	url5
Peter	2015-10-12 01:10:00	2015-10-12 01:15:10	url1
Peter	2015-10-12 01:15:10	2015-10-12 01:16:40	url2
Peter	2015-10-12 01:16:40	2015-10-12 02:13:00	url3
Peter	2015-10-12 02:13:00	2015-10-12 03:14:30	url4
Peter	2015-10-12 03:14:30	NULL	url5
```

用etime减去stime，然后按照用户分组累加就是，每个用户访问的总时间了。

```sql
select userid,
time stime,
lead(time) over(partition by userid order by time) etime,
UNIX_TIMESTAMP(lead(time) over(partition by userid order by time),'yyyy-MM-dd HH:mm:ss')- UNIX_TIMESTAMP(time,'yyyy-MM-dd HH:mm:ss') period,
url 
from test.user_log;
```

这里展示出了stime(开始时间)，etime(离开时间)，period(停留时长)，url(页面地址)，结果：

```
Marry	2015-11-12 01:10:00	2015-11-12 01:15:10	310	url1
Marry	2015-11-12 01:15:10	2015-11-12 01:16:40	90	url2
Marry	2015-11-12 01:16:40	2015-11-12 02:13:00	3380	url3
Marry	2015-11-12 02:13:00	2015-11-12 03:14:30	3690	url4
Marry	2015-11-12 03:14:30	NULL	NULL	url5
Peter	2015-10-12 01:10:00	2015-10-12 01:15:10	310	url1
Peter	2015-10-12 01:15:10	2015-10-12 01:16:40	90	url2
Peter	2015-10-12 01:16:40	2015-10-12 02:13:00	3380	url3
Peter	2015-10-12 02:13:00	2015-10-12 03:14:30	3690	url4
Peter	2015-10-12 03:14:30	NULL	NULL	url5
```

- 这里有空的情况，也就是没有获取到离开时间，这要看实际业务怎么定义了，如果算到23点，太长了。