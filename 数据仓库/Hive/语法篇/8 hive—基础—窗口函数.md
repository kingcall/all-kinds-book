[toc]
## 窗口函数

- 窗口函数可以让明细和聚合结果共存
- 在sql中有一类函数叫做聚合函数,例如sum()、avg()、max()等等,这类函数可以将多行数据按照规则聚集为一行,一般来讲聚集后的行数是要少于聚集前的行数的.但是有时我们想要既显示聚集前的数据,又要显示聚集后的数据,这时我们便引入了窗口函数
- 窗口函数是SQL语句最后执行的函数，因此可以把SQL结果集想象成输入数据）

```
- 如果只使用partition by子句,未指定order by的话,我们的聚合是分组内的聚合. 
- 使用了order by子句,未使用window子句的情况下,默认从起点到当前行.
```
## 窗口的定义
- 开窗函数一般就是说的是over（）函数，其窗口是由一个OVER子句定义的多行记录，其作用就如同它的名字，就是限定出一个窗口
- 开窗函数指定了分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变化而变化可以理解为一个分析函数必然要搭配一个窗口函数，以指定原数据的规则，便于分析函数实现。
- patition by是按照一个一个reduce去处理数据的，所以要使用全局排序order by
- distribute by是按照多个reduce去处理数据的，所以对应的排序是局部排序sort by
## 准备

- 建表语句
```
create table t_window(name string,orderdate string,cost int) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
LOAD DATA LOCAL INPATH '/home/admin/1.txt' OVERWRITE INTO TABLE t_window;
```
- 数据
```
jack,2015-01-01,10
tony,2015-01-02,15
jack,2015-02-03,23
tony,2015-01-04,29
jack,2015-01-05,46
jack,2015-04-06,42
tony,2015-01-07,50
jack,2015-01-08,55
mart,2015-04-08,62
mart,2015-04-09,68
neil,2015-05-10,12
mart,2015-04-11,75
neil,2015-06-12,80
mart,2015-04-13,94
```
## 例子
### sum
#### sum 不去重
- 查询在2015年4月份购买过的顾客及总人数
```
select name,count(*) over ()
from t_window
where substring(orderdate,1,7) = '2015-04
```
![image](https://note.youdao.com/yws/res/14564/EB771E377E6240E48C0F6201B38D659C)
#### sum 去重 1
```
select distinct name,count(*) over ()
from t_window
where substring(orderdate,1,7) = '2015-04
```
#### sum 去重 2
---- 这种写法性能更好
```
select name,count(*) over ()
from t_window
where substring(orderdate,1,7) = '2015-04'
group by name;
-- 和下面的语句进行区别
select name,count(*)
from t_window
where substring(orderdate,1,7) = '2015-04'
group by name
```
### partition by
```
select name,orderdate,cost,sum(cost) over(partition by month(orderdate))
from t_window
```
![image](https://note.youdao.com/yws/res/14585/8F838BE94E9B47D98243CC12A1BFE059)
### order by
- 窗口函数中的order by 是为了累积的计算
- order by子句会让输入的数据强制排序（文章前面提到过，窗口函数是SQL语句最后执行的函数，因此可以把SQL结果集想象成输入数据）。
- Order By子句对于诸如Row_Number()，Lead()，LAG()等函数是必须的，因为如果数据无序，这些函数的结果就没有任何意义。
- 因此如果有了Order By子句，则Count()，Min()等计算出来的结果就没有任何意义。
```
select name,orderdate,cost,sum(cost) over(partition by month(orderdate) order by orderdate) from t_window
```
![image](https://note.youdao.com/yws/res/14598/B8BF3B9991854481B47201D72ADE1B10)

### window子句
- PRECEDING：往前 
- FOLLOWING：往后 
- CURRENT ROW：当前行 
- UNBOUNDED：起点
- UNBOUNDED PRECEDING 表示从前面的起点
    - n PRECEDING(前面多少行)
- UNBOUNDED FOLLOWING：表示到后面的终点
```
select name,orderdate,cost,
sum(cost) over() as sample1,--所有行相加
sum(cost) over(partition by name) as sample2,--按name分组，组内数据相加
sum(cost) over(partition by name order by orderdate) as sample3,--按name分组，组内数据累加
sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING and current row )  as sample4 ,--和sample3一样,由起点到当前行的聚合
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING   and current row) as sample5, --当前行和前面一行做聚合
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING   AND 1 FOLLOWING  ) as sample6,--当前行和前边一行及后面一行
sum(cost) over(partition by name order by orderdate rows between current row and UNBOUNDED FOLLOWING ) as sample7 --当前行及后面所有行
from t_window;
```
```
i. sum(cost) over() as sample1  --所有行相加
ii. sum(cost) over(partition by name) as sample2,--按name分组，组内数据相加
iii. sum(cost) over(partition by name order by orderdate) as sample3,--按name分组，按照日期从小到大进行 cost的累加（组内排序，组内相加）
iv. sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING and current row )  as sample4 ,--和sample3一样,由起点到当前行的聚合
v. sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING   and current row) as sample5, --当前行和前面一行做聚合
vi. sum(cost) over(partition by name order by orderdaterows between 1 PRECEDING   AND 1 FOLLOWING  ) assample6,--当前行和前边一行及后面一行
sum(cost) over(partition by name order by orderdate rows between current row and UNBOUNDED FOLLOWING ) as sample7 --当前行及后面所有行
```

### 序列函数
#### NTILE
- NTILE(n)，用于将分组数据按照顺序切分成n片，返回当前切片值
- NTILE不支持ROWS BETWEEN， 
```
比如 NTILE(2) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND CURRENT ROW)
```
- 如果切片不均匀，默认增加第一个切片的分布




## 例子
### 数据准备
```
CREATE EXTERNAL TABLE lxw1234 (
cookieid string,
createtime string,   --day 
pv INT
) ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
stored as textfile location '/tmp/lxw11/';
 
DESC lxw1234;
cookieid                STRING 
createtime              STRING 
pv                      INT 
 
hive> select * from lxw1234;
OK
cookie1 2015-04-10      1
cookie1 2015-04-11      5
cookie1 2015-04-12      7
cookie1 2015-04-13      3
cookie1 2015-04-14      2
cookie1 2015-04-15      4
cookie1 2015-04-16      4
```
### 结果分析
```
SELECT cookieid,
createtime,
pv,
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime) AS pv1, -- 默认为从起点到当前行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS pv2, --从起点到当前行，结果同pv1 
SUM(pv) OVER(PARTITION BY cookieid) AS pv3,                             --分组内所有行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND CURRENT ROW) AS pv4,   --当前行+往前3行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND 1 FOLLOWING) AS pv5,    --当前行+往前3行+往后1行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS pv6   ---当前行+往后所有行  
FROM lxw1234;
 
cookieid createtime     pv      pv1     pv2     pv3     pv4     pv5      pv6 
-----------------------------------------------------------------------------
cookie1  2015-04-10      1       1       1       26      1       6       26
cookie1  2015-04-11      5       6       6       26      6       13      25
cookie1  2015-04-12      7       13      13      26      13      16      20
cookie1  2015-04-13      3       16      16      26      16      18      13
cookie1  2015-04-14      2       18      18      26      17      21      10
cookie1  2015-04-15      4       22      22      26      16      20      8
cookie1  2015-04-16      4       26      26      26      13      13      4

```

```
pv1: 分组内从起点到当前行的pv累积，如，11号的pv1=10号的pv+11号的pv, 12号=10号+11号+12号
pv2: 同pv1
pv3: 分组内(cookie1)所有的pv累加
pv4: 分组内当前行+往前3行，如，11号=10号+11号， 12号=10号+11号+12号， 13号=10号+11号+12号+13号， 14号=11号+12号+13号+14号
pv5: 分组内当前行+往前3行+往后1行，如，14号=11号+12号+13号+14号+15号=5+7+3+2+4=21
pv6: 分组内当前行+往后所有行，如，13号=13号+14号+15号+16号=3+2+4+4=13，14号=14号+15号+16号=2+4+4=10

如果不指定ROWS BETWEEN,默认为从起点到当前行;
如果不指定ORDER BY，则将分组内所有值累加;
关键是理解ROWS BETWEEN含义,也叫做WINDOW子句：
PRECEDING：往前
FOLLOWING：往后
CURRENT ROW：当前行
UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING：表示到后面的终点。
```





### row_number

- 不管col2字段的值是否相等，行号一直递增，比如：有两条记录的值相等，但一个是第一，一个是第二

### rank

- 上下两条记录的col2相等时，记录的行号是一样的，但下一个col2值的行号递增N（N是重复的次数），比如：有两条并列第一，下一个是第三，没有第二

### dense_rank

- 上下两条记录的col2相等时，下一个col2值的行号递增1，比如：有两条并列第一，下一个是第二