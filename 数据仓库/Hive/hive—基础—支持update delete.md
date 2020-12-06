如果一个表要实现update和delete功能，该表就必须支持ACID，而支持ACID，就必须满足以下条件：

1、表的存储格式必须是ORC（STORED AS ORC）；

2、表必须进行分桶（CLUSTERED BY (col_name, col_name, ...)  INTO num_buckets BUCKETS）；

3、Table property中参数transactional必须设定为True（tblproperties('transactional'='true')）；

4、以下配置项必须被设定：

Client端：
```
hive.support.concurrency–true
hive.enforce.bucketing–true
hive.exec.dynamic.partition.mode–nonstrict
hive.txn.manager–org.apache.hadoop.hive.ql.lockmgr.DbTxnManager
```
服务端：
```
hive.compactor.initiator.on–true
hive.compactor.worker.threads–1
hive.txn.manager–org.apache.hadoop.hive.ql.lockmgr.DbTxnManager（经过测试，服务端也需要设定该配置项）
```