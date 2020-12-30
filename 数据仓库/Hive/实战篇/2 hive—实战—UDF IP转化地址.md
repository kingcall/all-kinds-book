[TOC]

## IP 转化为地址

### 场景

落到ods层的用户日志里面通常都有一个ip字段，在进入dws层的时候，通常数据清洗需要将其转化为类似中国-湖北-武汉的格式，便于后面进行地域维度的统计分析，或者在查询的时候直接使用udf函数转化。

需要去这种服务商购买Ip-地址映射库，然后用他们给出的`demo`进行解析，这里分享一份：

```java
@Test
public void test2() throws IOException, IPv4FormatException {
    City city = new City("src/file/mydata4vipday2.datx");
    System.out.println(city.find("180.169.26.98"));
    String[] locations = city.find("122.115.230.26");
    for(String location : locations){
        System.out.println(location);
    }
}
```

在编写这种ip转地区的udf的时候，就可以按照demo里面的调用方式来写，然后将其注册为永久函数，便于后面使用。

### 代码实现

#### **第一步：**引入依赖

```
<dependency>
  <groupId>org.lionsoul</groupId>
  <artifactId>ip2region</artifactId>
  <version>1.7.2</version>
</dependency>
```



#### **第二步：**引入IP库

这个工具需要一份IP 地址库，其实到这里我们大概也能猜到，解析的性能好坏取决于上面代码的设计，而解析的准确性取决于IP库的完善程度以及是否能及时更新，这里其实就埋下了一个伏笔，和上一节的停用词一样，能否动态更新我们的数据对我们的UDF 至关重要



#### **第三步：**编写UDF

```java
@Description(name = "Ip2Region", value = "_FUNC_(ip) - return the ip address",
        extended = "Example: select _FUNC_('220.248.12.158') from src limit 1;\n"
                + "中国|0|上海|上海市|联通")
public class Ip2Region extends GenericUDF {

    private Converter converter;
    private DbSearcher searcher;

    @Override
    public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
        converter = ObjectInspectorConverters.getConverter(arguments[0], PrimitiveObjectInspectorFactory.writableStringObjectInspector);

  		// 加载IP 地址库 测试的时候使用 /ip2region.db 上线的时候使用  ip2region.db UDF 中读取不到这个文件，需要注册，注册之后就可以使用  ip2region.db 路径读取了
      // String dbPath = Ip2Region.class.getResource("/ip2region.db").getPath();
      String dbPath = "ip2region.db";

        File file = new File(dbPath);
        if (file.exists() == false) {
            System.out.println("Error: Invalid ip2region.db file");
            return null;
        }
        DbConfig config = null;
        try {
            config = new DbConfig();
            searcher = new DbSearcher(config, dbPath);
        } catch (DbMakerConfigException | FileNotFoundException e) {
            e.printStackTrace();
        }


        return PrimitiveObjectInspectorFactory.writableStringObjectInspector;

    }

    @Override
    public Object evaluate(DeferredObject[] arguments) throws HiveException {
        if (arguments[0].get() == null) {
            return null;
        }
        Text ip = (Text) converter.convert(arguments[0].get());

        //查询算法
        int algorithm = DbSearcher.BTREE_ALGORITHM;
        try {


            Method method = null;
            switch (algorithm) {
                case DbSearcher.BTREE_ALGORITHM:
                    method = searcher.getClass().getMethod("btreeSearch", String.class);
                    break;
                case DbSearcher.BINARY_ALGORITHM:
                    method = searcher.getClass().getMethod("binarySearch", String.class);
                    break;
                case DbSearcher.MEMORY_ALGORITYM:
                    method = searcher.getClass().getMethod("memorySearch", String.class);
                    break;
            }

            DataBlock dataBlock = null;
            if (Util.isIpAddress(ip.toString()) == false) {
                System.out.println("Error: Invalid ip address");
            }

            dataBlock = (DataBlock) method.invoke(searcher, ip.toString());

            return new Text(dataBlock.getRegion());

        } catch (Exception e) {
            e.printStackTrace();
        }

        return null;
    }

    @Override
    public String getDisplayString(String[] children) {
        return null;
    }
}
```



#### **第四步：**编写测试用例

```java
@Test
public void ip2Region() throws HiveException {
    Ip2Region udf = new Ip2Region();
    ObjectInspector valueOI0 = PrimitiveObjectInspectorFactory.javaStringObjectInspector;
    ObjectInspector[] init_args = {valueOI0};
    udf.initialize(init_args);
    String ip = "220.248.12.158";

    GenericUDF.DeferredObject valueObj0 = new GenericUDF.DeferredJavaObject(ip);

    GenericUDF.DeferredObject[] args = {valueObj0};
    Text res = (Text) udf.evaluate(args);
    System.out.println(res.toString());
}
```

运行结果，看起来没问题

![image-20201229083519414](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/29/08:35:20-image-20201229083519414.png)



#### **第五步：**创建UDF 并使用

首先执行打包 ` mvn clean package -DskipTests=true -Dmaven.javadoc.skip=true`

```
这里需要注意一下，你这里的路径要和udf 中的路径匹配，如果不匹配依然找不到，你可通过list file 的方式获取加载后的文件的路径
add file ip2region.db;
add jar /Users/liuwenqiang/workspace/code/idea/HiveUDF/target/HiveUDF-0.0.4.jar;
create temporary function ip2Region as 'com.kingcall.bigdata.HiveUDF.Ip2Region';
select ip2Region("220.248.12.158");
```

![image-20201229084358481](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/29/08:43:59-image-20201229084358481.png)

看起来中文的描述信息在展示的时候有点问题，接下来我们尝试使用一下

![image-20201229115502798](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/29/11:55:04-image-20201229115502798.png)

再次强调一下路径的问题，很重要

![image-20201229115704876](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/29/11:57:05-image-20201229115704876.png)

## 总结

1. ip 转化为地址是非常常用的的一种UDF
2. 我们让UDF 读取外部文件，需要注意的是当你add file之后，你需要确定文件的路径，然后将其路径给赋值给UDF 中文件路径的变量 

