最近发现离线任务对一个增量`Hive`表的查询越来越慢，这引起了我的注意，我在`cmd`窗口手动执行`count`操作查询发现，速度确实很慢，才不到五千万的数据，居然需要`300s`，这显然是有问题的，我推测可能是有小文件。

我去`hdfs`目录查看了一下该目录：

![微信截图_20210919161606](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2021/09/20/10:53:20-%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20210919161606.png)

发现确实有很多小文件，有480个小文件，我觉得我找到了问题所在，那么合并一下小文件吧：

```sql
insert into test select * from table distribute by floor (rand()*5);
```

这里使用`distribute by`进行了一个小文件的合并，通过rand() * 5，保证了从map端输出的数据，最多到5个reducer，将小文件数量控制了下来，现在只有3个文件了。

![微信截图_20210919172744](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2021/09/20/10:59:22-%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20210919172744.png)

合并小文件后，再次做同样的查询，`15s`就完成了。确实忽略了，增量数据会导致小文件，应该在当初做的时候就做定时的小文件合并，而不是等到现在才发现。

因为这个表每天是有增量数据进去的，增量数据会单独生成一个文件，因为增量数据本身不大，日积月累就形成了大量小文件。不仅对namenode的内存造成压力，对map端的小文件合并也有很大压力。



## 小文件产生的原因

- 动态分区插入数据的时候，会产生大量的小文件；

- 数据源本身就包含有大量的小文件；
- 做增量导入，比如Sqoop数据导入，一些增量insert等；
- 分桶表，分桶表通常也会遇到小文件，本质上还是增量导入的问题；
- 可以修改的表，这种Hive表是可以进行修改的，通过配置`stored as orc TBLPROPERTIES ("transactional"="true")`，这种表最坑，每天都会有一个快照，到后面10G大小的数据，表文件体积可以达到600G，时间越长越大；

小文件的问题有很多，实际中各种原因，由于自己的不小心，前期没有做好预防都会产生大量小文件，让线上的离线任务神不知鬼不觉，越跑越慢。



## 小文件的危害

1. 给namenode内存中fsImage的合并造成压力，如果namenode内存使用完了，这个集群将不能再存储文件了；
2. 虽然map阶段都设置了小文件合并，`org.apache.hadoop.hive.ql.io.CombineHiveInputFormat`，太多小文件导致合并时间较长，查询缓慢；

## 小文件的解决方案

彻底解决小文件，分为了两个方向，**一个是小文件的预防，一个是大量小文件问题已经出现了**，我们该怎么解决。

#### 1. 小文件的预防

网上有些解决方案，是调节参数，这些参数在我使用的`Hive2`是默认都开启了的：

```bash
//每个Map最大输入大小(这个值决定了合并后文件的数量)
set mapred.max.split.size=256000000;  
//一个节点上split的至少的大小(这个值决定了多个DataNode上的文件是否需要合并)
set mapred.min.split.size.per.node=100000000;
//一个交换机下split的至少的大小(这个值决定了多个交换机上的文件是否需要合并)  
set mapred.min.split.size.per.rack=100000000;
//执行Map前进行小文件合并
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; 
//设置map端输出进行合并，默认为true
set hive.merge.mapfiles = true
//设置reduce端输出进行合并，默认为false
set hive.merge.mapredfiles = true
//设置合并文件的大小
set hive.merge.size.per.task = 256*1000*1000
//当输出文件的平均大小小于该值时，启动一个独立的MapReduce任务进行文件merge。
set hive.merge.smallfiles.avgsize=16000000
```

有些公司用的版本不同，低版本可能有些配置不一样，最好检查一下上面这些配置是否设置，然后根据自己的实际集群情况进行设置。

小文件的预防，主要还是要根据小文件的产生原因，来进行预防。

1. 动态分区插入的时候，保证有静态分区，不要误判导致产生大量分区，大量分区加起来，自然就有大量小文件；
2. 如果源表是有大量小文件的，在导入数据到目标表的时候，如果只是`insert into dis select * from origin`的话，目标表通常也有很多小文件。如果有分区，比如`dt, hour`，可以使用`distribute by dt, hour`，保证每个小时的数据在一个reduce里面；
3. 类似`sqoop`增量导入，还有`hive`一些表的查询增量导入，这些肯定是有小文件的，需要进行**一周甚至一天**定时任务的小文件合并。

#### 2. 小文件的解决

上面是平时开发数据任务时候，小文件的预防，但如果由于我们的大意，小文件问题已经产生了，就需要解决了。通常就是`insert overwrite`了。

```sql
insert overwrite table test [partition(hour=...)] select * from test distribute by floor (rand()*5);
```

注：这个语句把`test`表的数据查询出来，`overwrite`覆盖`test`表，不用担心如果`overwrite`失败，数据没了，这里面是有事物性保证的，可以观察一下执行的时候，在test表`hdfs`文件目录下面有个临时文件夹。如果是分区表，加上`partition`，表示对该分区进行`overwrite`。

如果是orc格式存储的表，还可以使用`alter table test [partition(...)] concatenate`进行小文件的合并，不过这种方法仅仅适用于orc格式存储的表。