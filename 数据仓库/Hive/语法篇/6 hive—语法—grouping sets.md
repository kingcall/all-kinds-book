[TOC]

## GROUPING SETS

先说一下背景，我么知道GROUP BY 可以进行数据分组统计，我们将分组称之为统计的维度，例如 `GROUP BY school` 我们认为维度是学校，GROUP BY  也支持多个字段进行分组统计，例如``GROUP BY school,grade` 我们的维度就是`学校+年级`的组合，虽然是组合还是单维度的，组合在一起就一个依然是单个维度，因为统计出来的数据你只能得到每个学校的每个年级的信息，你得不到学校的单独统计信息也得不到年级的单独统计信息

```
学校名称	年级	学生数
清华小学  6			1000
北大小学  6			1300
北大小学  5			1600
南京小学  5     500
	............
```

就像上面的统计信息，如果你不做二次统计的话，你不能得到整个清华小学有多少人，你也得不到整个6年级有多少人

这就是我们说的维度，因为GROUP BY的维度是单一的，就是它只能计算某个维度的信息，而不能同时计算多个维度，在一个GROUPING SETS 查询中，根据不同的单维度组合进行聚合，等价于将不同维度的GROUP BY结果进行UNION ALL操作。

GROUPING SETS就是一种将多个GROUP BY逻辑UNION写在一个HIVE SQL语句中的便利写法。

GROUPING SETS会把在单个GROUP BY逻辑中没有参与GROUP BY的那一列置为NULL值，这样聚合出来的结果，未被GROUP BY的列将显示为NULL。

### 演示

我们先准备这样一份数据

```
school	grade	user
第一中学	一年级	张三1
第一中学	二年级	张三2
第一中学	三年级	张三3
第一中学	四年级	张三4
第一中学	五年级	张三5
第一中学	六年级	张三6
第二中学	一年级	李四1
第二中学	二年级	李四2
第二中学	三年级	李四3
第二中学	四年级	李四4
第二中学	五年级	李四5
第二中学	六年级	李四6
第三中学	一年级	王二1
第三中学	二年级	王二2
第三中学	三年级	王二3
第三中学	四年级	王二4
第三中学	五年级	王二5
第三中学	六年级	王二6
```

```sql
CREATE TABLE ods.ods_student (
  school string,
  grade string,
  `user` string
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
load data local inpath '/Users/liuwenqiang/workspace/hive/students.data' overwrite into table ods.ods_student;
```

首先我们查询一下每个学校每个年级有多少人，返回结果的格式是 `学校，年级，人数` 这样的格式 

```
select school,grade,count(1) as userCnt from ods.ods_student group by school,grade;
```

接下来我们希望同时查询出每个学校有多少人，然后和上面的查询结果一起返回，年级的位置是null

```
select school,grade,count(1) as userCnt from ods.ods_student group by school,grade
union all
select school,null,count(1) as userCnt from ods.ods_student group by school;
```

![image-20210101132922495](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210101132922495.png)



在上面的基础上，我们希望同时查询出每个年级有多少人，学校的位置设置为null

```
select school,grade,count(1) as userCnt from ods.ods_student group by school,grade
union all
select school,null,count(1) as userCnt from ods.ods_student group by school
union all
select null,grade,count(1) as userCnt from ods.ods_student group by grade;
```

其实看到这里我们也大概明白了，只要我们添加维度我们就在原来SQL基础上添加新的统计SQL,然后不参与计算的维度设置为NULL 就可以了

![image-20210101133105178](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210101133105178.png)



针对上面这种情况，Hive 为我们提供了 GROUPING SETS 函数，sets 你可以认为是group by 维度的集合，然后按照集合里的维度将每个维度的 group by 的结果合并起来

```sql
select school,grade,count(1) as userCnt from ods.ods_student group by school,grade grouping sets((school,grade),school,grade);
```

![image-20210101133239842](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210101133239842.png)

除了返回的顺序和上面实现的返回顺序不一样，结果值都是一样的，例如第一行的含义就是全部一年级的学生数量(所以学校的)，其实这我我们可以对NULL 值进行处理一下

```sql
select nvl(school,'全年级'),nvl(grade,'全学校'),count(1) as userCnt from ods.ods_student group by school,grade grouping sets((school,grade),school,grade);
```

![image-20210101133855807](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210101133855807.png)

这下输出就好看多了，也方便理解



### 语法规则

其实前面我们说过了，grouping sets 其实就是就将多个group by 的分组结果进行合并，虽然这个意义很明显个了，但是我们这里需要注意一些地方

1. grouping sets(dim1,dim2,dim3 ....) 每个维度都要么是在group by 中出现，要么是group by中的字端组合，例如 `group by school,grade` 然后grouping sets 总出现了`(school,grade)` 这样的组合
2. grouping sets(dim1,dim2,dim3 ....()) 中可以出现一个空的分组组合，这个的含义是全部的汇总，即不按照任何分组字段，也就是所有的分组字段的位置都是NULL

```sql
select nvl(school,'全年级'),nvl(grade,'全学校'),count(1) as userCnt from ods.ods_student group by school,grade grouping sets((school,grade),school,grade,());
```

![image-20210101191810718](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210101191810718.png)

在这里就是**全年级全学校**

### 实现原理

看了hive 中 grouping sets 实现（从源码及执行计划都可以看出与kylin实现不一样），并没有像kylin一样先按照group by 全字段聚合再上卷。

hive实现就是无脑复制，可以理解成是 group by grouping sets 所有组合 然后 在union 起来



### grouping__id 字段

在grouping sets做维度上卷的时候，其实就是有小维度到大维度的时候，每个维度都会分配一个ID,需要强调的时候每个维度都会分配一个ID,而不是每条记录，维度ID 从0 开始

```sql
select grouping__id, nvl(school,'全年级'),nvl(grade,'全学校'),count(1) as userCnt from ods.ods_student group by school,grade grouping sets((school,grade),school,grade,());
```

例如上面`grouping sets((school,grade),school,grade,());`总共有四个维度，分别是`(school,grade)`、`,school`、`grade`和`()`,所以这个四个维度的ID 依次是0 1 2 3

![image-20210101193013429](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210101193013429.png)

我们看到同一个维度的记录并没有全部放在一下，这个时候我们就可以利用这个字段进行排序

```sql
select grouping__id, nvl(school,'全年级'),nvl(grade,'全学校'),count(1) as usercnt from ods.ods_student group by school,grade grouping sets((school,grade),school,grade,()) order by grouping__id ;
```

![image-20210101193221062](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210101193221062.png)

## 总结

1. grouping sets 可以简化我们SQL 的写法，也更容易扩展，当维度多的情况下，优势更加明显，但是目前的实现并不能提高SQL 的查询性能，可能以后新版本的实现会优化这一点

2. grouping sets 在一个group by 的聚合查询中，根据不同的维度进行组合，我们可以选择性的去对维度聚合，也可以聚合所有维度，当维度上卷聚合时，维度值为 null，可用nvl函数去赋值，得到一个有意义的维度名称

3. grouping__id 是一个维度ID 字段，有时候我们可以利用其进行过滤和排序，这个字段的取值个grouping sets() 函数中参数的顺序保持一致

   

