[TOC]

## cume_dist和 percent_rank

这是我们要学习的最后两个窗口函数了，这两个窗口函数都是统计占比的

**cume_dist**： 小于等于当前值的行数/分组内总行数

**percent_rank** ：窗口内当前行的RANK值-1/窗口内总行数-1(这里的rank值就是指的是rank 函数的的返回值)

### 测试数据

下面有一份测试数据`id,dept,salary`,然后我们就使用这份测试数据学习我们的窗口排序函数

```
1,销售,10000
2,销售,14000
3,销售,10000
4,后端,20000
5,后端,25000
6,后端,32000
7,AI,40000
8,AI,35000
9,AI,60000
10,数仓,20000
11,数仓,30000
12,数仓,32000
13,数仓,42000
```

```sql
create table ods_num_window(
    id int,
    dept string,
    salary int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/workspace/hive/number.txt' OVERWRITE INTO TABLE ods_num_window;
```



###   从例子中学习cume_dist和 percent_rank



#### 小于当前工资的人数占比

```sql
select
    id,dept,salary,
    cume_dist() over (order by salary) as rn1
from
    ods_num_window
;
```

![image-20210106160115852](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106160115852.png)

从上面我们看到小于小于等于60000的占比是100%(最后一行)，大家可以按照cume_dist 定义去验证验证，加深自己的理解

#### 各个部门小于当前工资的人数占比

上面我们没有使用子窗口的定义，所以计算结果是全局的，下面我们看一下使用了partition by 的cume_dist

```
select
    id,dept,salary,
    cume_dist() over (partition by dept order by salary) as rn1
from
    ods_num_window
;
```

![image-20210106162243772](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106162243772.png)

我们看到每个部门都有自己的1.0 了，而且这个1.0 对于的就是部门的最高工资，这也就说明了我们的计算是在每个部门内部计算小于等于当前记录数的占比

#### 大于等于当前工资的人数占比

前面我们介绍到cume_dist计算的实小于等于当前值的记录占比，那么如何求大于等于呢，其实很简单，只需要改变排序的方向即可

```sql
select
    id,dept,salary,
    cume_dist() over (order by salary desc) as rn1
from
    ods_num_window
;
```

![image-20210106161654493](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106161654493.png)

这个时候我们就看到数据是降序排列的，工资大于等于10000 的人占比是100%



####  percent_rank 到底计算的是什么

percent_rank 到底是怎么计算的，好多人看了文档也不清楚，现在我们来梳理一下，下面的在计算rn1和rn2的时候，用到的分母12是记录总数13减去1得到的

```
select
    id,dept,salary,
    rank() over (order by salary) as rn,
    (rank() over (order by salary)-1)/12 as rn1,
    (dense_rank() over (order by salary)-1)/12 as rn2,
    percent_rank() over (order by salary) as rn3
from
    ods_num_window
;
```

![image-20210106164731292](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106164731292.png)

我们的percent_rank 函数的返回值是rn3，我们的发现rn3列和rn1 是完全相等的，那么我们就知道percent_rank实质上是`(rank() over (order by salary)-1)/当前窗口内的记录数-1`，那么既然如此我们就可以改变排序的顺序达到降序的效果

```
select
    id,dept,salary,
    rank() over (order by salary desc ) as rn,
    percent_rank() over (order by salary desc) as rn2
from
    ods_num_window
;
```

![image-20210106165511434](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106165511434.png)

同理我们也可以定义子窗口，去计算窗口内的一些计算指标



## 总结

1. cume_dist和 percent_rank 主要用来计算百分比，主要是要理解这两个窗口函数的定义**cume_dist**： 小于等于当前值的行数/分组内总行数，**percent_rank** ：窗口内当前行的RANK值-1/窗口内总行数-1(这里的rank值就是指的是rank 函数的的返回值)
2. 其实这两个函数在工作中用的不是很多

