[TOC]

## 数据仓库教程

这里我们做了一份关于整个数仓体系的教程，主要内容体系大家可以看目录，后续满面会完善，欢迎大家关注，一起讨论交流，这份教程的特点有四个

1. 知识体系完善，从数仓的概念、建模、数仓工具的使用、数仓的落地实践都有，还会有很多小案例
2. 使用的技术都是当前企业最常用的技术，版本也是比较新的，不会导致大家看到一个代码，然后一执行发现语法不支持或者废弃掉了
3. 后续的更新也会很及时，不会说更新了一段时间断更了，在更新完大纲之后，我也会不断完善该系列，不断添加新的知识点
4. 创作团队都是在企业一线员工，实战多于理论。

### 1. 数仓建模

​	[什么是数仓]()

​	[数仓架构发展史](https://blog.csdn.net/king14bhhb/article/details/110715863)

​	[数仓建模方法论](https://blog.csdn.net/king14bhhb/article/details/110894422)

​	[分层建设理论](https://blog.csdn.net/king14bhhb/article/details/110714959)

​	[数仓治理]()

​	[数据湖初识](https://blog.csdn.net/king14bhhb/article/details/111392617)

   [指标体系建设](https://blog.csdn.net/king14bhhb/article/details/110941854)

### 2. 数仓工具

#### 1. Hive

##### 1.Hive 基础篇

​	[1. 什么是Hive](https://blog.csdn.net/king14bhhb/article/details/111462896)

​	[2. Hive的编译安装](https://blog.csdn.net/king14bhhb/article/details/111568313)

​	[3. Hive表的基础操作](https://blog.csdn.net/king14bhhb/article/details/111584254)

​	[4. Hive数据的组织管理方式](https://blog.csdn.net/king14bhhb/article/details/111592592)

​	[5. Hive内部表和外部表](https://blog.csdn.net/king14bhhb/article/details/111595879)

​	[6. Hive动态分区](https://blog.csdn.net/king14bhhb/article/details/111598399)

​	[7. Hive命令行](https://blog.csdn.net/king14bhhb/article/details/111600665)

​	[8. Hive基本数据类型](https://blog.csdn.net/king14bhhb/article/details/111657942)

​	[9. Hive复合数据类型](https://blog.csdn.net/king14bhhb/article/details/111712993)

​	[10. Hive Streaming](https://blog.csdn.net/king14bhhb/article/details/111729038)

​	[11. Hive关键字](https://blog.csdn.net/king14bhhb/article/details/111735030)

​	[12. Hive函数大全](https://blog.csdn.net/king14bhhb/article/details/111765573)

​	[13. Hive的架构设计](https://blog.csdn.net/king14bhhb/article/details/111769279)

​	[14. Hive架构之HiveServer2](https://blog.csdn.net/king14bhhb/article/details/111770337)

​	[15. Hive的其他语言调用](https://blog.csdn.net/king14bhhb/article/details/111770561)

​	[16. Hive架构服务](https://blog.csdn.net/king14bhhb/article/details/111777544)

​	[17. Hive的严格模式和本地模式](https://blog.csdn.net/king14bhhb/article/details/111795036)

​	[18. Hive的执行引擎](https://blog.csdn.net/king14bhhb/article/details/111823588)

​	[19. Hive视图和物化视图](https://blog.csdn.net/king14bhhb/article/details/111827225)

​	[20. Hive UDF](https://blog.csdn.net/king14bhhb/article/details/111827549)

##### 2.Hive 语法篇

​	[1. Json 解析](https://blog.csdn.net/king14bhhb/article/details/111999201)

​	[2. like rlike regexp](https://blog.csdn.net/king14bhhb/article/details/112058139)

​	[3. explode 和 lateral view](https://blog.csdn.net/king14bhhb/article/details/112058141)

​	[4. with as和from](https://blog.csdn.net/king14bhhb/article/details/112058174)

​	[5. Order by, Sort by ,Dristribute by,Cluster By](https://blog.csdn.net/king14bhhb/article/details/112093373)

​	[6. grouping sets](https://blog.csdn.net/king14bhhb/article/details/112063657)

​	[7. cube和rollup](https://blog.csdn.net/king14bhhb/article/details/112069418)

​	[8. map join、reduce join、smb join](https://blog.csdn.net/king14bhhb/article/details/112132243)

​	[9. 窗口函数初识 max count sum](https://blog.csdn.net/king14bhhb/article/details/112172378)

​	[10. 窗口函数row_number、rank、dense_rank](https://blog.csdn.net/king14bhhb/article/details/112253118)

​	[11. 窗口函数ntile](https://blog.csdn.net/king14bhhb/article/details/112258298)

​	[12. 窗口函数first_value和last_value](https://blog.csdn.net/king14bhhb/article/details/112260539)

​	[13. 窗口函数lead和lag](https://blog.csdn.net/king14bhhb/article/details/112267910)

​	[14. 窗口函数cume_dist和 percent_rank](https://blog.csdn.net/king14bhhb/article/details/112283469)

​	[15. 窗口函数练习和总结](https://blog.csdn.net/king14bhhb/article/details/112390073)

​	[16. Hive语法之抽样](https://blog.csdn.net/king14bhhb/article/details/112528852)

##### 3.Hive 进阶篇

​	[1. Hive进阶之索引](https://blog.csdn.net/king14bhhb/article/details/111830230)

​	[2. Hive进阶之事务初识](https://blog.csdn.net/king14bhhb/article/details/111841190)

​	[3. Hive进阶之事务深度剖析](https://blog.csdn.net/king14bhhb/article/details/111866920)

​	[4. Hive进阶之执行计划](https://blog.csdn.net/king14bhhb/article/details/112391654)

​	[5. Hive进阶之数据存储格式](https://blog.csdn.net/king14bhhb/article/details/112520702)

​	[6. Hive进阶之数据压缩配置与格式](https://blog.csdn.net/king14bhhb/article/details/112520702)

​	[7. Hive进阶之SerDe](https://blog.csdn.net/king14bhhb/article/details/112541876)

​	[8 Hive进阶之权限管理](https://blog.csdn.net/king14bhhb/article/details/112576550)

​	[10. Hive优化指南](https://blog.csdn.net/king14bhhb/article/details/111244999)

##### 4.Hive 源码篇

##### 5. Hive 实战篇

​	[1. UDF分词](https://blog.csdn.net/king14bhhb/article/details/111875289)

​	[2. UDF IP 解析](https://blog.csdn.net/king14bhhb/article/details/111939061)

   [3. UDF SQL 解析](https://blog.csdn.net/king14bhhb/article/details/111939061)

#### 2. 高性能查询引擎

##### 1.Spark-SQL

##### 2.Impala

##### 3.Presto

##### 4.Druid

#### 3.数据同步工具

#### 4. 调度工具

##### 1. 调度工具的使用

##### 2. 调度工具整合

首先我们会创建常用的脚本然后配合配合数仓的SQL 进行数仓的整体的调度，脚本的话我们主要有抽数的脚本、执行SQL的脚本、监控的脚本、发布脚本

当然我们还会引入版本管理工具，管理我们的SQL和脚本，然后进行发布

#### 5. 元数据管理工具

#### 6. 监控工具

#### 7. 报表工具

### 3. 数仓实战

#### 1. K12赛道Top公司的数仓建设

#### 2. 知名游戏公司的数仓建设

#### 3. 如何设计企业级数据平台

#### 4. 企业级实时数仓建设案例



## 总结



