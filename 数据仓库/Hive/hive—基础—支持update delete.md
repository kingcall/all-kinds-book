如果一个表要实现update和delete功能，该表就必须支持ACID，而支持ACID，就必须满足以下条件：

1、表的存储格式必须是ORC（STORED AS ORC）；

2、表必须进行分桶（CLUSTERED BY (col_name, col_name, ...)  INTO num_buckets BUCKETS）；

3、Table property中参数transactional必须设定为True（tblproperties('transactional'='true')）；

4、以下配置项必须被设定：

Client端：
```sql
set hive.support.concurrency=true
set hive.enforce.bucketing=true
set hive.exec.dynamic.partition.mode=nonstrict
set hive.txn.manager=org.apache.hadoop.hive.ql.lockmgr.DbTxnManager
```
服务端：
```sql
set hive.compactor.initiator.on=true
set hive.compactor.worker.threads=1
set hive.txn.manager=org.apache.hadoop.hive.ql.lockmgr.DbTxnManager（经过测试，服务端也需要设定该配置项）
```