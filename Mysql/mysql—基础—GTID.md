# GTID(Global Transaction ID)

## 背景
- 从MySQL 5.6.5 开始新增了一种基于 GTID 的复制方式。通过 GTID 保证了每个在主库上提交的事务在集群中有一个唯一的ID。这种方式强化了数据库的主备一致性，故障恢复以及容错能力。
- GTID (Global Transaction ID)是全局事务ID,当在主库上提交事务或者被从库应用时，可以定位和追踪每一个事务，对DBA来说意义就很大了，我们可以适当的解放出来，不用手工去可以找偏移量的值了，而是通过CHANGE MASTER TO MASTER_HOST='xxx', MASTER_AUTO_POSITION=1的即可方便的搭建从库，在故障修复中也可以采用MASTER_AUTO_POSITION=‘X’的方式。
- GTID 和事务会记录到 binlog 中，用来标识事务。
GTID 是用来替代以前 classic 复制方法，MySQL-5.6.2 开始支持 GTID，在 MySQL-5.6.10 后完善。

## GTID 组成
- GTID 是由 server_uuid:Sequence_Number 。
    - Server_Uuid：是一个MySQL实例的全局唯一标识；存放为在$datadir/auto.cnf
    - Sequence_Number：是 MySQL 内部的一个事务的编号，一个 MySQL 实例不会重复的序列号（保证服务器内唯一），也表示在该实例上已经提交事务的数量，并且随着事务提交而递增。
- 根据 GTID 可以知道事务最初是在哪个实例上提交的，方便故障排查和切换
```
cat /data/mysql/data/auto.cnf 
[auto]
server-uuid=b3f31135-4851-11e8-b758-000c29148b03
```