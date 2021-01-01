[TOC]

##   cube 和 rollup

我们知道grouping sets 可以按照我们定义的维度(grouping sets的参数)进行分组统计,就像下面我们定义的维度就是`(school,grade),school,grade,()`,也就是说我们定义的什么维度就是什么维度，例如我们这里定义了四个就是四个，定义了一个就是一个

```sql
select
    grouping__id, nvl(school,'全年级'),nvl(grade,'全学校'),count(1) as userCnt
from
    ods.ods_student
group by
    school,grade
grouping sets(
       (school,grade),school,grade,()
)
order by
    grouping__id ;
```



### cube

例如上面在grouping sets中我们就是根据group by 的分组字段，然后按照我们的需求确定自己需要统计的维度，然后将其传给grouping sets函数，在上面的例子中就是`(school,grade),school,grade,()`

cube 在一个group by 的聚合查询中，将分组字段的全部组合作为维度，你可以认为是grouping sets的一种特殊情况，而不是像grouping sets那样，就是我们指定的几个维度

例如上面我们的分组字段是school,grade，那我们的cube组合指的就是`(school,grade),(school,null),(null,grade),(null,null)`，这个是等价于`(school,grade),school,grade,()`的

```
select
    grouping__id, nvl(school,'全年级'),nvl(grade,'全学校'),count(1) as userCnt
from
    ods.ods_student
group by
    school,grade
with cube
order by
    grouping__id ;
```

![image-20210101204249110](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210101204249110.png)

其实到这里我们也看出来了，cube 是grouping sets的特殊情况，那就是我们需要根据分组字段进行组合全部维度的时候，那这个时候使用cube可以简化我们SQL的书写，可以对比上面两条SQL 是等价的

尤其是当我们的分组字段比较多的时候，因为这个时候你去手动组合可能会漏掉一些特定的组合

这里我们学习注意一下，组合分组字段的方法（a,b）=> (a,b),(a,null),(null,b),(null,null)   



### rollup

是CUBE的子集，以最左侧的维度为主，从该维度进行层级聚合。左侧为空的时候右侧必须为空 ，例如`group by month,day 的rollup 维度是(month,day), (month,null), (null,null)` 而`group by month,day 的cube 维度是(month,day), (month,null), (null,day),(null,null)`

```sql
select
    grouping__id, nvl(school,'全年级'),nvl(grade,'全学校'),count(1) as userCnt
from
    ods.ods_student
group by
    school,grade
with rollup
order by
    grouping__id ;
```

根据我们对rollup 的说明，我们知道上面的SQL没有`null,grade` 这个维度，根据我们对grouping\__id 的定义，也就是没有grouping__id=2 的记录

![image-20210101205643911](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210101205643911.png)



## 总结

- grouping sets可以自由组合实现，也就是按照我们的定义进行任意维度的汇总

- cube 可以看做是grouping sets的特殊情况，那就是维度是group by字段的全部组合

- rollup 可以看做是cube子集合，去掉维度中左边字段是null,但是右边字段不是null 的情况

  