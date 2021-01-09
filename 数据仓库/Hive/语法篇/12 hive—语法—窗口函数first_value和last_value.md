[TOC]

## first_value和last_value

今天我们再来学习两个窗口函数，first_value和last_value 可以实现截止到当前的top-1 和 last-1 ,因为是累积计算所以它是一个动态值

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



### 从例子中学习first_value和last_value



#### 计算

```
select
    *,
    first_value(id) over(partition by dept order by salary desc ) as first_id,
    row_number() over(partition by dept order by salary desc ) as rn
from
    ods_num_window
;
```



#### 计算截止到当前工资最高和最低的人

这里我们将工资降序排列，进行计算工资最高的人和最低的人

```sql
select
    *,
    first_value(id) over(order by salary desc ) as high_id,
    last_value(id) over(order by salary desc ) as low_id
from
    ods_num_window
;
```

![image-20210106104800264](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106104800264.png)

我们看到high_id一开始是9，一直到最后也是9，但是low_id呢一直在变化，这是因为我们是降序排列的，所以当只有前面第一行的时候，id 是9 的人既是工资最高的人，也是工资最低的人，但是当前面有两行的时候，工资最高的依然是第一样也是id 为9 的人，但是工资最低的却不是再是第一行了，而是第二行也就是id 为13 的人，其他的同理

这个first_value和last_value 定义中的截止到当前行的意思

#### 计算每个部门截止到当前工资最高和最低的人

同样你可以像我们上面那样去分析这个数据，看它是不是和你理解的一样

![image-20210106105918094](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106105918094.png)



#### 计算截止到当前工资最高和最低的人以及给出当前记录所在的一个排序值



![image-20210106110327051](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210106110327051.png)

有了上面这个运行结果，我们就可以更加容易理解first_value和last_value了，我们的rn 是我们的排序值，所以我们可以看出的high_id 一直就是我们的排名第一的值(rn=1),我们的low_id也一直就是我们到当前行排名最末的那行

## 总结

1. first_value和last_value 见名知意,就是计算当前窗口截止当前行的第一个值和最后一个值(或者是排名的第一名个最后一名)
2. 其实我们可以理解为取当前窗口截止当前行的最值(最大值和最小值)
3. 这两个窗口函数其实用的不多，也没什么常见的场景