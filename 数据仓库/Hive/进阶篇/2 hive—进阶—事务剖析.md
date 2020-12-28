[TOC]

## 事务

上一节我们学习了如何让Hive 支持 insert values(...),delete和update,以及Hive 事务的配置和实现的基本原理,接下来我们具体看一下它的实现

### 1 安装工具

开始之前我们先去安装一个ORC 文件的工具，去git 上将项目clone 下来`https://github.com/apache/orc` ，然后进行项目下的Java 目录，直接打包` mvn clean package -DskipTests=true -Dmaven.javadoc.skip=true`，

![image-20201228115325010](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/11:53:26-image-20201228115325010.png)

最后打包得到的`orc-tools-1.7.0-SNAPSHOT-uber.jar` 就是我们的工具,它就在tools 目录下

![image-20201228115400186](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/11:54:00-image-20201228115400186.png)

接下来我们就可以使用它了`java -jar orc-tools-X.Y.Z-uber.jar <sub-command> <args>` 

![image-20201228115952707](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/11:59:53-image-20201228115952707.png)

这下我们就可以使用它查询ORC 文件的内容了,为了以后使用方面我们可以将其包装起来成为一个脚本

```
#/bin/sh
command=$1
args=$2
java -jar orc-tools-1.7.0-SNAPSHOT-uber.jar $command $args
```

然后给其创建别名方便使用,你可以将其写到profile 文件中，以便永久生效

```
alias orc=/Users/liuwenqiang/workspace/code/script/orc_tools.sh
```

接下来我们就使用一下吧

![image-20201228121730263](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/12:17:30-image-20201228121730263.png)



### 2. 事务信息解读

前面我们讲了所有 INSERT 语句都会创建 `delta` 目录。UPDATE 语句也会创建 `delta` 目录，但会先创建一个 `delete` 目录，即先删除、后插入。`delete` 目录的前缀是 delete_delta； 既然如此那我们的Hive 是怎么在读取数据的时候返回给我们正确的数据呢

前面气门说了ORC 文件中的每一行数据都会以 `rowId` 作为标识。从 ACID 事务表中读取数据就是对这些文件进行合并，从而得到最新事务的结果。这一过程是在 `OrcInputFormat` 和 `OrcRawRecordMerger` 类中实现的，本质上是一个合并排序的算法。第一个输出是我们第一次插入的一条数据，它的`rowId` 是1，后面我们又插入3条同样的数据，第二个输出是我们对Hive 表中的数据做了一次Update 操作

![image-20201228132535808](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/13:25:36-image-20201228132535808.png)

这个信息你也可以通过SQL 的方式进行查询

![image-20201228132753759](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/13:27:54-image-20201228132753759.png)

接下来我们就解读一下这里的一些信息，方便我们后面理解



Hive 会为所有的事务生成一个全局唯一的 ID，包括读操作和写操作。针对写事务（INSERT、DELETE 等），Hive 还会创建一个写事务 ID（Write ID），该 ID 在表范围内唯一。写事务 ID 会编码到 `delta` 和 `delete` 目录的名称中；语句 ID（Statement ID）则是当一个事务中有多条写入语句时使用的，用作唯一标识。整个文件夹的采取的命名格式 `delta_minWID_maxWID_stmtID`，即 delta 前缀、写事务的 ID 范围、以及语句 ID。删除的时候文件件的前缀是` delete_delta` 里面包含了要删除的文件。

`_orc_acid_version` 的内容是 2，即当前 ACID 版本号是 2。它和版本 1 的主要区别是 UPDATE 语句采用了 split-update 特性，即上文提到的先删除、后插入。这个特性能够使 ACID 表支持条件下推等功能，具体可以查看 [HIVE-14035](https://jira.apache.org/jira/browse/HIVE-14035)，这个文件不是ORC 文件，所以你可以下载下来直接查看

`bucket_00000` 文件则是写入的数据内容。由于这张表没有分区和分桶，所以只有这一个文件。事务表都以 [ORC](https://orc.apache.org/) 格式存储的，

文件中的数据会按 (`originalTransaction`, `bucket`, `rowId`) 进行排序，这点对后面的读取操作非常关键



#### operation

 0 表示插入，1 表示更新，2 表示删除。由于使用了 split-update，UPDATE 是不会出现的，所以delat 文件中的operation 是0 ， delete_delta 文件中的operation 是2

![image-20201228133649565](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/13:36:50-image-20201228133649565.png)

#### rowId

rowId是一个自增的唯一 ID，在写事务和分桶的组合中唯一

![image-20201228143906662](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/14:39:07-image-20201228143906662.png)

####  originalTransaction

 是该条记录的原始写事务 ID，对于 INSERT 操作，该值和 `currentTransaction` 是一致的。对于 DELETE，则是该条记录第一次插入时的写事务 ID；对于Update 操作则和`currentTransaction`也 是一致的，因为Update 就是delete+insert 实现的

**update**

![image-20201228134749984](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/13:47:50-image-20201228134749984.png)

**Delete**

![image-20201228134813996](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/13:48:14-image-20201228134813996.png)

#### currentTransaction

当前的写事务 ID

#### row

具体数据。对于 DELETE 语句，则为 `null`，对于Insert 就是插入的数据，对于Update 就是更新后的数据

### 3. 数据读取过程

ORC 文件中的每一行数据都会以 `row__id` 作为标识并排序。从 ACID 事务表中读取数据就是对这些文件进行合并，从而得到最新事务的结果。这一过程是在 `OrcInputFormat` 和 `OrcRawRecordMerger` 类中实现的，本质上是一个合并排序的算法。

合并过程为：对所有数据行按照 (originalTransaction, bucketId, rowId) 正序排列，(currentTransaction) 倒序排列

以下列出的文件为例，产生这些文件的操作为：插入三条记录，进行一次 Major Compaction，然后更新两条记录。1-0-0-1 是对 originalTransaction - bucketId - rowId - currentTransaction 的缩写。

```javascript
+----------+    +----------+    +----------+
| base_1   |    | delete_2 |    | delta_2  |
+----------+    +----------+    +----------+
| 1-0-0-1  |    | 1-0-1-2  |    | 2-0-0-2  |
| 1-0-1-1  |    | 1-0-2-2  |    | 2-0-1-2  |
| 1-0-2-1  |    +----------+    +----------+
+----------+
```

合并过程为：

- 对所有数据行按照 (originalTransaction, bucketId, rowId) 正序排列，(currentTransaction) 倒序排列，即：
  - ​    1-0-0-1
  - ​    1-0-1-2
  - ​    1-0-1-1
  - ​    …
  - ​    2-0-1-2
- 获取第一条记录；
- 如果当前记录的 row__id 和上条数据一样，则跳过；
- 如果当前记录的操作类型为 DELETE，也跳过；
  - 通过以上两条规则，对于 1-0-1-2 和 1-0-1-1，这条记录会被跳过；
- 如果没有跳过，记录将被输出给下游；
- 重复以上过程。

合并过程是流式的，即 Hive 会将所有文件打开，预读第一条记录，并将 row__id 信息存入到 ReaderKey 类型中。该类型实现了 Comparable 接口，因此可以按照上述规则自定义排序：

```java
public class RecordIdentifier implements WritableComparable<RecordIdentifier> {
  private long writeId;
  private int bucketId;
  private long rowId;
  protected int compareToInternal(RecordIdentifier other) {
    if (other == null) { return -1; }
    if (writeId != other.writeId) { return writeId < other.writeId ? -1 : 1; }
    if (bucketId != other.bucketId) { return bucketId < other.bucketId ? - 1 : 1; }
    if (rowId != other.rowId) { return rowId < other.rowId ? -1 : 1; }
    return 0;
  }
}

public class ReaderKey extends RecordIdentifier {
  private long currentWriteId;
  private boolean isDeleteEvent = false;
  public int compareTo(RecordIdentifier other) {
    int sup = compareToInternal(other);
    if (sup == 0) {
      if (other.getClass() == ReaderKey.class) {
        ReaderKey oth = (ReaderKey) other;
        if (currentWriteId != oth.currentWriteId) { return currentWriteId < oth.currentWriteId ? +1 : -1; }
        if (isDeleteEvent != oth.isDeleteEvent) { return isDeleteEvent ? -1 : +1; }
      } else {
        return -1;
      }
    }
    return sup;
  }
}
```

然后，`ReaderKey` 会和文件句柄一起存入到 `TreeMap` 结构中。根据该结构的特性，我们每次获取第一个元素时就能得到排序后的结果，并读取数据了。

```java
public class OrcRawRecordMerger {
  private TreeMap<ReaderKey, ReaderPair> readers = new TreeMap<>();
  public boolean next(RecordIdentifier recordIdentifier, OrcStruct prev) {
    Map.Entry<ReaderKey, ReaderPair> entry = readers.pollFirstEntry();
  }
}
```

#### 选择文件

上文中提到，事务表目录中会同时存在多个事务的快照文件，因此 Hive 首先要选择出反映了最新事务结果的文件集合，然后再进行合并。举例来说，下列文件是一系列操作后的结果：两次插入，一次 Minor Compaction，一次 Major Compaction，一次删除。

过滤过程为：

- 从 Hive Metastore 中获取所有成功提交的写事务 ID 列表；
- 从文件名中解析出文件类型、写事务 ID 范围、以及语句 ID；
- 选取写事务 ID 最大且合法的那个 base 目录，如果存在的话；
- 对 delta 和 delete 文件进行排序：
  - minWID 较小的优先；
  - 如果 minWID 相等，则 maxWID 较大的优先；
  - 如果都相等，则按 stmtID 排序；没有 stmtID 的会排在前面；
- 将 base 文件中的写事务 ID 作为当前 ID，循环过滤所有 delta 文件：
  - 如果 maxWID 大于当前 ID，则保留这个文件，并以此更新当前 ID；
  - 如果 ID 范围相同，也会保留这个文件；
  - 重复上述步骤。

## 总结

1. Hive 事务的实现原理主要就是利用增量文件进行记录当前操作，然后合并增量文件和原始文件产生最新的结果
2. 关于具体的代码分析我们会在Hive 的源码篇中单独介绍

