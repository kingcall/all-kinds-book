[toc]
## 窗口函数初始

窗口函数可以让**明细和聚合结果共存**，在sql中有一类函数叫做聚合函数,例如sum()、avg()、max()等等,这类函数可以将多行数据按照规则聚集为一行,一般来讲聚集后的行数是要少于聚集前的行数的，但是有时我们想要**既显示聚集前的数据,又要显示聚集后的数据**,这时我们便引入了窗口函数，**窗口函数是SQL语句最后执行的函数**而且仅位于Order by字句之前**，因此可以把SQL结果集想象成输入数据**

其实上面的描述信息，还是不能让我们很好的理解窗口函数到底是什么，但是我们知道了它可以让明细和聚合结果共存，其实这个有点和侧视图有点像，学习侧视图的时候我们说到，侧视图就是将一条记录输入，然后多条数据输出，然后原数数据的数据规模扩大n 倍，和多条输出进行匹配

![image-20201231162007648](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201231162007648.png)

![image-20201231162254979](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201231162254979.png)



窗口函数，就是在特定的数据上进行汇总计算，然后将原始明细数据和其对应的汇总信息进行一行输出

![image-20210104123211660](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210104123211660.png)



### 从例子中认识窗口函数

我们先看一些最直观的例子，再去探究冷冰冰的语法规则，原理什么的

#### 准备数据

建表语句

```sql
create table ods_t_window(
	name string,
	orderdate string,
	cost int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/workspace/hive/userorder.txt' OVERWRITE INTO TABLE ods_t_window;
```

数据

```#
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

#### 2015年4月份购买过的顾客及总人数

其实从上面的需求的定义中，我们就发现它同时需要购买过的顾客——明细信息，总人数——汇总信息

```sql
select 
	name,count(*) over ()
from 
	ods_t_window
where 
	substring(orderdate,1,7) = '2015-04'
;
```
![image-20210103202353919](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210103202353919.png)

从查询结果看2015年4月一共有5次购买记录,mart购买了4次,jack购买了1次.事实上,大多数情况下,我们是只看去重后的结果的.针对于这种情况,我们有两种实现方式

第一种实现方式,直接使用distinct 实现

```sql
select 
	distinct name,count(*) over ()
from 
	ods_t_window
where
	substring(orderdate,1,7) = '2015-04'
;
```

第二种实现方式是使用group by,这种写法性能更好

```sql
select 
	name,count(*)
from 
	ods_t_window
where
	substring(orderdate,1,7) = '2015-04'
group by
	name
;
```
![image-20210104085358524](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210104085358524.png)

去重之后我们就可以看到去重后的明细了

#### 顾客的购买明细以及总的月购买金额

```sql
select
    name,orderdate,cost,sum(cost) over(partition by month(orderdate))
from
    ods_t_window
;
```
![image-20210104090016791](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210104090016791.png)

其实这个时候我们可以稍微理解一下窗口这个东西了，首先窗口的计算函数，这里就是sum,我们知道sum 是一个聚合函数，一般情况下我们需要对一个数据集做计算，往往是一个完整的表或者是一个完整的分区，但是多以有over 标记的窗口函数而言，它计算的数据集合就是一个窗口内的数据，那么窗口函数将整个表，也就是`ods_t_window` 分成了很多窗口，怎么分的就是通过partition by 的定义分的，都是将数据按照边界值分组，而over之前的窗口计算函数在每一个分组之内进行，如果超出了分组，则函数会重新计算。

因为1月的数据都是在一月这个窗口内，所以`sum_window_0` 这个字段的汇总值是一样的，同理二月也可以这样理解



#### 顾客的购买明细以及累积的月购买金额

前面我们计算了总的月购买金额，我们看到一个窗口内，例如一月的总金额是一次性计算出来的，所以一月的数据的`sum_window_0` 字段都是相同的，也就是205

![image-20210104091823537](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210104091823537.png)

还有就是一月里的购买明细里时间也是乱序的，不是升序也不是降序,因为一月的数据都是在同一个窗口里，所以有时候我们希望它们在窗口内是有序的

```sql
select
    name,orderdate,cost,sum(cost) over(partition by month(orderdate) order by orderdate)
from
    ods_t_window
;
```
![image-20210104092157925](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210104092157925.png)

这下我们就看到这个数据在窗口内是有序的(如果窗口本身也是有序的话，那就是全局有序了)，而且`sum_window_0` 字段也是累加计算的，例如第二行的汇总值25 就是第一行的10+第二行的15计算出来的，同理你都可以这样去推



#### 顾客的购买明细以及该顾客过去2次的购买总金额

其实这个需求我们可以先进行简单分析一下，前面我们是计算月的总金额或者月的累计金额，我们计算对象是月，窗口都是对时间进行开窗的，一个窗口大小就是一个月`partition by month(orderdate)`,当前这个需求我们可以发现我们的统计信息是针对顾客的，也就是说我们的计算对象是顾客，也就是说我们要对顾客开窗

上面我们确定了开窗的对象，接下来我们分析一下过去3次怎么做，前面我们学习了累积的计算，使用的实order by 语句来完成的，它的计算方式是当前行所在窗口的全部数据(包括当前行)进行累加，也就是该窗口内的第一条数据到当前行(包括当前行)

```
+-------+-------------+-------+---------------+
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2015-01-01  | 10    | 10            | 当前行 第一行(10) 汇总值(10)
| tony  | 2015-01-02  | 15    | 25            | 当前行 第一行(10) + 第二行(15) 汇总值(25)
| tony  | 2015-01-04  | 29    | 54            | 当前行 第一行(10) + 第二行(15) + 第三行(29) 汇总值(54)
| jack  | 2015-01-05  | 46    | 100           |
| tony  | 2015-01-07  | 50    | 150           |
| jack  | 2015-01-08  | 55    | 205           |
| jack  | 2015-02-03  | 23    | 23            |从这里开始就是一个新的窗口了
| jack  | 2015-04-06  | 42    | 42            |
| mart  | 2015-04-08  | 62    | 104           |
| mart  | 2015-04-09  | 68    | 172           |
| mart  | 2015-04-11  | 75    | 247           |
| mart  | 2015-04-13  | 94    | 341           |
| neil  | 2015-05-10  | 12    | 12            |
| neil  | 2015-06-12  | 80    | 80            |
+-------+-------------+-------+---------------+
```

通过文字的解释和数据的阐述，相信大家对order by 的累积也有一定的了解了，现在我们要做的其实就是累加过去3条(包括当前行)，而不是累加窗口内的全部数据，我们请出今天的主角 window 子句，和patition by，order by  一样，都是窗口函数的一部分，window 可以定义我们计算窗口中的那些数据，默认情况下我们会计算该窗口内的全部数据，需要注意的是window 子句是以rows between  开头的

```
select
    name,orderdate,cost,
    sum(cost) over(partition by name order by orderdate rows between 2 PRECEDING and current row)
from
    ods_t_window
;
```

`rows between 2 PRECEDING and current row` 就是当前行的前面两行和当前行的定义

![image-20210104101422712](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210104101422712.png)

```
+-------+-------------+-------+---------------+
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2015-01-01  | 10    | 10            |第一行(10) 汇总值(10) 没有前面两行
| jack  | 2015-01-05  | 46    | 56            |第一行(10) 第二行(46) 汇总值(56)只有前面一行
| jack  | 2015-01-08  | 55    | 111           |第一行(10) 第二行(46) 第三行(55) 汇总值(111)  有前面两行
| jack  | 2015-02-03  | 23    | 124           |第二行(46) 第三行(55)  第四行(23) 汇总值(124)  有前面两行
| jack  | 2015-04-06  | 42    | 120           |第三行(55) 第四行(23)  第五行(42) 汇总值(120)  有前面两行
| mart  | 2015-04-08  | 62    | 62            |
| mart  | 2015-04-09  | 68    | 130           |
| mart  | 2015-04-11  | 75    | 205           |
| mart  | 2015-04-13  | 94    | 237           |
| neil  | 2015-05-10  | 12    | 12            |
| neil  | 2015-06-12  | 80    | 92            |
| tony  | 2015-01-02  | 15    | 15            |
| tony  | 2015-01-04  | 29    | 44            |
| tony  | 2015-01-07  | 50    | 94            |
+-------+-------------+-------+---------------+
```

上面我们通过分析给大家分析了一下，前面我们说到没有指定window 子句的默认是计算该窗口的全部数据的，如果指定了order by 的话，则是计算累积数据那是因为order by 默认的window 子句就是当前行之前的全部行`rows between UNBOUNDED PRECEDING and current row` ，这里有一个表达就是`UNBOUNDED PRECEDING ` 意思是前面全部数据，`n PRECEDING ` 就是前面n 行数据

现在我们看一下指定了order by 计算全部数据

```
select
    name,orderdate,cost,
    sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING and UNBOUNDED FOLLOWING)
from
    ods_t_window
;
```

![image-20210104103120344](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210104103120344.png)

那我们能不能手动指定一下累积的计算方式呢，就是手动指定一下order by 默认的window 子句，例如我们前面的第二个需求顾客的购买明细以及累积的月购买金额，下面是我们前面的写法

```
select
    name,orderdate,cost,sum(cost) over(partition by month(orderdate) order by orderdate)
from
    ods_t_window
;
```

现在我们尝试手动指定一下

```
select
    name,orderdate,cost,sum(cost) over(partition by month(orderdate) order by orderdate rows between UNBOUNDED PRECEDING and current row)
from
    ods_t_window
;
```

![image-20210104105905016](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210104105905016.png)



### 窗口函数的的定义

通过前面的例子学习，我们已经对窗口函数有了初步的认知，也介绍了可窗口函数可以让明细信息和汇总信息放在一起展示，这可以方便我们做很多事情，有一点还需要注意和区别的是窗口函数计算的对象和我们常规的汇总函数计算的对象是不一样的，窗口函数计算的对象是窗口内的数据，常规函数计算的是整个表或者分区

所以窗口函数我们可以认为是由两部分组成的，**第一部分就是计算函数，第二部分就是窗口的定义**

接下来我们就看一下窗口函数完整定义

![image-20210104112008927](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210104112008927.png)

窗口函数的定义就是我们上面截图上展示的这样的，下面我们一一介绍一下这些部分的定义

#### 窗口计算函数 function

窗口计算函数决定了我们对窗口内的数据执行什么样的计算，可以是我们常用的sum、count、min、max、avg 等函数

窗口计算函数是为了计算窗口里的数据的，所以下面我们介绍一下如何窗口的窗口内的数据——哪些数据属于一个窗口

#### 窗口标记(定义)over

over 我们可以认为是窗口函数的一个标记，它标记了我们这个函数是一个窗口函数而不是一个普通函数，所以窗口定义相关的参数都是以参数的形式传递给over 的

其实它更准确的来说是窗口的定义，而不是标记，里面定义了窗口数据的划分方式——partition by,窗口数据的计算方式——order by,窗口计算的边界rows between

#### 窗口定义边界 patition by

前面我们介绍了窗口计算函数是计算一个窗口内的数据的，那我们是泽阳定义一个窗口的呢？或者是我们认为什么样的数据是属于一个窗口的呢，那就是patition by 来定义了哪些数据属于一个窗口，如果没有 patition by 语句，那么整个数据源就是在一个窗口内

` over(partition by month(orderdate) order by orderdate)` 定义了`month(orderdate) `相等的数据属于一分区，也就是购买日期在一个同一个月的数据属于一个分区`  sum(cost) over(partition by name)` 定义了名字相等的数据属于一个分区

如果我们通过patition by 定义了窗口的划分方式，那我们的窗口计算函数就可以作用在一个个窗口上了，其实到这里我们看出我们一直说的窗口函数，它定义了窗口计算函数，然后作用于它定义的一些列窗口之上

#### 窗口计算方式 order by

order by  看起来像是排序，但是如果我们只是计算一个窗口内数据的汇总信息，窗口内的数据有没有顺序对我们是没有影响的，那为什么要排序呢，其实有一种汇总信息排序是对其有影响的，那就是累积的汇总数据，所以order by 一旦出现在了我们的窗口定义中，那我们的计算就变成累积计算了，至于是累积sum 还是累积count 取决于我们的窗口计算函数 是sum 还是count

order by 标志着使用累积计算，也就是改变了我们默认的计算patition by定义的该窗口内的全部数据的这一行为，所以它在窗口内数据的基础上，引进了另外一个计算的边界，原来的边界就是该窗口内的全部数据，也就是窗口的边界，那么order by 的计算边界到底是什么呢——就是当前行的之前的全部数据(包括当前行)



#### 窗口计算边界rows between

我们将rows between定义的数据范围又叫做window clause

前面我们介绍了order by会影响默认窗口计算函数的计算边界，并且它有自己的计算边界，其实它的计算边界就是通过rows between 进行限制的，只不过它提供了一个默认的rows between` rows between UNBOUNDED PRECEDING and current row`，所以我们就没有写,下面有几个关键字可以方便的让我们为去写一些window clause 语句

- PRECEDING：往前 
- FOLLOWING：往后 
- CURRENT ROW：当前行 
- UNBOUNDED：边界点(无限制的)
- UNBOUNDED PRECEDING 表示前面的边界
  - n PRECEDING(前面多少行，针对当前行而言的)
- UNBOUNDED FOLLOWING：表示到后面的边界
  - n FOLLOWING(后面多少行，针对当前行而言的)

下面我们提供一个例子

```sql
select 
  name,orderdate,cost,
  sum(cost) over() as sample1,--所有行相加
  sum(cost) over(partition by name) as sample2,--按name分组，组内数据相加
  sum(cost) over(partition by name order by orderdate) as sample3,--按name分组，组内数据累加
  sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING and current row )  as sample4 ,--和sample3一样,由起点到当前行的聚合
  sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING   and current row) as sample5, --当前行和前面一行做聚合
  sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING   AND 1 FOLLOWING  ) as sample6,--当前行和前边一行及后面一行
  sum(cost) over(partition by name order by orderdate rows between current row and UNBOUNDED FOLLOWING ) as sample7 --当前行及后面所有行
from 
	ods_t_window;
```

![image-20210104121017373](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210104121017373.png)



## 总结

窗口函数除了我们常见的统计函数+over 的这种我们姑且称之为通用窗口函数，和hive 还结合应用场景为我们提供了用于特定场景下的窗口函数，例如 row_number 我们称之为特殊窗口函数

**窗口函数应用场景：** 用于分区排序  动态Group By、 Top N、累计计算 、层次查询，hive中一般取top n时，row_number(),rank,dense_rank()这三个函数就派上用场了，后面我们会单独介绍全部的窗口函数

窗口函数中，窗口的定义中有的子句可以省略，如果只使用partition by子句,未指定order by的话,我们的聚合是分组内(特定窗口内)的聚合，如果使用了order by子句,未使用window子句的情况下,则是累积计算默认从起点到当前行，如果使用了window clause 子句的话则是累积计算我们定义的边界内的数据，如果连partition  by子句都没有的话，则是整个输入数据集放在一个窗口内计算



