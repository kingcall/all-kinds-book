[toc]

## 常见的数据格式
### 背景
- 当今的数据处理大致可分为两大类，联机事务处理 OLTP(on-line transaction processing)联机分析处理 OLAP(On-Line Analytical Processing)
> OLTP 是传统关系型数据库的主要应用用来执行一些基本的、日常的事务处理比如数据库记录的增、删、改、查等等而OLAP则是分布式数据库的主要应用它对实时性要求不高，但处理的数据量大通常应用于复杂的动态报表系统上

![image-20201206214745123](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:47:45-image-20201206214745123.png)


#### 行存储的特点
- 查询满足条件的一整行数据的时，只需要找到其中一个值，其余的值都在相邻地方，所以此时行存储查询的速度更快。
- 传统的关系型数据库，如 Oracle、DB2、MySQL、SQL SERVER 等采用行式存储法(Row-based)，在基于行式存储的数据库中， **数据是按照行数据为基础逻辑存储单元进行存储的， 一行中的数据在存储介质中以连续存储形式存在**。
- - 行存储的特点：查询满足条件的一整行数据的时，只需要找到其中一个值，其余的值都在相邻地方，所以此时行存储查询的速度更快
- TEXTFILE和SEQUENCEFILE的存储格式都是基于行存储的；


#### 列存储的特点
- 因为每个字段的数据聚集存储，在查询只需要少数几个字段的时候，能大大减少读取的数据量；每个字段的数据类型一定是相同的，列式存储可以针对性的设计更好的设计压缩算法。TEXTFILE和SEQUENCEFILE的存储格式都是基于行存储的；ORC和PARQUET是基于列式存储的。
- 列式存储(Column-based)是相对于行式存储来说的，新兴的Hbase、HPVertica、EMCGreenplum等分布式数据库均采用列式存储。在基于列式存储的数据库中， **数据是按照列为基础逻辑存储单元进行存储的，一列中的数据在存储介质中以连续存储形式存在**。

![image-20201206214802357](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:48:02-image-20201206214802357.png)

- 列存储的特点：因为每个字段的数据聚集存储，在查询只需要少数几个字段的时候，能**大大减少读取的数据量**；每个字段的数据类型一定是相同的，列式存储可以针对性的设计更好的设计压缩算法。ORC和PARQUET是基于列式存储的。

### TextFile
- 默认格式，存储方式为行存储，数据不做压缩，磁盘开销大，数据解析开销大。 
- 可结合Gzip、Bzip2使用(系统自动检查，执行查询时自动解压)，但使用这种方式，压缩后的文件不支持split，Hive不会对数据进行切分，从而无法对数据进行并行操作。
- 并且在反序列化过程中，必须逐个字符判断是不是分隔符和行结束符，因此反序列化开销会比SequenceFile高几十倍。
```
create table log_text(track_time string, url string, session_id string, referer string, ip string, end_user_id string, city_id string)
row format delimited fields terminated by '\t'
stored as textfile;
```

### SequenceFile
- SequenceFile是Hadoop API提供的一种二进制文件支持，，存储方式为行存储，其具有使用方便、可分割、可压缩的特点。SequenceFile支持三种压缩选择：NONE，RECORD，BLOCK。Record压缩率低，一般建议使用BLOCK压缩。优势是文件和hadoop api中的MapFile是相互兼容的。

### RCfile：
存储方式：数据按行分块，每块按列存储。结合了行存储和列存储的优点：首先，RCFile 保证同一行的数据位于同一节点，因此元组重构的开销很低；其次，像列存储一样，RCFile 能够利用列维度的数据压缩，并且能跳过不必要的列读取；

### ORCfile
- 存储方式：**数据按行分块 每块按照列存储**(不是真正意义上的列存储，可以理解为分段列存储)。
> 假设一个表有10亿行数据，按照列式存储的定义，应该先将某个字段的10亿条数据存储完之后，再存储其他字段。
- 压缩快 快速列存取。效率比rcfile高,是rcfile的改良版本。
- 是以二进制方式存储的，所以是不可以直接读取，ORC文件也是自解析的，它包含许多的元数据，这些元数据都是同构ProtoBuffer进行序列化的。

```
create table log_orc(track_time string, url string, session_id string, referer string, ip string, end_user_id string, ciry_id string)
row format delimited fields terminated by '/t'
stored as orc;
```
- 默认创建的ORC存储方式，导入数据后的大小为:2.8 M  /user/hive/warehouse/log_orc/000000
比Snappy压缩的还小。原因是orc存储文件默认采用ZLIB压缩，ZLIB采用的是deflate压缩算法。比snappy压缩的小。

#### 格式
- 每个Orc文件由1个或多个stripe组成，**每个stripe一般为HDFS的块大小**，每一个stripe包含多条记录，**这些记录按照列进行独立存*储*对应到Parquet中的row group的概念。每个Stripe里有三部分组成，分别是Index Data，Row Data，Stripe Footer；
- Stripe对应到Parquet中的row group的概念。

![image-20201206214822743](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:48:23-image-20201206214822743.png)
- 每次读取文件是以行组为单位的，一般为HDFS的块大小，保存了每一列的索引和数据。


#### 数据访问
- **读取ORC文件是从尾部开始的**，第一次读取16KB的大小，尽可能的将Postscript和Footer数据都读入内存。文件的最后一个字节保存着PostScript的长度，它的长度不会超过256字节，PostScript中保存着整个文件的元数据信息，它包括文件的压缩格式、文件内部每一个压缩块的最大长度(每次分配内存的大小)、Footer长度，以及一些版本信息。在Postscript和Footer之间存储着整个文件的统计信息(上图中未画出)，这部分的统计信息包括每一个stripe中每一列的信息，主要统计成员数、最大值、最小值、是否有空值等


### Parquet
- Parquet文件是**以二进制方式存储的，所以是不可以直接读取的**，文件中包括该文件的数据和元数据，因此Parquet格式文件是自解析的。
```
create table log_parquet(track_time string, url string, session_id string, referer string, ipstring,end_user_idstring, city_id string)
row format delimited fields terminated by '/t'
stored as parquet;
```

### 自定义存储格式

There is not a single "Hive format" in which data must be stored. Hive comes with built in connectors for comma and tab-separated values (CSV/TSV) text files, [Apache Parquet](http://parquet.apache.org/)***\*[™](http://hadoop.apache.org/)\****, [Apache ORC](http://orc.apache.org/)***\*[™](http://hadoop.apache.org/)\****, and other formats. Users can extend Hive with connectors for other formats. Please see [File Formats](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-FileFormats) and [Hive SerDe](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe) in the [Developer Guide](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide) for details.

**我们可以带着下面问题来阅读文章****问题：****1.hive适合处理什么数据？****2.hive的数据，存在哪里？**初次使用 hive ，应该说上手还是挺快的。 Hive 提供的类 SQL 语句与 mysql 语句极为相似，语法上有大量相同的地方，这给我们上手带来了很大的方便，但是要得心应手地写好这些语句，还需要对 hive 有较好的了解，才能结合 hive 特色写出精妙的语句。关于 hive 语言的详细语法可参考官方 wiki 的语言手册：http://wiki.apache.org/hadoop/Hive/LanguageManual虽然语法风格为我们提供了便利，但初次使用遇到的问题还是不少的，下面针对业务场景谈谈我们遇到的问题，和对 hive 功能的定制。1、 分隔符问题首先遇到的是日志数据的分隔符问题，我们的日志数据的大致格式如下：2010-05-24 00:00:02@$_$@QQ2010@$_$@all@$_$@NOKIA_1681C@$_$@1@$_$@10@$_$@@$_$@-1@$_$@10@$_$@application@$_$@1从格式可见其分隔符是“ @$_$@ ”，这是为了尽可能防止日志正文出现与分隔符相同的字符而导致数据混淆。本来 hive支持在建表的时候指定自定义分隔符的，但经过多次测试发现只支持单个字符的自定义分隔符，像“ @$_$@ ”这样的分隔符是不能被支持的，但是我们可以通过对分隔符的定制解决这个问题， hive 的内部分隔符是“ \001 ”，只要把分隔符替换成“\001 ”即可。经过探索我们发现有两条途径解决这个问题。a)自定义 outputformat 和 inputformat 。Hive 的 outputformat/inputformat 与 hadoop 的 outputformat/inputformat 相当类似， inputformat 负责把输入数据进行格式化，然后提供给 hive ， outputformat 负责把 hive 输出的数据重新格式化成目标格式再输出到文件，这种对格式进行定制的方式较为底层，对其进行定制也相对简单，重写 InputFormat 中 RecordReader 类中的 next 方法即可，示例代码如下：`public boolean next(LongWritable key, BytesWritable value)      throws IOException {      while ( reader .next(key, text ) ) {      String strReplace = text .toString().toLowerCase().replace( "@$_$@" , "\001" );      Text txtReplace = new Text();      txtReplace.set(strReplace );      value.set(txtReplace.getBytes(), 0, txtReplace.getLength());      return true ;     }       return false ; }      重写 HiveIgnoreKeyTextOutputFormat 中 RecordWriter 中的 write 方法，示例代码如下：   public void write (Writable w) throws IOException {     String strReplace = ((Text)w).toString().replace( "\001" , "@$_$@" );     Text txtReplace = new Text();     txtReplace.set(strReplace);     byte [] output = txtReplace.getBytes();     bytesWritable .set(output, 0, output. length );     writer .write( bytesWritable ); }*复制代码*`自定义 outputformat/inputformat 后，在建表时需要指定 outputformat/inputformat ，如下示例：stored as INPUTFORMAT 'com.aspire.search.loganalysis.hive.SearchLogInputFormat' OUTPUTFORMAT 'com.aspire.search.loganalysis.hive.SearchLogOutputFormat'b) 通过 SerDe(serialize/deserialize) ，在数据序列化和反序列化时格式化数据。这种方式稍微复杂一点，对数据的控制能力也要弱一些，它使用正则表达式来匹配和处理数据，性能也会有所影响。但它的优点是可以自定义表属性信息 SERDEPROPERTIES ，在 SerDe 中通过这些属性信息可以有更多的定制行为。2、 数据导入导出a) 多版本日志格式的兼容由于 hive 的应用场景主要是处理冷数据（只读不写），因此它只支持批量导入和导出数据，并不支持单条数据的写入或更新，所以如果要导入的数据存在某些不太规范的行，则需要我们定制一些扩展功能对其进行处理。我们需要处理的日志数据存在多个版本，各个版本每个字段的数据内容存在一些差异，可能版本 A 日志数据的第二个列是搜索关键字，但版本 B 的第二列却是搜索的终端类型，如果这两个版本的日志直接导入 hive 中，很明显数据将会混乱，统计结果也不会正确。我们的任务是要使多个版本的日志数据能在 hive 数据仓库中共存，且表的 input/output 操作能够最终映射到正确的日志版本的正确字段。这里我们不关心这部分繁琐的工作，只关心技术实现的关键点，这个功能该在哪里实现才能让 hive 认得这些不同格式的数据呢？经过多方尝试，在中间任何环节做这个版本适配都将导致复杂化，最终这个工作还是在 inputformat/outputformat 中完成最为优雅，毕竟 inputformat 是源头， outputformat 是最终归宿。具体来说，是在前面提到的 inputformat 的 next 方法中和在 outputformat 的 write 方法中完成这个适配工作。b) Hive 操作本地数据一开始，总是把本地数据先传到 HDFS ，再由 hive 操作 hdfs 上的数据，然后再把数据从 HDFS 上传回本地数据。后来发现大可不必如此， hive 语句都提供了“ local ”关键字，支持直接从本地导入数据到 hive ，也能从 hive 直接导出数据到本地，不过其内部计算时当然是用 HDFS 上的数据，只是自动为我们完成导入导出而已。3、 数据处理日志数据的统计处理在这里反倒没有什么特别之处，就是一些 SQL 语句而已，也没有什么高深的技巧，不过还是列举一些语句示例，以示 hive 处理数据的方便之处，并展示 hive 的一些用法。a) 为 hive 添加用户定制功能，自定义功能都位于 hive_contrib.jar 包中add jar /opt/hadoop/hive-0.5.0-bin/lib/hive_contrib.jar;b) 统计每个关键词的搜索量，并按搜索量降序排列，然后把结果存入表 keyword_20100603 中create table keyword_20100603 as select keyword,count(keyword) as count from searchlog_20100603 group by keyword order by count desc;c) 统计每类用户终端的搜索量，并按搜索量降序排列，然后把结果存入表 device_20100603 中create table device_20100603 as select device,count(device) as count from searchlog_20100603 group by device order by count desc;d) 创建表 time_20100603 ，使用自定义的 INPUTFORMAT 和 OUTPUTFORMAT ，并指定表数据的真实存放位置在 '/LogAnalysis/results/time_20100603' （ HDFS 路径），而不是放在 hive 自己的数据目录中create external table if not exists time_20100603(time string, count int) stored as INPUTFORMAT 'com.aspire.search.loganalysis.hive.XmlResultInputFormat' OUTPUTFORMAT 'com.aspire.search.loganalysis.hive.XmlResultOutputFormat' LOCATION '/LogAnalysis/results/time_20100603';e) 统计每秒访问量 TPS ，按访问量降序排列，并把结果输出到表 time_20100603 中，这个表我们在上面刚刚定义过，其真实位置在 '/LogAnalysis/results/time_20100603' ，并且由于 XmlResultOutputFormat 的格式化，文件内容是 XML 格式。insert overwrite table time_20100603 select time,count(time) as count from searchlog_20100603 group by time order by count desc;f) 计算每个搜索请求响应时间的最大值，最小值和平均值insert overwrite table response_20100603 select max(responsetime) as max,min(responsetime) as min,avg(responsetime) as avg from searchlog_20100603;g)创建一个表用于存放今天与昨天的关键词搜索量和增量及其增量比率，表数据位于 '/LogAnalysis/results/keyword_20100604_20100603' ，内容将是 XML 格式。create external table if not exists keyword_20100604_20100603(keyword string, count int, increment int, incrementrate double) stored as INPUTFORMAT 'com.aspire.search.loganalysis.hive.XmlResultInputFormat' OUTPUTFORMAT 'com.aspire.search.loganalysis.hive.XmlResultOutputFormat' LOCATION '/LogAnalysis/results/keyword_20100604_20100603';h)设置表的属性，以便 XmlResultInputFormat 和 XmlResultOutputFormat 能根据 output.resulttype 的不同内容输出不同格式的 XML 文件。alter table keyword_20100604_20100603 set tblproperties ('output.resulttype'='keyword');i) 关联今天关键词统计结果表（ keyword_20100604 ）与昨天关键词统计结果表（ keyword_20100603 ），统计今天与昨天同时出现的关键词的搜索次数，今天相对昨天的增量和增量比率，并按增量比率降序排列，结果输出到刚刚定义的 keyword_20100604_20100603 表中，其数据文件内容将为 XML 格式。insert overwrite table keyword_20100604_20100603 select cur.keyword, cur.count, cur.count-yes.count as increment, (cur.count-yes.count)/yes.count as incrementrate from keyword_20100604 cur join keyword_20100603 yes on (cur.keyword = yes.keyword) order by incrementrate desc;4、用户自定义函数 UDF部分统计结果需要以 CSV 的格式输出，对于这类文件体全是有效内容的文件，不需要像 XML 一样包含 version ， encoding 等信息的文件头，最适合用 UDF(user define function) 了。UDF 函数可直接应用于 select 语句，对查询结构做格式化处理之后，再输出内容。自定义 UDF 需要继承 org.apache.hadoop.hive.ql.exec.UDF ，并实现 evaluate 函数， Evaluate 函数支持重载，还支持可变参数。我们实现了一个支持可变字符串参数的 UDF ，支持把 select 得出的任意个数的不同类型数据转换为字符串后，按 CSV 格式输出，由于代码较简单，这里给出源码示例：`public String evaluate(String... strs) {     StringBuilder sb = new StringBuilder();     for ( int i = 0; i < strs. length ; i++) {        sb.append(ConvertCSVField(strs[i])).append( ',' );     }     sb.deleteCharAt(sb.length()-1);     return sb.toString(); }*复制代码*`需要注意的是，要使用 UDF 功能，除了实现自定义 UDF 外，还需要加入包含 UDF 的包，示例：add jar /opt/hadoop/hive-0.5.0-bin/lib/hive_contrib.jar;然后创建临时方法，示例：CREATE TEMPORARY FUNCTION Result2CSv AS ‘com.aspire.search.loganalysis.hive. Result2CSv';使用完毕还要 drop 方法，示例：DROP TEMPORARY FUNCTION Result2CSv;5、输出 XML 格式的统计结果前面看到部分日志统计结果输出到一个表中，借助 XmlResultInputFormat 和 XmlResultOutputFormat 格式化成 XML 文件，考虑到创建这个表只是为了得到 XML 格式的输出数据，我们只需实现 XmlResultOutputFormat 即可，如果还要支持 select 查询，则我们还需要实现 XmlResultInputFormat ，这里我们只介绍 XmlResultOutputFormat 。前面介绍过，定制 XmlResultOutputFormat 我们只需重写 write 即可，这个方法将会把 hive 的以 ’\001’ 分隔的多字段数据格式化为我们需要的 XML 格式，被简化的示例代码如下：`public void write(Writable w) throws IOException {        String[] strFields = ((Text) w).toString().split( "\001" );        StringBuffer sbXml = new StringBuffer();        if ( strResultType .equals( "keyword" )) {   sbXml.append( "<record><keyword>" ).append(strFields[0]).append(   "</keyword><count>" ).append(strFields[1]).append(       "</count><increment>" ).append(strFields[2]).append(   "</increment><rate>" ).append(strFields[3]).append( "</rate></result>" );        }        Text txtXml = new Text();        byte [] strBytes = sbXml.toString().getBytes( "utf-8" );        txtXml.set(strBytes, 0, strBytes. length );        byte [] output = txtXml.getBytes();        bytesWritable .set(output, 0, output. length );        writer .write( bytesWritable );   }*复制代码*`其中的 strResultType .equals( "keyword" ) 指定关键词统计结果，这个属性来自以下语句对结果类型的指定，通过这个属性我们还可以用同一个 outputformat 输出多种类型的结果。alter table keyword_20100604_20100603 set tblproperties ('output.resulttype'='keyword');仔细看看 write 函数的实现便可发现，其实这里只输出了 XML 文件的正文，而 XML 的文件头和结束标签在哪里输出呢？所幸我们采用的是基于 outputformat 的实现，我们可以在构造函数输出 version ， encoding 等文件头信息，在 close() 方法中输出结束标签。这也是我们为什么不使用 UDF 来输出结果的原因，自定义 UDF 函数不能输出文件头和文件尾，对于 XML 格式的数据无法输出完整格式，只能输出 CSV 这类所有行都是有效数据的文件。五、总结Hive 是一个可扩展性极强的数据仓库工具，借助于 hadoop 分布式存储计算平台和 hive 对 SQL 语句的理解能力，我们所要做的大部分工作就是输入和输出数据的适配，恰恰这两部分 IO 格式是千变万化的，我们只需要定制我们自己的输入输出适配器， hive将为我们透明化存储和处理这些数据，大大简化我们的工作。本文的重心也正在于此，这部分工作相信每一个做数据分析的朋友都会面对的，希望对您有益。

## 存储与压缩

## 存储与索引
- 分区与分桶可以看做文件块上的索引可以快速定位到数据块，那么底层的存储格式Parquet和Orc 可以看做是文件里面的索引，可以快速定位到数据