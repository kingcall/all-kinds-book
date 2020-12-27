[TOC]



## 一、视图

### 1.1 简介

Hive 中的视图和 RDBMS 中视图的概念一致，都是一组数据的**逻辑表示**，本质上就是由一条 SELECT 语句查询的结果集组成的虚拟表，在数据库中，存放的只是视图的定义，而不存放视图包含的数据项，这些项目仍然存放在原来的基本表结构中。

**视图的作用有：**

1. 可以简化数据查询语句(例如我们可以将一个复杂SQL中的一部分数据创建为一个视图)
2. 通过引入视图可以提高数据的安全性( 可以被定义为多个表的连接，也可以被定义为只有部分列可见，也可为部分行可见,这里我就可以让敏感数据不可见)

视图是纯粹的逻辑对象，没有关联的存储 (Hive 3.0.0 引入的物化视图除外)，当查询引用视图时，Hive 可以将视图的定义与查询结合起来，例如将查询中的过滤器推送到视图中。

### 1.2 创建视图

```sql
CREATE VIEW [IF NOT EXISTS] [db_name.]view_name   -- 视图名称
  [(column_name [COMMENT column_comment], ...) ]    --列名
  [COMMENT view_comment]  --视图注释
  [TBLPROPERTIES (property_name = property_value, ...)]  --额外信息
  AS SELECT ...;
复制代码
```

在 Hive 中可以使用 `CREATE VIEW` 创建视图，如果已存在具有相同名称的表或视图，则会抛出异常，建议使用 `IF NOT EXISTS` 预做判断。在使用视图时候需要注意以下事项：

- 视图是只读的，不能用作 LOAD / INSERT / ALTER 的目标；

- 在创建视图时候视图就已经固定，对基表的后续更改（如添加列）将不会反映在视图；

- 删除基表并不会删除视图，需要手动删除视图；

- 视图可能包含 ORDER BY 和 LIMIT 子句。如果引用视图的查询语句也包含这类子句，其执行优先级低于视图对应字句。例如，视图 `custom_view` 指定 LIMIT 5，查询语句为 `select * from custom_view  LIMIT 10`，此时结果最多返回 5 行。

- 创建视图时，如果未提供列名，则将从 SELECT 语句中自动派生列名；

- 创建视图时，如果 SELECT 语句中包含其他表达式，例如 x + y，则列名称将以_C0，_C1 等形式生成,有一点需要注意的是，C0 并不是从第一个表达式字段开始的，而是从第一个定义字段开始的

  ```sql
  CREATE VIEW  IF NOT EXISTS user_info_view AS SELECT user_id,cid,ckid,username,"man" ,1+2 from user_info;
  ```

  ![image-20201227204009933](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/20:40:10-image-20201227204009933.png)



```sql
select * from user_info_view;
```



![image-20201227204256751](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/20:42:57-image-20201227204256751.png)



### 1.3 查看视图

```sql
-- 查看所有视图： 没有单独查看视图列表的语句，只能使用 show tables
show tables;
-- 查看某个视图
desc view_name;
-- 查看某个视图详细信息
desc formatted view_name;
```

### 1.4 删除视图

```sql
DROP VIEW [IF EXISTS] [db_name.]view_name;
```

删除视图时，如果被删除的视图被其他视图所引用，这时候程序不会发出警告，但是引用该视图其他视图已经失效，需要进行重建或者删除。

### 1.5 修改视图

```sql
ALTER VIEW [db_name.]view_name AS select_statement;
```

被更改的视图必须存在，且视图不能具有分区，如果视图具有分区，则修改失败。

### 1.6 修改视图属性

语法：

```sql
ALTER VIEW [db_name.]view_name SET TBLPROPERTIES table_properties;
 
table_properties:
  : (property_name = property_value, property_name = property_value, ...)
```

示例：

```sql
ALTER VIEW user_info_view SET TBLPROPERTIES ('userName'='kingcall','date'='2020-12-27');
```

![image-20201227204651788](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/20:46:52-image-20201227204651788.png)



### 1.7 物化视图

普通视图它其实是一张虚表，在视图中不缓冲记录，也没有提高性能，而物化视图能够缓存数据，hive把物化视图当成一张"表"，将数据缓存到orc文件中(可以配置),这里我们做个测试，前面在讲[Hive streaming](https://blog.csdn.net/king14bhhb/article/details/111729038)的时候创建的测试数据，如果有需要可以去看这一节

首先我查询一张普通的表`select weekday,count(1) from ods.u_data_new group by weekday;` 可以看到耗时16.524 seconds

![image-20201227205200002](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/20:52:00-image-20201227205200002.png)

接下来我把它做成视图再查询一下`create view week_pv as select weekday,count(1) from ods.u_data_new group by weekday;`接下来我查询刚才的视图`select * from week_pv;`可以看到耗时几乎没变，甚至更长了15.611 seconds

其实这也可以立即，因为此时的SQL 其实等价于 `select * from (select weekday,count(1) from ods.u_data_new group by weekday)tmp` 的，所以时间长了可以理解

![image-20201227205434059](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/20:54:34-image-20201227205434059.png)

接下来我们看一下物化视图`create materialized view week_pv_materialized as select weekday,count(1) from ods.u_data_new group by weekday;` 然后你就得到了下面的错误信息

```
FAILED: SemanticException org.apache.hadoop.hive.ql.parse.SemanticException: Automatic rewriting for materialized view cannot be enabled if the materialized view uses non-transactional tables (state=42000,code=40000)
```

这里我们就不详细解释怎么去开启事务了，因为我们单独有一节就是将这个的，所以我们的重点还是回到物化视图上来。

既然如此我们就修改表的属性，使其支持事务` ALTER table ods.u_data_new SET TBLPROPERTIES ('transactional'='true');` 然后又报错了，说是存储类型必须是orc

```sql
Error: Error while processing statement: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. Unable to alter table. The table must be stored using an ACID compliant format (such as ORC): ods.u_data_new (state=08S01,code=1)
```

好的那我们就修改存储类型为orc `ALTER TABLE ods.u_data_new SET FILEFORMAT orc` 然后再去修改表使其支持事务，发现都成功了

![image-20201227211829587](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/21:18:30-image-20201227211829587.png)

接下来我们再去尝试创建物化视图，TMD 还是报错了

```
Error: Error while compiling statement: FAILED: SemanticException [Error 10265]: This command is not allowed on an ACID table ods.u_data_new with a non-ACID transaction manager. 
Failed command: create materialized view week_pv_materialized as select weekday,count(1) from ods.u_data_new group by weekday (state=42000,code=10265)
```

说我们没有事务管理器，看起来我们开需要配置hive 让其开启事务管理，但是这个平时用的不是很多，所以我们只在当前会话里设置一下

```
SET hive.support.concurrency = true;
SET hive.enforce.bucketing = true;
SET hive.exec.dynamic.partition.mode = nonstrict;
SET hive.txn.manager = org.apache.hadoop.hive.ql.lockmgr.DbTxnManager;
SET hive.compactor.initiator.on = true;
SET hive.compactor.worker.threads = 1;
```

这下应该差不多了吧，那我们再尝试创建一下我们的物化视图,发现还是报错了，这个时候我发现我们直接从`select * from ods.u_data_new;`这张表查询都报错了

```
Error: java.io.IOException: java.lang.RuntimeException: ORC split generation failed with exception: org.apache.orc.FileFormatException: Malformed ORC file hdfs://kingcall:9000/user/hive/warehouse/ods.db/u_data_new/000000_0. Invalid postscript. (state=,code=0)
```

我还是老老实实创建一个事务表吧，不修改了，然后我们将数据导入到下面这张表里

```sql
CREATE TABLE `ods.u_data_new_transactional`(
   `userid` int,             
   `movieid` int,            
   `rating` int,             
   `weekday` int
)
stored as orc
tblproperties('transactional'='true');
```

` insert overwrite table ods.u_data_new_transactional select * from ods.u_data_new;` 事务表创建好之后，我们再尝试创建一下物化视图`create materialized view week_pv_materialized as select weekday,count(1) from ods.u_data_new_transactional group by weekday;`

![image-20201227215328547](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/21:53:29-image-20201227215328547.png)

一番折腾终于成功的创建了物化视图，我们发现整个创建过程非常慢，耗时 17.836 seconds，这是因为物化视图创建的时候，query的执行数据自动落地，"自动"也即在query的执行期间，任何用户对该物化视图是不可见的。

创建成功之后，我们去Hdfs 上看，我们发现物化视图是会真正创建表的

![image-20201227215517268](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/21:55:17-image-20201227215517268.png)

接下来我们尝试从物化视图查询一下数据`select * from week_pv_materialized;`

![image-20201227215730444](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/21:57:31-image-20201227215730444.png)

发现非常快，这就是物化视图中，物化一词的体现。

## 二. Hive视图应用场景

- 当Hive中的查询变得很长或复杂时，通过视图将这个查询语句分割成多个小的、更可控的片段可以降低这种复杂度；
- 当Hive中需要通过视图限制基于条件过滤的数据时；

## 三. 总结

1. 物化视图是需要事务支持的
2. 物化视图需要事务表上创建