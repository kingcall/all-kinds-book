## 背景

日常工作中有许多数据处理需求需要解决，在此之间，获得需求，用hive实现需求，最终完成任务。



https://www.jianshu.com/p/90d0657c0218

https://www.cnblogs.com/chentianwei/p/12677902.html

https://blog.csdn.net/weixin_45941889/article/details/107348238

https://www.jianshu.com/p/3f3cf58472ca



## 题目

**数据源在：hive中的adventure_ods库的ods_sales_orders表**

| 表名             | 表注释     | 字段                 | 字段注释 |                          |
| ---------------- | ---------- | -------------------- | -------- | ------------------------ |
| ods_sales_orders | 订单明细表 | sales_order_key      | 订单主键 | 一个订单表示销售一个产品 |
| ods_sales_orders | 订单明细表 | create_date          | 订单日期 |                          |
| ods_sales_orders | 订单明细表 | customer_key         | 客户编号 |                          |
| ods_sales_orders | 订单明细表 | product_key          | 产品编号 |                          |
| ods_sales_orders | 订单明细表 | english_product_name | 产品名   |                          |
| ods_sales_orders | 订单明细表 | cpzl_zw              | 产品子类 |                          |
| ods_sales_orders | 订单明细表 | cplb_zw              | 产品类别 |                          |
| ods_sales_orders | 订单明细表 | unit_price           | 产品单价 |                          |

 

### 题目一：每个用户截止到每月为止的最大交易金额和累计到该月的总交易金额，结果数据格式如下

| customer_key | umonth（当月） | ucount（月订单量） | current_max（最大交易金额） | current_sum（累计该月总交易金额） |
| ------------ | -------------- | ------------------ | --------------------------- | --------------------------------- |
| 11009        | 2018-12        | 1                  | 53.99                       | 53.99                             |
| 1358999      | 2019-2         | 1                  | 28.99                       | 28.99                             |
| 1358999      | 2019-4         | 1                  | 69.99                       | 98.98                             |
| 1359000      | 2019-1         | 1                  | 2294.99                     | 2294.99                           |
| 1359002      | 2019-11        | 1                  | 8.99                        | 8.99                              |
| 1359003      | 2020-1         | 1                  | 1120.49                     | 1120.49                           |
| 1359005      | 2019-2         | 1                  | 782.99                      | 782.99                            |
| 1359009      | 2019-1         | 1                  | 2384.07                     | 2384.07                           |
| 1359014      | 2019-1         | 1                  | 69.99                       | 69.99                             |
| 1359014      | 2019-2         | 1                  | 69.99                       | 94.98                             |

**思路：**

- 1.对数据按照客户及其年-月分组
- 2.分组后就每月销售金额之和
- 3.使用窗口函数，对每个客户不同月份分组求最大值(max)和累计值(sum)

 

### 题目二：计算用户的回购率和复购率

**复购率:** 当前月份购买2次及以上的客户占所有客户比例

**回购率：**当前月份购买且上个月份也购买的客户占当月所有月份客户比例

**思路**：

复购率分析过程：

1. 分组：每个客户+月；聚合函数：count(当月购买次数)
2. 新表：分组：月； 聚合函数：count(条件) / count(*)

回购率

- 1、筛选当月及上月部分
- 2、利用客户id进行当月连上月，推荐左连
- 3、对同一条客户id均有购买记录的，认为是回购群体

```
#1 根据客户号和购买的月份分组：
select customer_key, substr(create_date, 1, 7) as umonth
from adventure_ods.ods_sales_orders
group by customer_key, substr(create_date, 1, 7);

#2 利用笛卡尔乘积： 相同两个表关联，然后使用关联条件。筛选出当前月关联上个月的数据。
select * 
from () a left join () b
on a.customer_key = b.customer_key and a表月份 = b表月份 - 1

#3 新的连和表，以a的月份排序。进行count计算
```



### 题目三：求用户号对应不同的产品 ⚠️比较难的题目，没实际意义。

| 用户号 | 产品 | 购买时间   |
| ------ | ---- | ---------- |
| 1      | A    | 2019-12-23 |
| 1      | B    | 2019-12-23 |
| 2      | C    | 2019-12-23 |
| 2      | A    | 2019-12-24 |
| 2      | B    | 2019-12-23 |

 

**要求输出例子**:用户号-产品1-产品2

**例如：**1-A-B (按先后时间顺序，同时不限定)

**参考:**https://www.jianshu.com/p/90d0657c0218

**思路**:

- 1.利用窗口函数，对用户号分组，按时间对产品进行排序
- 2.利用左连或其他方法拼接，筛选排序顺序为1、2的
- 3.用concat或者其他函数拼接获得结果

 

### 题目四：查询每年5月份购买过的顾客及总人数

**思路：**

- 1.筛选月份为5月
- 2.对客户去重或分组操作
- 3.计算总人数，拼接

###  

### 题目五：查询顾客的购买明细及月购买总额

**备注：**

- **每条记录后附加一个字段：这条记录所在月的总购买额。**
- **可以使用窗口函数。**

 

 

### 题目六：上述的场景,要将unit_price 按照日期进行累加

**参考链接**:https://www.iteye.com/blog/yugouai-1908121

其中涉及的ROWS UNBOUNDED PRECEDING

###

### 题目七：查询顾客上次的购买时间

**提示**：使用偏移的窗口函数lag()。

```
LAG(value_expr [, offset ] [, default ])
   OVER ([ query_partition_clause ] order_by_clause)
```

- a analytic function。通过query子句得到一系列行，然后把这个系列行排序；
- lag()告诉程序要查找的范围是当前行前面的部分，
  - 参数value_expr这个行的一个字段值。
  - 参数offset相当于指针(默认值为1)，返回当前行前面的n个的行中的value。
  - 参数default, 自定义的值，代表如果查找失败，返回default值。

例子：

```
SELECT last_name, hire_date, salary,
   LAG(salary, 1, 0) OVER (ORDER BY hire_date) AS prev_sal
   FROM employees
   WHERE job_id = 'PU_CLERK';
   
LAST_NAME                 HIRE_DATE     SALARY   PREV_SAL
------------------------- --------- ---------- ----------
Khoo                      18-MAY-95       3100          0
Tobias                    24-JUL-97       2800       3100
Baida                     24-DEC-97       2900       2800
Himuro                    15-NOV-98       2600       2900
Colmenares                10-AUG-99       2500       2600
```



 

### 题目八：查询最近前20%时间的订单信息

**提示**：使用ntile(x)：分割x份。x是整数。

```
NTILE(x)
   OVER ([ query_partition_clause ] order_by_clause)
```

-   Divides an ordered partition into x groups called buckets and assigns a bucket number to each row in the partition.
-   This allows easy calculation of tertiles, quartiles, deciles, percentiles and other common summary statistics. 

 

## 答案

### 题目一：计算每个用户截止到每月为止的最大交易金额和累计到该月的总交易金额

第一步：提取需要字段及按客户id、年月分组，求分组后的订单量及消费金额



```
select customer_key,
substr(create_date, 1,7) as umonth,
count(sales_order_key) as ucount,
sum(unit_price) as usum
from adventure_ods.ods_sales_orders
group by customer_key, substr(create_date, 1,7)   ⚠️group by内可以用表达式，但是不能用别名，否则报错❌。
order by customer_key asc , umonth asc   　　　　   ⚠️，这句话不加也可以，默认就是这个排序。
limit 10;
```



 

第二步：利用窗口函数，对客户按照月份排序，求最大金额及累积金额。



```
select t.customer_key, t.umonth, t.ucount,
  max(usum) over(partition by t.customer_key order by umonth) as current_max,
  sum(usum) over(partition by t.customer_key order by umonth) as current_sum
from
  (select customer_key,
    substr(create_date, 1,7) as umonth,
    count(sales_order_key) as ucount,
    sum(unit_price) as usum
  from adventure_ods.ods_sales_orders
  group by customer_key, substr(create_date, 1,7)
  ) as t limit 10;   　　　　　　　　　　　　　　⚠️👆的列名，不加别名t也可以
```



参考: Hive分析窗口函数(一) SUM,AVG,MIN,MAX https://www.cnblogs.com/qingyunzong/p/8782794.html

备注：也可以使用with tmp as () 句法：



```
with 
  tmp as (
    select customer_key, substr(create_date,1,7) as umonth,
      count(sales_order_key) as ucount,
      sum(unit_price) as income_per_month
    from adventure_ods.ods_sales_orders
    group by customer_key, substr(create_date,1,7)
  )
select 
  customer_key, umonth, ucount,
  max(income_per_month) over(partition by customer_key order by umonth) as current_max,
  sum(income_per_month) over(partition by customer_key order by umonth) as current_sum
from tmp limit 10;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

⚠️：再次⏰，窗口函数的作用范围，有partition by,但没有order by则函数作用范围就是当前分组的全部行。

 

 

### 题目二：计算用户的回购率和复购率

复购率:

 \1. 明确问题，求2020年2月的复购率。首先聚合操作：

```
select customer_key,
  count(sales_order_key) as ncount
from adventure_ods.ods_sales_orders
where substr(create_date, 1, 7) = '2020-02'
group by customer_key
```

\2. 进行计算操作：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
select count(ncount), count(if(ncount>1, 1, null)), count(if(ncount>1,1,null))/count(ncount) as ratio
from
(select customer_key,
  count(sales_order_key) as ncount
from adventure_ods.ods_sales_orders
where substr(create_date, 1, 7) = '2020-02'
group by customer_key) t
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

备注：计算一个月复购率和所有月的复购率的方法类似，下面是计算所有月的复购率：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
with 
  tmp as (
    select 
      customer_key,
      substr(create_date,1,7) as umonth,
      count(sales_order_key) as purchase_num
    from adventure_ods.ods_sales_orders
    group by customer_key, substr(create_date,1,7))
select 
  tmp.umonth,
  count(if(tmp.purchase_num > 1, 1, null)) as a,  --购买多次的客户的数量  
  count(tmp.customer_key) as b     ,            --所有购买的客户的数量
  concat(round((count(if(tmp.purchase_num >1, 1, null))/count(tmp.customer_key))*100, 2), "%") as ratio  --相除:复购率
from tmp
group by tmp.umonth;
```

-- 返回结果
-- umonth a b ratio
-- 2018-12 0 1 0.0%
-- 2019-01 635 11628 5.46%
-- 2019-02 304 10784 2.82%
-- 2019-03 257 12034 2.14%
-- 2019-04 174 11722 1.48%
-- 2019-05 153 12141 1.26%
-- 2019-06 112 11796 0.95%
-- 2019-07 104 12190 0.85%
-- 2019-08 92 12209 0.75%
-- 2019-09 63 11826 0.53%
-- 2019-10 68 12226 0.56%
-- 2019-11 78 13297 0.59%
-- 2019-12 61 12229 0.5%
-- 2020-01 49 12249 0.4%
-- 2020-02 41 11059 0.37%
-- Time taken: 3.967 seconds, Fetched: 15 row(s)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1276550/202004/1276550-20200412171313926-235473313.gif)

 

### 回购率:

我的计算：指定2020-02的回购率



```
#求得了得到2月和1月都有购买的人数same_num
select
  count(*)
from
  (select customer_key
  from adventure_ods.ods_sales_orders
  where substr(create_date, 1, 7) = '2020-01'
  group by customer_key) a1
inner join
  (select customer_key
  from adventure_ods.ods_sales_orders
  where substr(create_date, 1, 7) = '2020-02'
  group by customer_key
  ) a2
on a1.customer_key = a2.customer_key

# 计算2月的购买人数
select count(*)
from
  (select customer_key
  from adventure_ods.ods_sales_orders
  where substr(create_date, 1, 7) = '2020-02'
  group by customer_key) a

# 最后2个数字相除即可。
```



 

#### 计算全表的每个月的回购率：

方法1：正常思路下的方法：（推荐）✅

- tmp1: 每个客户的不同月份的购买记录。-》
- tmp2:得出连续2个月都购买的客户名单，即回购客户名单。-》
- tmp3:计算每个月，回购客户的数量。
- tmp4:计算每个月，当月的客户数量。
- 最后，tmp3/tmp4=回购率表。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
with 
  tmp as (
    select 
      customer_key,
      substr(create_date,1,7) as umonth
    from adventure_ods.ods_sales_orders
    group by customer_key, substr(create_date,1,7) ),
  tmp2 as (
    -- 内连接，得到连续两个月都消费的人的记录
    select a1.*                                  
    from tmp as a1 inner join tmp as a2          
    on a1.customer_key = a2.customer_key
    and substr(a1.umonth,6,2)  == substr(a2.umonth,6,2) - 1  --保证连续月
    and substr(a1.umonth,1,4)  == substr(a2.umonth,1,4)     --保证是当年的
  ),
  tmp3 as ( 
    --分子 ， 由👆2个表得到✅
    select umonth, count(customer_key) as active_customer
    from tmp2
    group by umonth),
  tmp4 as (
    --分母 按月分组,统计每个月购买的人数。✅
    select 
      substr(create_date,1,7) as umonth,
      count(distinct customer_key) as num
    from  adventure_ods.ods_sales_orders
    group by substr(create_date,1,7)
  )
select 
  tmp4.umonth, 
  tmp3.active_customer as active_customer,
  tmp4.num as current_customer,
  concat(round((tmp3.active_customer / tmp4.num)*100, 2), "%") as ratio
from tmp4 left join tmp3
on tmp4.umonth = tmp3.umonth;
-- tmp4.umonth    rcount    lcount    ratio
-- 2018-12    NULL    1    NULL
-- 2019-01    600    11628    5.16%
-- 2019-02    423    10784    3.92%
-- 2019-03    353    12034    2.93%
-- 2019-04    291    11722    2.48%
-- 2019-05    240    12141    1.98%
-- 2019-06    182    11796    1.54%
-- 2019-07    189    12190    1.55%
-- 2019-08    156    12209    1.28%
-- 2019-09    132    11826    1.12%
-- 2019-10    161    12226    1.32%
-- 2019-11    112    13297    0.84%
-- 2019-12    NULL    12229    NULL
-- 2020-01    86    12249    0.7%
-- 2020-02    NULL    11059    NULL
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

方法2:不使用with, 用左连接。

第一步：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#笛卡尔乘积去重
## 相同的表做连接，然后去重。
## substr(a.umonth,6,2) =  (substr(b.umonth,6,2) -1) 表示当月和上个月的关联。

select * from 
(select  customer_key,substr(create_date,1,7) as umonth from ods_sales_orders
group by customer_key,substr(create_date,1,7)) a
left join 
(select  customer_key,substr(create_date,1,7) as umonth from ods_sales_orders
group by customer_key,substr(create_date,1,7)) b 
on a.customer_key = b.customer_key and substr(a.umonth,6,2) =  (substr(b.umonth,6,2) -1) and substr(a1.umonth,1,4)  == substr(a2.umonth,1,4)     --保证是当年的
limit;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

第二步：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# 根据上面的表，以a.umonth分组,然后进行计算。
 select 
  a.umonth,
  count(a.customer_key) as mcount,
  count(b.customer_key) as lcount,
  concat(round((count(b.customer_key)/count(a.customer_key))*100,2),"%") as ratio  '''计算字段内的计算不能用别名'''
 from 
 (
 (select  customer_key,substr(create_date,1,7) as umonth from ods_sales_orders
 group by customer_key,substr(create_date,1,7)) a
 left join 
 (select  customer_key,substr(create_date,1,7) as umonth from ods_sales_orders
 group by customer_key,substr(create_date,1,7)) b 
 on a.customer_key = b.customer_key and substring(a.umonth,6,2) =  (substring(b.umonth,6,2) -1) )  and substr(a1.umonth,1,4)  == substr(a2.umonth,1,4)
group by a.umonth; 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 ⚠️连接表之后得到一个新的表。a.customer_key, b.customer_key都是指这个新表的列的数据。

因此，b.customer_key代表的是当前月和上个月都购买的客户的数据。

 

### 题目三：求用户号对应不同的产品

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
with 
  tmp as(
    select 
      customer_key,
      cpzl_zw,order_num,cpzl_zw1 
    from  
      (SELECT customer_key ,
        cpzl_zw,
        row_number() over(partition by customer_key order by create_date asc) as order_num,
        lag(cpzl_zw,1,null) OVER(partition by customer_key order by create_date asc) AS cpzl_zw1 
      from ods_sales_orders) as a 
    where cpzl_zw != cpzl_zw1),
  tmp2 as (
    select customer_key,cpzl_zw,order_num,cpzl_zw1,
    row_number() over(partition by customer_key order by order_num ) as cpzl_zw_num
    from tmp)
select concat( customer_key,'-',concat_ws('-', collect_set(cpzl_zw)) )
from tmp2 where cpzl_zw_num <3
group by customer_key;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

-  lag窗口函数的用法
-  concat()用法。

 

### 题目四：查询每年5月份购买过的顾客及总人数

over() 指定函数工作的数据窗口大小, over()内为空，则范围是所有行。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
select customer_key,
count(*) over() 
from ods_sales_orders
where month(create_date)="5"
group by customer_key; 
 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

### 题目五：查询顾客的购买明细及月购买总额

```
select * ,sum(unit_price) over(partition by substr(create_date,1,7)) from ods_sales_orders limit 10;
```

 

 

### 题目六：上述的场景,要将unit_price 按照日期进行累加

```
select * ,
  sum(unit_price) over(sort by create_date rows between unbounded preceding and current row ) as sumcost
from adventure_ods.ods_sales_orders
```

  

### 题目七：查询顾客上次的购买时间

 

```
select *, 
lag(create_date,1) over(distribute by customer_key sort by create_date) from ods_sales_orders limit 10;
```

 

 

 

### 题目八：查询最近前20%时间的订单信息

用ntile函数将订单时间按顺序分为5堆

```
select * from (
  select *,
    ntile(5) over(sort by create_date asc)  as five_num
  from adventure_ods.ods_sales_orders) t
where five_num = 1
```

 

 