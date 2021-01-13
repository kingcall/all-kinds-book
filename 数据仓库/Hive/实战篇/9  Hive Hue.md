**Apache Hive Hue**

(Hue工具)

Apache Hive  Hue

**HUE简介**

⊙**Hue**是一个开源的Apache Hadoop UI系统，由Cloudera Desktop演化而来，最后Cloudera公司将其贡献给Apache基金会的Hadoop社区，它是基于Python Web框架Django实现的。通过使用Hue我们可以在浏览器端的Web控制台上与Hadoop集群进行交互来分析处理数据，例如操作HDFS上的数据，运行MapReduce Job，执行Hive的SQL语句，浏览[HBase](https://cloud.tencent.com/product/hbase?from=10680)数据库等等。

⊙Hue在**数据库**方面，默认使用的是SQLite数据库来管理自身的数据，包括用户认证和授权，另外，可以自定义为[MySQL](https://cloud.tencent.com/product/cdb?from=10680)数据库、Postgresql数据库、以及Oracle数据库。其自身的功能包含有：

⊙对HDFS的访问，通过浏览器来查阅HDFS的数据。

⊙Hive编辑器：可以**编写HQL和运行HQL脚本**，以及查看运行结果等相关Hive功能。

⊙提供Solr搜索应用，并对应相应的可视化数据视图以及DashBoard。

⊙提供Impala的应用进行**数据交互**查询。

⊙最新的版本集成了***Spark编辑器和DashBoard***

⊙支持Pig编辑器，并能够运行编写的脚本任务。

⊙Oozie调度器，可以通过DashBoard来提交和**监控**Workflow、Coordinator以及Bundle。

⊙支持HBase对数据的查询修改以及可视化。

⊙支持对Metastore的浏览，可以访问***Hive的元数据***以及对应的HCatalog。

⊙另外，还有对Job的支持，Sqoop，ZooKeeper以及DB（MySQL，SQLite，Oracle等）的支持。

Hue In Hadoop

![img](https://ask.qcloudimg.com/http-save/yehe-6511370/d58mfpqzb0.jpeg?imageView2/2/w/1620)

**HUE  Job**

![img](https://ask.qcloudimg.com/http-save/yehe-6511370/wddklhgiun.jpeg?imageView2/2/w/1620)

**查询 （非常方便 自动提示）**

![img](https://ask.qcloudimg.com/http-save/yehe-6511370/vwj9ye9xfo.gif)

**查看表结构详情**

![img](https://ask.qcloudimg.com/http-save/yehe-6511370/551izzevt9.gif)

![img](https://ask.qcloudimg.com/http-save/yehe-6511370/0eva5pf9ke.gif)

***更改Hue任务名称***

![img](https://ask.qcloudimg.com/http-save/yehe-6511370/9b7td1czok.gif)

**场景：**

   通常呢，在我们工作的时候，如果你不知道怎么更改你的任务名，那么就会出现一个很尴尬的情况。 就是再job界面，大家都可以看到是你的任务再跑（**PS**:工作情况，一般的库名都是以自己名字命名。**EG: A2Data ）**

**解决方案：**

​    我们可以通过参数来设置名称，使其不再是看上去那么 ‘惨淡直白’！！

​    一般设置成你的需求名称最佳。

***例如这样：***

![img](https://ask.qcloudimg.com/http-save/yehe-6511370/v2ayus93km.jpeg?imageView2/2/w/1620)

**更改任务名**

```javascript
更改HUE任务名以及参数设置：
set mapred.job.name=A2Data;
```

![img](https://ask.qcloudimg.com/http-save/yehe-6511370/0ngbeefypp.jpeg?imageView2/2/w/1620)