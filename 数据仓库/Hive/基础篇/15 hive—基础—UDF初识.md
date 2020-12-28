

[TOC]

## 用户自定义函数



hive作为一个sql查询引擎，自带了一些基本的函数，比如`count`(计数)，`sum`(求和)，有时候这些基本函数满足不了我们的需求，这时候就要写`hive hdf(user defined funation)`，又叫用户自定义函数。

关于Hive 的内置函数可以看[Hive函数大全](https://blog.csdn.net/king14bhhb/article/details/111765573)

### UDF 创建与使用步骤

- 添加相关依赖，创建项目，这里我用的管理工具是maven,所以我创建的也是一个maven 项目(这个时候你需要选择合适的依赖版本，主要是Hadoop 和 Hive,可以使用`hadoop version`和`hive --version` 来分别查看版本)

- 继承`org.apache.hadoop.hive.ql.exec.UDF`类，实现evaluate方法，然后打包
- 使用 `add`方法添加jar 包到分布式缓存，**如果jar包是上传到$HIVE_HOME/lib/目录以下，就不需要执行add命令了**
- 通过`create temporary function`创建临时函数，不加`temporary`就创建了一个永久函数；
- 在SQL 中使用你创建的UDF

### 例子

下面我把pom 文件的主要内容帖在这里，因为我这个是所有的udf 函数都在一个项目里所以有其他的一些依赖，大家只选择自己需要的即可

```xml
<dependencies>
  <dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>${hadoop.version}</version>
    <scope>provided</scope>
  </dependency>
  <dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-hdfs</artifactId>
    <version>${hadoop.version}</version>
    <scope>provided</scope>
  </dependency>
  <dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>${hive.version}</version>
    <scope>provided</scope>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
  </dependency>

  <dependency>
    <groupId>org.ansj</groupId>
    <artifactId>ansj_seg</artifactId>
    <version>5.1.6</version>
    <scope>compile</scope>
  </dependency>

  <dependency>
    <groupId>com.janeluo</groupId>
    <artifactId>ikanalyzer</artifactId>
    <version>2012_u6</version>
  </dependency>
</dependencies>
```



#### 例一 判断字符串是否包含100

下面是一个判断hive表字段是否包含`’100’`这个子串的简单`udf`:

```java
package com.kingcall.bigdata.HiveUDF.demo;

import org.apache.hadoop.hive.ql.exec.UDF;

public class IsContains100 extends UDF{

    public String evaluate(String s){

        if(s == null || s.length() == 0){
            return "0";
        }

        return s.contains("100") ? "1" : "0";
    }
}
```

使用maven将其打包，进入`hive cli`，输入命令：

```shell
add jar /Users/liuwenqiang/workspace/code/idea/HiveUDF/target/HiveUDF-0.0.4.jar;
create temporary function isContains100 as 'com.kingcall.bigdata.HiveUDF.demo.IsContains100';
```

创建完临时函数，即可使用这个函数了：

```sql
select isContains100('abc100def') from table limit 1;
```

#### 例二

通过读取mysql数据库中的规则，为hive中的workflow返回对应的，类型：

```
type workflow
a	1
a	2
b	11
b	22
b	33
```

**需求**：我们希望，将hive的workflow字段取值为 1，2的变为类型(type)`a`，取值为11,22,33的全部变为`b`，就是归类的意思。

这个udf可以这么实现：

```java
package com.js.dataclean.hive.udf.hm2.workflow;

import org.apache.hadoop.hive.ql.exec.UDF;

import java.sql.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class GetWorkflow extends UDF{

    private static final String host = "0.0.0.0";
    private static final String port = "3306";
    private static final String database = "root";
    private static final String userName = "root";
    private static final String password = "123456";
    private static String url = "";
    private static final String driver = "com.mysql.jdbc.Driver";
    private static Connection conn = null;
    private static Map<String, List<String>> workflowType = null;

    static {
        url = "jdbc:mysql://" + host + ":" + port + "/" + database;
        try {
            // Class.forName(driver);
            conn = DriverManager.getConnection(url, userName, password);
            workflowType = getWorkflowType(conn);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    private static Map<String, List<String>> getWorkflowType(Connection conn){
        Map<String, List<String>> workflowType = new HashMap<>();
        String sql = "select * from flash_player_workflow";
        PreparedStatement ps = null;
        try {
            ps = conn.prepareStatement(sql);
            ResultSet rs = ps.executeQuery();
            while (rs.next()){
                String workflow = rs.getString("workflow");
                String type = rs.getString("flag");

                List<String> workflows = workflowType.get(type);
                if(workflows == null){
                    workflows = new ArrayList<>();
                }
                workflows.add(workflow);
                workflowType.put(type, workflows);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {

            // 关闭链接
            if(conn != null){
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
        return workflowType;

    }

    public String evaluate(String s){
        assert workflowType != null;

        for(String type:workflowType.keySet()){
            List<String> workflows = workflowType.get(type);
            if(workflows.contains(s)){
                return type;
            }
        }

        return s;
    }

}
```

打好`jar`包，创建函数: workflow2type（省略），然后使用：

```sql
select workflow2type(workflow) from table;

a
a
b
b
b
```

这样就把很多取值归为几个大类了。

### 查看hive function的用法

查month 相关的函数

```sql
show functions like '*month*';
```

查看 add_months 函数的用法

```sql
desc function add_months;
```

查看 add_months 函数的详细说明并举例

```sql
desc function extended add_months;
```

### hive 中的 UDAF

可以看出，udf就是一个输入一个输出，输入一个性别，返回’男’或者’女’，如果我们想实现`select date,count(1) from table`，统计每天的流量呢？这就是一个分组统计，显然是多个输入，一个输出，这时候udf已经不能满足我们的需要，就需要写udaf，`user defined aggregare function`(用户自定义聚合函数)。

这里写一个字符串连接函数，相当于`concat`的功能，将多行输入，合并为一个字符串，当然了hive中有字符串连接函数，这里是举例说明`UDAF`的用法：

```java
package com.js.dataclean.hive.udaf.hm2;

import com.js.dataclean.utils.StringUtil;
import org.apache.hadoop.hive.ql.exec.UDAF;
import org.apache.hadoop.hive.ql.exec.UDAFEvaluator;

public class MutiStringConcat extends UDAF{
    public static class SumState{
        private String sumStr;
    }

    public static class SumEvaluator implements UDAFEvaluator{
        SumState sumState;

        public SumEvaluator(){
            super();
            sumState = new SumState();
            init();
        }

        @Override
        public void init() {
            sumState.sumStr = "";
        }

        /**
         * 来了一行数据
         * @param s
         * @return
         */
        public boolean iterate(String s){
            if(!StringUtil.isNull(s)){
                sumState.sumStr += s;
            }
            return true;
        }

        /**
         * 状态传递
         * @return
         */
        public SumState terminatePartial() {
            return sumState;
        }

        /**
         * 子任务合并
         * @param state
         * @return
         */
        public boolean merge(SumState state){
            if(state != null){
                sumState.sumStr += state.sumStr;
            }
            return true;
        }

        /**
         * 返回最终结果
         * @return
         */
        public String terminate(){
            return sumState.sumStr;
        }
    }
}
```

用法，与udf一样，还是需要打包并且到hive cli中注册使用。

*关于UDAF开发注意点：*

- 需要`import org.apache.hadoop.hive.ql.exec.UDAF`以及`org.apache.hadoop.hive.ql.exec.UDAFEvaluator`,这两个类都是必须的
- 函数类需要继承UDAF类，内部类Evaluator实现UDAFEvaluator接口
- Evaluator需要实现 init、iterate、terminatePartial、merge、terminate这几个函数
  - init函数类似于构造函数，用于UDAF的初始化
  - iterate接收传入的参数，并进行内部的轮转。其返回类型为boolean
  - terminatePartial无参数，其为iterate函数轮转结束后，返回乱转数据，iterate和terminatePartial类似于hadoop的Combiner
  - merge接收terminatePartial的返回结果，进行数据merge操作，其返回类型为boolean
  - terminate返回最终的聚集函数结果

### 临时与永久函数

`Hive`自定义函数分为临时与永久函数，顾名思义，分别是临时使用和永久有效使用的意思。

#### 临时函数

临时函数，关闭会话就结束了生命周期，下次要想使用，需要重新注册。

```sql
add jar /path/xx.jar（存储在本地磁盘）

// 临时注册UDF函数（hive会话生效）
create temporary function 函数名 as '包名.类名';
```

删除临时函数：

- drop temporary function 数据库名.函数名;

#### 永久函数

永久函数一旦注册，可以在hive cli，远程连接hiveserver2等地方永久使用，步骤为：

- 先上传jar包到HDFS

- 永久注册

```sql
CREATE FUNCTION 函数名 AS '包名.类名' USING JAR 'hdfs:///path/xxxx.jar';
```

**注意**：指定jar包路径需要是`hdfs`路径。

- 删除永久函数：

```sql
drop function 数据库名.函数名字;
```

新增的永久函数，比如在hive cli命令行注册的，可能会在beeline或者hiveserver2远程连接时，提示不存在该函数。解决办法是，在无法使用UDF的HiveServer2上，执行`reload function`命令，将`MetaStore`中新增的UDF信息同步到HiveServer2内存中。

## 总结

1. 这一节我们主要介绍了如何创建UDF ,主要集中在创建的流程，以及如何使用，例子都是很简单的，就是为了讲清楚一些细节的东西，下一节我们学习几个有用的UDF,可以很好的解决一些工作中遇到的一些问题
2. 需要注意的是我们看到当我们去继承`UDF`类的时候提示我们去实现`GenericUDF` ,所以后面我们的实现都是基于`GenericUDF`实现的，如果是针对简单的数据类型（比如String、Integer等）可以使用UDF，如果是针对复杂的数据类型（比如Array、Map、Struct等），可以使用GenericUDF，另外，GenericUDF还可以在函数开始之前和结束之后做一些初始化和关闭的处理操作。