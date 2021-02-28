# Hadoop 分布式缓存

## 什么是 Hadoop 分布式缓存

分布式缓存是 Hadoop MapReduce 框架提供的一种数据缓存机制。它可以缓存只读文本文件，压缩文件，jar包等文件。一旦对文件执行缓存操作，那么每个执行 map/reduce 任务的节点都可以使用该缓存的文件。
![hadoop分布式缓存原理](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570095316000_20191003173516221425-20210111223341216.png)

## 分布式缓存的优点

- 存储复杂的数据 它分发了简单、只读的文本文件和复杂类型的文件，如jar包、压缩包。这些压缩包将在各个slave节点解压。特点就是，它会将需要缓存的文件分发到各个执行任务的子节点的机器中，**各个节点可以自行读取本地文件系统上的数据进行处理**。
- 数据一致性 Hadoop分布式缓存追踪了缓存文件的修改时间戳。然后当job在执行时，它也会通知这些文件不能被修改。使用hash 算法，缓存引擎可以始终确定特定键值对在哪个节点上。所以，缓存cluster只有一个状态，它永远不会是不一致的。
- 单点失败 分布式缓存作为一个跨越多个节点独立运行的进程。因此单个节点失败，不会导致整个缓存失败。

## 分布式缓存的使用

旧版本的 DistributedCache 已经被注解为过时，以下为 Hadoop-2.2.0 以上的新API接口。

```
Job job = Job.getInstance(conf);
//将hdfs上的文件加入分布式缓存,可以看出来时一个特定的文件
job.addCacheFile(new URI("hdfs://url:port/filename#symlink"));
```

不同文件类型的添加方法：

```
job.addArchiveToClassPath(archive); // 缓存jar包到task运行节点的classpath中 
job.addFileToClassPath(file); // 缓存普通文件到task运行节点的classpath中 
job.addCacheArchive(uri); // 缓存压缩包文件到task运行节点的工作目录 
job.addCacheFile(uri) // 缓存普通文件到task运行节点的工作目录
```

由于新版 API 中已经默认创建符号连接，所以不需要再调用 setSymlink(true) 方法了，可以下面代码来查看是否开启了创建符号连接。

```
System.out.println(context.getSymlink());
```



之后在 map/reduce 函数中可以通过 context 来访问到缓存的文件，一般是重写 setup 方法来进行初始化：

> ```
> 在Mapper类的setup方法中读取分布式缓存文件：Mapper类主要有4个方法：setup(),map(),cleanup(),run(),run()方法调用其它三个方法，顺序是：setup()-map()-cleanup。并且setup()执行且仅执行一次，主要用来为map()方法做一些准备工作，所以很多初始化的工作尽量放在这里做。Reducer类也是类似的。
> setup()方法实现示例如下：
> ```

```
@Override
protected void setup(Context context) throws IOException, InterruptedException {
        super.setup(context);
        if (context.getCacheFiles() != null && context.getCacheFiles().length > 0) {
        String path = context.getLocalCacheFiles()[0].getName();
        File itermOccurrenceMatrix = new File(path);
        FileReader fileReader = new FileReader(itermOccurrenceMatrix);
        BufferedReader bufferedReader = new BufferedReader(fileReader);
        String s;
        while ((s = bufferedReader.readLine()) != null) {
            //TODO:读取每行内容进行相关的操作
        }
        bufferedReader.close();
        fileReader.close();
    }
}
```

得到的path为本地文件系统上的路径。

这里的 getLocalCacheFiles 方法也被注解为过时了，只能使用 context.getCacheFiles 方法，和 getLocalCacheFiles 不同的是，getCacheFiles 得到的路径是 HDFS 上的文件路径，如果使用这个方法，那么程序中读取的就不再试缓存在各个节点上的数据了，相当于共同访问 HDFS 上的同一个文件。可以直接通过符号连接来跳 过getLocalCacheFiles 获得本地的文件。

## 分布式缓存的大小

可以在文件 mapred-site.xml 中设置，默认为10GB。

## 注意事项

- 需要分发的文件必须是存储在 HDFS 上了。
- 文件只读。
- 不缓存太大的文件，执行task之前对文件进行分发，影响task的启动速度。