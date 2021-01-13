# Presto连接Hive

## 配置 Hive Connector

- etc/catalog/hive.properties

```javascript
connector.name=hive-hadoop2
hive.metastore.uri=thrift://<hive_metastore_ip>:9083
hive.config.resources=/opt/presto-server-0.211/etc/cluster/core-site.xml,/opt/presto-server-0.211/etc/cluster/hdfs-site.xml
```

其中 hive.metastore.uri 可以从 hive-site.xml 文件中获取。

将配置复制到其它节点的相同目录下。

## hdfs 配置文件

从 hdfs 的环境中复制 core-site.xml 和 hdfs-site.xml 文件到 presto 的 etc/cluster 目录下。

将配置复制到其它节点的相同目录下。

## 启动 Prestore

分别在两个节点上重新启动 Presto 服务。

## 在 Hive 中创建数据库、数据表和数据

```javascript
$ hive

# 创建数据库
hive> create schema test; 

# 查询数据库
hive> show databases;
+---------------------+
|    database_name    |
+---------------------+
| default             |
| information_schema  |
| sys                 |
| test                |
+---------------------+

# 显示数据库中的表
hive> show tables from test;
+-----------+
| tab_name  |
+-----------+
+-----------+

# 创建数据表
hive> CREATE TABLE test.users(id int, username string, password string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
...

# 插入数据
hive> insert into table test.users values (1, 'user1', 'password1'), (2, 'user2', 'password2'), (3, 'user3', 'password3');
...

# 查询数据
hive> select * from test.users;
+-----------+-----------------+-----------------+
| users.id  | users.username  | users.password  |
+-----------+-----------------+-----------------+
| 1         | user1           | password1       |
| 2         | user2           | password2       |
| 3         | user3           | password3       |
+-----------+-----------------+-----------------+
```

## 通过 Presto 查询数据

```javascript
# 启动 presto 命令行
$ ./presto --server bd1:8080 --catalog hive --schema test

# 查询数据库
presto:test> show schemas from hive;
       Schema       
--------------------
 default            
 information_schema 
 sys                
 test               
(4 rows)

# 查询数据表
presto:test> show tables from hive.test;
 Table  
--------
 users  
 users2 
(2 rows)

# 查询数据
presto:test> select * from hive.test.users;
 id | username | password 
----+----------+----------
(0 rows)
```