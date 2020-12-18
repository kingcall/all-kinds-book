前面遇到过一次因为`Hive`中表被锁住了，导致定时任务一直失败。这两天又出现了表被锁，原因是连接`hiveserver2`过于频繁，`mysql`连接被打满，引发的连锁反应，导致我们的小时任务一直失败，下午重点注意到这个问题，才解决好。

## Hive中的锁
在执行`insert into`或`insert overwrite`任务时，中途手动将程序停掉，会出现卡死情况（无法提交MapReduce），只能执行查询操作，而`drop insert`操作均不可操作，无论执行多久，都会保持卡死状态。

查看`Hive`的中死锁，可以使用`show locks [table]`来查看。

![clipboard](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/07/21:15:41-clipboard.png)

可以看到里面的那个Type下的EXCLUSIVE，这是一种互斥锁，需要解决，否则后续的查询和插入任务都会影响。

hive存在两种锁，共享锁`Shared (S)`和互斥锁`Exclusive (X）`

| 锁   | S    | X    |
| ---- | ---- | ---- |
| S    | 是   | 否   |
| X    | 否   | 否   |

锁的基本机制是：

- 元信息和数据的变更需要互斥锁

- 数据的读取需要共享锁

触发共享锁的操作是可以并发执行的，但是触发互斥锁，那么该表和该分区就不能并发的执行作业了。

![微信截图_20201207211428](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/07/21:15:25-%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20201207211428.png)

对于上面的情况，使用解锁命令：

```sql
unlock table tableName
```

**注意**：表锁和分区锁是两个不同的锁，对表解锁，对分区是无效的，分区需要单独解锁

## 解锁方法

查看表被锁的情况：

```sql
show locks tableName
```

常规解锁方法：

```sql
unlock table 表名;  -- 解锁表
unlock table 表名 partition(dt='2014-04-01');  -- 解锁某个分区
```

高版本hive默认插入数据时，不能查询，因为有锁

### 可能出现的问题

解锁之路通常不是一帆风顺的，可能会遇到各种问题，笔者是在`Hive2.1.1`下面测试，比如：

![clipboard3](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/07/21:21:33-clipboard3.png)

这个命令无法执行，说`LockManager`没有指定，这时候需要执行命令：

```sql
set hive.support.concurrency=true;
set hive.txn.manager = org.apache.hadoop.hive.ql.lockmgr.DummyTxnManager;
```

这样重新执行，命令就可以执行了

如果还！是！不！行，终极方法，可以直接去mysql元数据执行：

```sql
select * from HIVE_LOCKS;
```

查到所有的锁，然后根据条件把对应的锁删掉，这个锁住的表即可释放出来了。

```sql
delete from HIVE_LOCKS where HL_DB = 'cdn' and HL_TABLE = 'cdn_log';
```

**注意**：表名和字段都需要大写。

通过这种办法，通常可以彻底解决锁的问题。