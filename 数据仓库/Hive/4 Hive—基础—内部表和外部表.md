[TOC]

## 内部表和外部表

**未被external修饰的是内部表（managed table），被external修饰的为外部表（external table）；因为默认情况下我们不加external关键字修饰，所以默认情况下是内部表**

区别：

- 内部表数据由Hive自身管理，外部表数据由HDFS管理；
- 内部表数据存储的位置是hive.metastore.warehouse.dir（默认：/user/hive/warehouse），外部表数据的存储位置由自己制定（如果没有LOCATION，Hive将在HDFS上的/user/hive/warehouse文件夹下以外部表的表名创建一个文件夹，并将属于这个表的数据存放在这里）；
- 删除内部表会直接删除元数据（metadata）及存储数据；删除外部表仅仅会删除元数据，HDFS上的文件并不会被删除；
- 对内部表的修改会将修改直接同步给元数据，而对外部表的表结构和分区进行修改，则需要修复（MSCK REPAIR TABLE table_name;）

### 创建内部表

```sql
CREATE TABLE ods.u_data (
  userid INT,
  movieid INT,
  rating INT,
  unixtime STRING)
 partitioned by(year string,month string ,day string) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/ml-100k/u.data' OVERWRITE INTO TABLE ods.u_data partition(year='2020',month='2020-12',day='2020-12-21');
```

当你建表成功并且把数据load 进去之后，你发现数据就在数仓的默认位置下

![image-20201223180105432](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/18:01:05-image-20201223180105432.png)

当你将表删除之后数据就不见了，你可以刷新一下页面

![image-20201223180204069](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/18:02:04-image-20201223180204069.png)

### 创建外部表

**不指定位置**

```
CREATE external TABLE ods.u_data (
  userid INT,
  movieid INT,
  rating INT,
  unixtime STRING)
 partitioned by(year string,month string ,day string) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/ml-100k/u.data' OVERWRITE INTO TABLE ods.u_data partition(year='2020',month='2020-12',day='2020-12-21');
```

![image-20201223180322641](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/18:03:23-image-20201223180322641.png)

删除表再看看`drop table ods.u_data;` 发现数据还在

![image-20201223180402203](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/18:04:02-image-20201223180402203.png)



**指定位置**

建表之前，我们先看一下，因为我打算在跟目录下创建一个外部表

![image-20201223180822636](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/18:08:23-image-20201223180822636.png)

创建表

```
CREATE external TABLE ods.u_data (
  userid INT,
  movieid INT,
  rating INT,
  unixtime STRING)
 partitioned by(year string,month string ,day string) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/external'; 
```

创建表完成后，我们发现多了一个文件夹，就是我们指定的位置，但是数仓目录下却没有以这个表名称命名的文件夹了，因为它是外部表，数据存储在指定的文件路径下，不需要创建默认的文件夹了

![image-20201223180946506](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/18:09:47-image-20201223180946506.png)

## 总结

1. hive 中的表大概可以分为这么几种，Table内部表   External Table 外部表  Partition分区表   Bucket Table 桶表(没有划分标准)
2. 外部表一般适用一些注重数据安全的场景或者数据已经存在特定位置的的场景下

