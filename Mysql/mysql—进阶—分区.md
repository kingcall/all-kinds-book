## 分区
- 分区是一种表的设计模式，通俗地讲表分区是将一大表，根据条件分割成若干个小表。但是对于应用程序来讲，分区的表和没有分区的表是一样的。换句话来讲，分区对于应用是透明的，只是数据库对于数据的重新整理
- MySQL在创建表的时候可以通过使用PARTITION BY子句定义每个分区存放的数据。在执行查询的时候，优化器根据分区定义过滤那些没有我们需要的数据的分区，这样查询就可以无需扫描所有分区，只需要查找包含需要数据的分区即可。
- **分区的另一个目的是将数据按照一个较粗的粒度分别存放在不同的表中**。这样做可以将相关的数据存放在一起，另外，当我们想要一次批量删除整个分区的数据也会变得很方便。

## 分区类型
### RANGE分区
- 最为常用，基于属于一个给定连续区间的列值，把多行分配给分区。最常见的是基于时间字段

### LIST分区
- LIST分区和RANGE分区类似，区别在于LIST是枚举值列表的集合，RANGE是连续的区间值的集合。

### HASH分区
- 基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含MySQL中有效的、产生非负整数值的任何表达式。

### KEY分区
- 类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数。必须有一列或多列包含整数值。

> 上述四种分区类型中，RANGE分区 即范围分区是最常用的。RANGE分区的特点是多个分区的范围要连续，但是不能重叠，默认情况下使用VALUES LESS THAN属性，即每个分区不包括指定的那个值。

```
# 创建分区表
mysql> CREATE TABLE `tr` (
    ->   `id` INT, 
    ->   `name` VARCHAR(50), 
    ->   `purchased` DATE
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8
    -> PARTITION BY RANGE( YEAR(purchased) ) (
    -> PARTITION p0 VALUES LESS THAN (1990),
    -> PARTITION p1 VALUES LESS THAN (1995),
    -> PARTITION p2 VALUES LESS THAN (2000),
    -> PARTITION p3 VALUES LESS THAN (2005),
    -> PARTITION p4 VALUES LESS THAN (2010),
    -> PARTITION p5 VALUES LESS THAN (2015)
    -> );
Query OK, 0 rows affected (0.28 sec)

# 插入数据
mysql> INSERT INTO `tr` VALUES
    ->     (1, 'desk organiser', '2003-10-15'),
    ->     (2, 'alarm clock', '1997-11-05'),
    ->     (3, 'chair', '2009-03-10'),
    ->     (4, 'bookcase', '1989-01-10'),
    ->     (5, 'exercise bike', '2014-05-09'),
    ->     (6, 'sofa', '1987-06-05'),
    ->     (7, 'espresso maker', '2011-11-22'),
    ->     (8, 'aquarium', '1992-08-04'),
    ->     (9, 'study desk', '2006-09-16'),
    ->     (10, 'lava lamp', '1998-12-25');

# 查看某个分区的数据
-  SELECT * FROM tr PARTITION (p2);

# 增加分区
mysql> alter table tr add partition(
    -> PARTITION p6 VALUES LESS THAN (2020)
    -> );
Query OK, 0 rows affected (0.06 sec)
Records: 0  Duplicates: 0  Warnings: 0

# 拆分分区
mysql> alter table tr reorganize partition p5 into(
    ->   partition s0 values less than(2012),
    ->   partition s1 values less than(2015)
    -> );
Query OK, 0 rows affected (0.26 sec)
Records: 0  Duplicates: 0  Warnings: 0

# 合并分区
mysql> alter table tr reorganize partition s0,s1 into ( 
    ->     partition p5 values less than (2015) 
    -> );
Query OK, 0 rows affected (0.12 sec)
Records: 0  Duplicates: 0  Warnings: 0

# 清空某分区的数据
mysql> alter table tr truncate partition p0;
Query OK, 0 rows affected (0.11 sec)

# 删除分区
mysql> alter table tr drop partition p1;
Query OK, 0 rows affected (0.06 sec)
Records: 0  Duplicates: 0  Warnings: 0

# 交换分区
# 先创建与分区表同样结构的交换表
mysql> CREATE TABLE `tr_archive` (
    ->   `id` INT, 
    ->   `name` VARCHAR(50), 
    ->   `purchased` DATE
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.28 sec)
# 执行exchange交换分区
mysql> alter table tr exchange PARTITION p2 with table tr_archive;
```

## 分区表为什么不常用
- 分区字段的选择有限制。
- 若查询不走分区键，则可能会扫描所有分区，效率不会提升。
- 若数据分布不均，分区大小差别较大，可能性能提升也有限。
- 普通表改造成分区表比较繁琐。
- **需要持续对分区进行维护，比如到了6月份前就要新增6月份的分区**。
- 增加学习成本，存在未知风险。