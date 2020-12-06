### 全局排序 （order by）
- 降序：desc
- 升序：asc 不需要指定，默认是升序

### 区内排序 （sort by ）分区字段 (distribute by)
- 不是全局排序，其在数据进入reducer前完成排序。因此，如果用sort by进行排序，并且设置mapreduce.job.reduces>1，则sort by只保证每个reducer的输出有序，不保证全局有序。

### 3、distribute by
- 类似于MapReduce中分区partation，对数据进行分区，结合sort by进行**使用 distribute by控制在map端如何拆分数据给reduce端**。
- hive会根据distribute by后面列，对应reduce的个数进行分发，默认是采用hash算法。

### 当分区条件和排序条件相同使用（cluster by）
- Cluster by 除了具有distribute by的功能外，还会对该字段进行排序。当distribute by和sort by 字段相同时，可以使用cluster by 代替