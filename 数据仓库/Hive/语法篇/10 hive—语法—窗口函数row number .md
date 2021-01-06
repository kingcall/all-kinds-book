

[TOC]

## row_number

前面我们介绍[窗口函数](https://blog.csdn.net/king14bhhb/article/details/112172378)的时候说到了窗口函数的使用场景，我们也给它起了一个名字进行区分，通用窗口函数和特殊窗口函数，今天我们就来看一下排序相关的窗口函数，因为是窗口函数，并且我们说它是用来排序的，我们大概也能猜到它就是用来对窗口内的数据进行排序的

其实关于排序我们前面也介绍过order by,sort by 等排序的方式[Hive语法之常见排序方式](https://blog.csdn.net/king14bhhb/article/details/112093373),为什么还有窗口函数进行排序的，因为前面的order by,sort by 等虽然可以排序但是不能给我们返回排序的值(名次)，如果你用过mysql 的话，这个时候你就知道写存储过程或者使用自定义变量来完成这个功能，row number 也是一样的道理，可以按照我们自定义的排序规则，返回对应的排序先后顺序的值

所以我们认为row_number是窗口排序函数，但是hive 也没有提供非窗口的排序函数，但是我们前面说过了如果没有窗口的定义中没有partition by 那就是将整个数据输入当成一个窗口，那么这种情况下我们也可以使用窗口排序函数完成全局排序。

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
    id string,
    dept string,
    salary int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/workspace/hive/number.txt' OVERWRITE INTO TABLE ods_num_window;
```



### 从例子中学习 row_number

#### 每个部门的员工按照工资降序排序

```sql
select
    *,row_number() over(partition by dept order by salary desc) as rn
from
    ods_num_window
;
```

![image-20210105203040264](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210105203040264.png)

我们看到每个部门都有自己的第一名，明显的可以看到排序是发生在每个部门内部的

#### 全部的员工按照工资降序排序

```sql
select
    *,row_number() over(order by salary desc) as rn
from
    ods_num_window
;
```

![image-20210105203343950](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210105203343950.png)

当我们没有定义partition by 子句的时候，我们的所有数据都放在一个窗口里面，这个时候我们的排序就是全局排序，其实如果你仔细看过我们的[Hive语法之窗口函数初识](https://blog.csdn.net/king14bhhb/article/details/112172378)这一节的话，你就知道partition by 其实是定义了子窗口，如果没有子窗口的话，那就就是一个窗口，如果所有的数据都放在一个窗口的话那就是全局排序



#### 取每个部门的工资前两名

这个是row_number()  函数非常常见的使用场景`top-N`,其实如果你仔细看过我们的[Hive语法之窗口函数初识](https://blog.csdn.net/king14bhhb/article/details/112172378)这一节的话，你就知道partition by 其实是定义了子窗口，那其实这里的`top-N`,本质上是子窗口的的`top-N`

```sql
select
    *
from(
   select
       *,row_number() over(partition by dept order by salary desc) as rn
   from
       ods_num_window
) tmp
where
    rn <=2
;
```

![image-20210105204433374](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210105204433374.png)

其实这个的实现方式就是我们对数据在子窗口内进行排序，然后选择出我们我们需要的数据，也就是这里的` rn <=2`



### rank 和 dense_rank

其实这两个窗口函数和row_number 是一样的，都是窗口排序函数，既然这样那为什么还有这两个函数呢，存在即合理，我们看一下row_number 函数，这次我们采用升序排序

```
select
    *,row_number() over(partition by dept order by salary) as rn
from
    ods_num_window
;
```

我们看到在销售部门有两个人的工资其实是一样的10000，但是排名不一样

![image-20210105205323962](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210105205323962.png)

接下来我们看一下rank，我们发现销售部门那两个工资相等的实并列第一了，然后下一个人直接第三了

![image-20210105205551317](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210105205551317.png)

接下来我们再看一下 dense_rank，工资相等的两个人依然是排名相等的，但是下一个人还是第二

![image-20210105205904656](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210105205904656.png)

## 总结

1. rank() 排序相同时会重复，总数不会变(会有间隙跳跃，数据不连续)
2. dense_rank() 排序相同时会重复，总数会减少(不会有间隙，数据连续的)
3. row_number() 会根据顺序计算，不会重复不会减少

