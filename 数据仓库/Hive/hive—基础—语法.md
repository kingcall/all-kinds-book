[toc]

## 自定义函数
- UDF 一进一出 处理原文件内容某些字段包含 [] 
- UDAF 多进一出 sum() avg() max() min()
- UDTF 一进多出 ip -> 国家 省 市
## 4种排序
###  order by 
- 可以指定desc 降序 asc 升序
- order by会对输入做全局排序，因此只有一个Reducer(多个Reducer无法保证全局有序)，然而只有一个Reducer，会导致当输入规模较大时，消耗较长的计算时间。

### sort by 
-对分区内的数据进行排序
- sort by不是全局排序，其在数据进入reducer前完成排序
- 因此，如果用sort by进行排序，并且设置mapred.reduce.tasks>1，则sortby只会保证每个reducer的输出有序，并不保证全局有序。
- sort by不同于order by它不受Hive.mapred.mode属性的影响，sort by的数据只能保证在同一个reduce中的数据可以按指定字段排序。使用sort by你可以指定执行的reduce个数(通过set mapred.reduce.tasks=n来指定)，对输出的数据**再执行归并排序**，即可得到全部结果。

### distribute by 
- 【对map输出进行分区】
- distribute by是控制在map端如何拆分数据给reduce端的。hive会根据distribute by后面列，对应reduce的个数进行分发，默认是采用hash算法。使用DISTRIBUTE BY 可以保证相同KEY的记录被划分到一个Reduce 中。
- sort by为每个reduce产生一个排序文件。
- 在有些情况下，你需要控制某个特定行应该到哪个reducer，这通常是为了进行后续的聚集操作。distribute by刚好可以做这件事。因此，distribute by经常和sort by配合使用。

#### 使用distribute by 防止数据倾斜
- distribute by ：用来控制map输出结果的分发，即map端如何拆分数据给reduce端。 会根据distribute by 后边定义的列，根据reduce的个数进行数据分发，默认是采用hash算法。
- 当 distribute by 后边跟的列是：rand()时，即保证每个分区的数据量基本一致。

### cluster by
- cluster by除了具有distribute by的功能外还兼具sort by的功能。当distribute by和sort by 是同一个字段的时候可以使用cluster by替代。**但是排序只能是倒叙排序，不能指定排序规则为ASC或者DESC**。

## 三种分组的区别
### row_number
- 不管col2字段的值是否相等，行号一直递增，比如：有两条记录的值相等，但一个是第一，一个是第二
### rank
- 上下两条记录的col2相等时，记录的行号是一样的，但下一个col2值的行号递增N（N是重复的次数），比如：有两条并列第一，下一个是第三，没有第二

### dense_rank
- 上下两条记录的col2相等时，下一个col2值的行号递增1，比如：有两条并列第一，下一个是第二

## 3种常见的join
### Map-side Join
- mapJoin的主要意思就是，当链接的两个表是一个比较小的表和一个特别大的表的时候，我们把比较小的table直接放到内存中去，然后再对比较大的表格进行map操作。join就发生在map操作的时候，每当扫描一个大的table中的数据，就要去去查看小表的数据，哪条与之相符，继而进行连接。这里的join并不会涉及reduce操作。map端join的优势就是在于没有shuffle，真好。在实际的应用中，我们这样设置：set hive.auto.convert.join=true;
```
DistributedCache是分布式缓存的一种实现，它在整个MapReduce框架中起着相当重要的作用，他可以支撑我们写一些相当复杂高效的分布式程

这里的第一句话就是运行本地的map join任务，继而转存文件到XXX.hashtable下面，在给这个文件里面上传一个文件进行map join，之后才运行了MR代码去运行计数任务。说白了，在本质上mapjoin根本就没有运行MR进程，仅仅是在内存就进行了两个表的联合。
```

### Reduce-side Join
- hive join操作默认使用的就是reduce join
- Reduce-side Join原理上要简单得多，它也不能保证相同key但分散在不同dataset中的数据能够进入同一个Mapper，整个数据集合的排序在Mapper之后的shuffle过程中完成。
- 相对于Map-side Join，它不需要每个Mapper都去读取所有的dataset，这是好处，但也有坏处，
即这样一来Mapper之后需要排序的数据集合会非常大，因此shuffle阶段的效率要低于Map-side Join。
- reduce side join是一种最简单的join方式，其主要思想如下：
在map阶段，map函数同时读取两个文件File1和File2，为了区分两种来源的key/value数据对，对每条数据打一个标签（tag）

### semi join 
- 小表对大表,是reudce join的变种 map阶段过滤掉不需要join的字段 相当于Hivw SQL加的where过滤
#### 特点
- left semi join 的限制是， JOIN 子句中右边的表只能在 ON 子句中设置过滤条件，在 WHERE 子句、SELECT 子句或其他地方过滤都不行。
- left semi join 是只传递表的 join key 给 map 阶段，因此left semi join 中最后 select 的结果只许出现左表。
- 因为 left semi join 是 in(keySet)的关系，遇到右表重复记录，左表会跳过，而join则会一直遍历。这就导致右表有重复值得情况下 left semi join 只产生一条，join 会产生多条，也会导致 left semi join 的性能更高。
- ![image-20201206214502620](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:45:03-image-20201206214502620.png)


### SMB Join（sort merge bucket）
- SMB 存在的目的主要是为了解决大表与大表间的 Join 问题，分桶其实就是把大表化成了“小表”，然后 Map-Side Join 解决之，这是典型的分而治之的思想。
```
set hive.enforce.bucketing=true;
set hive.enforce.sorting=true;
```
表优化数据目标：相同数据尽量聚集在一起

## From
