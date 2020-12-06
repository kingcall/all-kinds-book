[toc]
## 动态分区调整
- 动态分区属性：设置为true表示开启动态分区功能（默认为false）hive.exec.dynamic.partition=true;
- 动态分区属性：设置为nonstrict,表示允许所有分区都是动态的（默认为strict）设置为strict，表示必须保证至少有一个分区是静态的hive.exec.dynamic.partition.mode=strict;
- 动态分区属性：每个mapper或reducer可以创建的最大动态分区个数hive.exec.max.dynamic.partitions.pernode=100;
- 动态分区属性：一个动态分区创建语句可以创建的最大动态分区个数hive.exec.max.dynamic.partitions=1000;
- 动态分区属性：全局可以创建的最大文件个数hive.exec.max.created.files=100000;
- 控制DataNode一次可以打开的文件个数 这个参数必须设置在DataNode的$HADOOP_HOME/conf/hdfs-site.xml文件中
```
<property>
    <name>dfs.datanode.max.xcievers</name>
    <value>8192</value>
</property>
```
### 注意
> 在Hive中，动态分区会造成在插入数据过程中，生成过多零碎的小文件