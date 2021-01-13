[TOC]

## 窗口函数练习

窗口函数其实日常中用的是比较多的，加上之前我们分别介绍了各个窗口函数，今天我们就练习和总结一下,从而可以更好的掌握窗口函数

### 题目

#### 题目一：每个用户截止到每月为止的最大交易金额和该月的累积总交易金额

数据源格式如下

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

输出结果数据格式如下

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
| 1359014      | 2019-2         | 1                  | 69.99                       | 69.99                             |

**思路：**

- 1.对数据按照客户及其年-月分组
- 2.分组后就每月销售金额之和
- 3.使用窗口函数，对每个客户不同月份分组求最大值(max)和累计值(sum)

**解答:**

```sql
       select t.customer_key, t.umonth, t.ucount,
  max(current_max) over(partition by t.customer_key order by umonth) as current_max,
  sum(current_sum) over(partition by t.customer_key order by umonth) as current_sum
from
  (
     select 
        customer_key, substr(create_date,1,7) as umonth,
        count(sales_order_key) as ucount,
        max(unit_price) as current_max,
        sum(unit_price) as current_sum
    from 
      adventure_ods.ods_sales_orders
    group by 
      customer_key, substr(create_date,1,7)
  ) tpm;   　　　　　　　　　　　　　
```

备注：也可以使用with tmp as () 句法：

```sql
with tmp as (
       select 
          customer_key, substr(create_date,1,7) as umonth,
          count(sales_order_key) as ucount,
          max(unit_price) as current_max,
          sum(unit_price) as current_sum
      from 
        adventure_ods.ods_sales_orders
      group by 
        customer_key, substr(create_date,1,7)
  )
select 
  customer_key, umonth, ucount,
  max(current_max) over(partition by customer_key order by umonth) as current_max,
  sum(current_sum) over(partition by customer_key order by umonth) as current_sum
from tmp limit 10;
```

点评：

上面的写法就是为了窗口函数而窗口函数的，而且还计算错了你看到最后的结果是累积和最大是相等的，问题出在那里了呢，1 这个需求是一个汇总性质的结果，所以我们可以不用使用窗口函数 2 子查询过后每个用户每个月就只有一条数据了

```sql
select 
		customer_key, substr(create_date,1,7) as umonth,
    count(sales_order_key) as ucount,
    max(unit_price) as current_max,
    sum(unit_price) as current_sum
from 
	adventure_ods.ods_sales_orders
group by 
	customer_key, substr(create_date,1,7)
```

这样就可以了

#### 题目二：求用户号对应不同的产品

数据源格式如下

| 用户号 | 产品 | 购买时间   |
| ------ | ---- | ---------- |
| 1      | A    | 2019-12-23 |
| 1      | B    | 2019-12-23 |
| 2      | C    | 2019-12-23 |
| 2      | A    | 2019-12-24 |
| 2      | B    | 2019-12-23 |

```
create table ods_user_product_log(
    userid string,
    product string,
    ctm string
) row format delimited fields terminated by ',';
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/workspace/hive/ods_user_product_log.txt' OVERWRITE INTO TABLE ods_user_product_log;
```

 

**要求输出例子**:用户号-产品1-产品2(前两个产品)

**例如：**1-A-B (按先后时间顺序，实现相同时不限定顺序)

**思路**:

- 1.利用窗口函数，对用户号分组，按时间对产品进行排序
- 2.利用左连或其他方法拼接，然后进行筛选
- 3.用concat或者其他函数拼接获得结果

```sql
select
    userid,product,row_number() over (partition by userid order by ctm) as rn
from
     ods_user_product_log
;
```

![image-20210109111333057](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210109111333057.png)

 接下来我们可以使用自关联来获取下一个使用的产品，因为是自关联所以我们可以使用with as 的写法

```sql
with tmp as (
    select
        userid,product,row_number() over (partition by userid order by ctm) as rn
    from
        ods_user_product_log
)
select
    a.userid,a.product,b.product
from
     tmp a
inner join
    tmp b
on
    a.userid=b.userid
where 
    a.rn=1
    and b.rn=2
;
```

![image-20210109112136015](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210109112136015.png)

接下来你只要使用concat_ws 就可以了

其实前面我们说的 lead,lag 在很多场合下可以替代自关联，接下来我们看看怎么使用lead来完成上面的需求

```
select concat_ws('-', a.userid, a.product, a.next_product)
from (
     select a.userid,
            a.product,
            lead(product, 1) over (partition by userid order by ctm) as next_product
     from ods_user_product_log a
) a
where next_product is not null
;
```

![image-20210109112904606](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210109112904606.png)

上面的结果还是需要再处理一步

但是需要注意的是， lead,lag 虽好，但是就上面这个例子，如果我要计算的不是前两个产品，而是全部产品，那你就要使用自关联了



#### 题目三：查询5月份购买过的顾客及总人数

其实我们看到这个题目就是一个普通的聚合操作，因为这里是顾客明细，不是购买明细

```sql
select
    a.customer_key,count(customer_key)
from
    ods_sales_orders a
where
	month(create_date)='05'
group by
    a.customer_key
;
```

当然如果你是在想用窗口实现也不是不可以

```sql
select 
	customer_key,
	count(*) over() 
from 
	ods_sales_orders
where 
	month(create_date)="05"
group by 
	customer_key
; 
```



#### 题目四：查询顾客的购买明细及月购买总额

这才是一个非常典型的窗口函数应用，明细和汇总数据都需要的场景

 ```sql
select 
	* ,
	sum(unit_price) over(partition by customer_key,substr(create_date,1,7))
from 
	ods_sales_orders 
;
 ```



#### 题目五：查询顾客的购买明细及当月累积购买总额和月购买总额

```sql
select 
	* ,
  sum(unit_price) over(partition by customer_key,substr(create_date,1,7) sort by create_date rows between unbounded preceding and current row ) as sumcost
from 
	ods_sales_orders
```



#### 题目六：查询顾客上次的购买时间

**提示**：使用偏移的窗口函数lag()

```sql
select 
	* ,
  lag(create_date,1) over(partition by customer_key sort by create_date) as last_time
from 
	ods_sales_orders
```



#### 题目七：查询最近前20%时间的订单信息

**提示**：使用ntile(x)：分割x份。x是整数。

用ntile函数将订单时间按顺序分为5堆

```sql
select * from (
  select
  	*,
    ntile(5) over(sort by create_date asc)  as five_num
  from 
  	ods_sales_orders
 ) t
where 
five_num = 1
```

 

### 总结

#### 窗口函数应用场景

**（1）用于分区排序**

**（2）动态Group By**

**（3）Top N** 

**（4）累计计算**

**（5）层次查询**

#### 常见的窗口函数

**汇总函数:**

1. sum(col) over() :  分组对col求和，over() 中的语法如下
2. count(col) over() : 分组对col求个数，over() 中的语法如下
3. min(col) over() : 分组对col求最小值
4. max(col) over() : 分组求col的最大值
5. avg(col) over() : 分组求col列的平均值

**获取特定记录函数：**

1. first_value(col) over() : 某分区排序后的第一个col值
2. last_value(col) over() : 某分区排序后的最后一个col值
3. lag(col,n,DEFAULT) : 统计往前n行的col值，n可选，默认为1，DEFAULT当往上第n行为NULL时候，取默认值，如不指定，则为NULL
4. lead(col,n,DEFAULT) : 统计往后n行的col值，n可选，默认为1，DEFAULT当往下第n行为NULL时候，取默认值，如不指定，则为NULL

**分片函数：**

1. ntile(n) : 用于将分组数据按照顺序切分成n片，返回当前切片值。注意：n必须为int类型。

**排名函数**：

1. row_number() over() : 排名函数，不会重复，适合于生成主键或者不并列排名
2. rank() over() :  排名函数，有并列名次，名次不连续。如:1,1,3
3. dense_rank() over() : 排名函数，有并列名次，名次连续。如：1，1，2

**计算百分比函数：**

1. cume_dist
2. percent_rank