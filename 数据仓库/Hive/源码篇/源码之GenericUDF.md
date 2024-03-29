##  GenericUDF

开发自定义UDF函数有两种方式，一个是继承org.apache.hadoop.hive.ql.exec.UDF，另一个是继承org.apache.hadoop.hive.ql.udf.generic.GenericUDF；

如果是针对简单的数据类型（比如String、Integer等）可以使用UDF

如果是针对复杂的数据类型（比如Array、Map、Struct等），可以使用GenericUDF，另外，GenericUDF还可以在函数开始之前和结束之后做一些初始化和关闭的处理操作。

接下来说说为什么要有这样的改进吧：一个是可以接受和返回复杂数据类型了，例如Array什么的结构体类型，而不像UDF类那样只能是int、string之类的基本类型（当然真正代码中定义的是包装过的后缀为Writable的类型，但还是表示基本类型）；新的改进可以接受可变长度以及无限长度的参数了，因为可以用数组来表示输入参数了，而不需要像UDF类的实现类那样，要几种参数组合，就得重载几种方法；最重要的改进是可以通过DeferredObject类来实现所谓的”short-circuit“优化

提供了更好的参数和返回值检查, 效率更高, 适合处理HIVE中的复杂数据类型

[@Describtion](https://github.com/Describtion) 注解是可选的，用于对函数进行说明，其中的 *FUNC* 字符串表示函数名，当使用 DESCRIBE FUNCTION 命令时，替换成函数名。
[@Describtion](https://github.com/Describtion) 包含三个属性：
**name**：用于指定Hive中的函数名。
**value**：用于描述函数的参数。
**extended**：额外的说明，如，给出示例。当使用 DESCRIBE FUNCTION EXTENDED name 的时候打印。而且，Hive 要使用 UDF，需要把 Java 文件编译、打包成 jar 文件，然后将 jar 文件加入到 CLASSPATH 中，最后使用 CREATE FUNCTION 语句定义这个 Java 类的函数：



### 重写方法

继承org.apache.hadoop.hive.ql.udf.generic.GenericUDF之后，需要重写几个重要的方法：

**public void** configure(MapredContext context) {}

//可选，该方法中可以通过context.getJobConf()获取job执行时候的Configuration；

//可以通过Configuration传递参数值

**public ObjectInspector initialize(ObjectInspector[] arguments)**

//必选，该方法用于函数初始化操作，并定义函数的返回值类型；

//比如，在该方法中可以初始化对象实例，初始化数据库链接，初始化读取文件等； 检测函数参数类型。检测函数参数个数

**public Object evaluate(DeferredObject[] args){}**

//必选，函数处理的核心方法，用途和UDF中的evaluate一样；

**public String getDisplayString(String[] children)**

//必选，显示函数的帮助信息

**public void close(){}**

//可选，map完成后，执行关闭操作