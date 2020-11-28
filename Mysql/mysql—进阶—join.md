[toc]

left join :左连接，返回左表中所有的记录以及右表中连接字段相等的记录。

right join :右连接，返回右表中所有的记录以及左表中连接字段相等的记录。

inner join: 内连接，又叫等值连接，只返回两个表中连接字段相等的行。

full join:外连接，返回两个表中的行：left join + right join。

cross join:结果是笛卡尔积，就是第一个表的行数乘以第二个表的行数

left join exclude inner join

right join exclude inner join

## MySQL Join的底层实现原理
- mysql只支持一种join算法：Nested-Loop Join（嵌套循环连接），但Nested-Loop Join有三种变种：Simple Nested-Loop Join，Index Nested-Loop Join，Block Nested-Loop Join

### Simple Nested-Loop Join：
- r为驱动表，s为匹配表，可以看到从r中分别取出r1、r2、......、rn去匹配s表的左右列，然后再合并数据，对s表进行了rn次访问，对数据库开销大

![image-20201127213711927](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/21:37:13-image-20201127213711927.png)
```
For each row a in A do
    For each row b in B do
        If a and b satify the join condition
            Then output the tuple
```


### Index Nested-Loop Join（索引嵌套）

- 这个要求非驱动表（匹配表s）上有索引，可以通过索引来减少比较，加速查询。在查询时，驱动表（r）会根据关联字段的索引进行查找，当在索引上找到符合的值，再回表进行查询，也就是只有当匹配到索引以后才会进行回表查询。
- 如果非驱动表（s）的关联健是主键的话，性能会非常高，如果不是主键，要进行多次回表查询，先关联索引，然后根据二级索引的主键ID进行回表操作，性能上比索引是主键要慢。

![image-20201127213729499](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/21:37:30-image-20201127213729499.png)


### Block Nested-Loop Join
- 如果有索引，会选取第二种方式进行join，但如果join列没有索引，就会采用Block Nested-Loop Join。
- 可以看到中间有个join buffer缓冲区，是**将驱动表的所有join相关的列都**先缓存到join buffer中，然后批量与匹配表进行匹配，将第一种多次比较合并为一次，降低了非驱动表（s）的访问频率。
> 在外层循环扫描 A 中的所有记录。扫描的时候，会把需要进行 join 用到的列都缓存到 buffer 中。buffer 中的数据有一个特点，里面的记录不需要一条一条地取出来和 B 表进行比较，而是**整个 buffer 和 B 表进行批量比较**。

- 默认情况下join_buffer_size=256K，在查找的时候MySQL会将所有的需要的列缓存到join buffer当中，**包括select的列，而不是仅仅只缓存关联列**。
- 在一个有N个JOIN关联的SQL当中会在执行时候分配N-1个join buffer。

![image-20201127213757421](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/21:37:57-image-20201127213757421.png)

> 把驱动表放在内存也符合小表在前的特点

