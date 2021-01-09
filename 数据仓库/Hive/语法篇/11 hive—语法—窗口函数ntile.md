[TOC]

## ntile

今天我们学习一个窗口函数ntile(tile 的意思是瓦片，动词的意思是并列显示)，ntile(n)用于将分组数据**按照顺序切均匀分成n片**，返回每条数据当前所在的切片值,其实就是将数据分为n 组，然后告诉你这条数据属于那一组和其他窗口函数不同的是，它不支持ROWS BETWEEN

从按照顺序均匀分成n 片的描述我们就能知道这个窗口函数是按照某一顺序对数据进行均匀分片的，如果我们不指定order by 子句，那就是按照数据的输入逆序进行的。

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



### 从例子中学习ntile

#### 不指定排序将数据分成3分

```sql
select
    *,ntile(3) over() as rn
from
    ods_num_window
;
```

![image-20210106085130285](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106085130285.png)

我们看到由于我们的数据有13条不能被均分成3分，所以我们发现第一份是多了一条数据的，这就是ntile 的一个规律，如果数据不能被均分，多出来的数据均匀的分到前面几份上

还有就是我们的如数顺序是从`1 ..... 13` 但是输出成了`13 ...... 1` 这样的顺序，那就是说如果我们不指定ntile 的排序，ntile 就会按照我们输入顺序的逆序进行输出

#### 指定排序将数据分成3分

```
select
    *,ntile(3) over(order by id asc ) as rn
from
    ods_num_window
;
```

![image-20210106085945201](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106085945201.png)



#### 指定 partition by 进行将数据分成3分

前面我们不止一次的讲到partition by 就是定义子窗口的意思，那么如果我们指定了 partition by 那么我们的窗口分片函数就会将其分片的功能体现在每个子窗口里，也就是说多每个子窗口进行分片，前面我们演示都是对所有数据进行分片

```sql
select
    *,ntile(3) over(partition by dept) as rn
from
    ods_num_window
;
```

![image-20210106091417478](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106091417478.png)

我们看到这下每个子窗口的数据都被分成了3片，由于数仓的人数是4人，不能被分成3片，所以第一片是两个人

### 使用场景

#### 求全局百分比`Top-%n`

例如我想按照工资由高到低前20% 的人，其实前面我们学习row_number 的时候也学习过类似的，只不过我们当时求的不是百分比，而是前面多少个`top-N`,这里我们的实现方式就是将数据按照工资从高到低分成5分，第一份就是前20的人

```sql
select
    *
from (
    select
        *,ntile(5) over(order by salary desc) as rn
    from
        ods_num_window
) tmp
where
      rn=1
;
```

![image-20210106092207179](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106092207179.png)



#### 求子窗口百分比`Top-%n`

其实这个和上面的有点类似，我们就是求每个部门的工资从高低的前20%

```sql
select
    *
from (
    select
        *,ntile(5) over(partition by dept order by salary desc) as rn
    from
        ods_num_window
) tmp
where
      rn=1
;

```

![image-20210106092347847](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106092347847.png)

#### 数据分片处理

有时候我们需要将数据分片，然后交给多个程序并行的去处理，我们一般都会按照ID 对数据进行分割，例如某一个范围的交给一个程序，另一个范围的交给另外一个程序去处理

 然而很多时候我们的ID 并不是连续的，并且最小值和最大值之间差异特别大，这个时候我们就不能直接通过(max-min)/n 这样的方式计算，这里我们演示一下如何使用ntile 完成这个需求

```sql
select
    rn ,min(id) as startId ,max(id) as endId
from (
    select
        id,ntile(5) over(order by id asc ) as rn
    from
        ods_num_window
) tmp
group by
    rn
;
```



![image-20210106094234348](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106094234348.png)

## 总结

1. ntile 如果输入数据不能被均匀切片，那么多余的数据(count(1)%n) 将会被均匀的分布在前几片上，例如我有109条要分10片，那么我余下的数据就是(109%10)=9 条，那这9条数据就会被均匀的分布在前面9片上
2. ntile 如果不指定数据分片的排序顺序，则是按照你输入数据的逆序进行排序然后分片
3. 我们介绍ntile 典型的两种应用场景，虽然是两种场景，但是都是基于数据分片的思想实现的