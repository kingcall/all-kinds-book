先解释一下几个名词：

- metadata ：hive元数据，即hive定义的表名，字段名，类型，分区，用户这些数据。一般存储关系型书库mysql中，在测试阶段也可以用hive内置Derby数据库。

- metastore ：hivestore服务端。主要提供将DDL，DML等语句转换为MapReduce，提交到hdfs中。

- hiveserver2：hive服务端。提供hive服务。客户端可以通过beeline，jdbc（即用java代码链接）等多种方式链接到hive。

- beeline：hive客户端链接到hive的一个工具。可以理解成mysql的客户端。如：navite cat 等。

其它语言访问`hive`主要是通过hiveserver2服务，HiveServer2(HS2)是一种能使客户端执行Hive查询的服务。HiveServer2可以支持对 HiveServer2 的嵌入式和远程访问，支持多客户端并发和身份认证。旨在为开放API客户端（如JDBC和ODBC）提供更好的支持。



会启动一个hive服务端默认端口为：10000，可以通过beeline，jdbc，odbc的方式链接到hive。hiveserver2启动的时候会先检查有没有配置hive.metastore.uris，如果没有会先启动一个metastore服务，然后在启动hiveserver2。如果有配置hive.metastore.uris。会连接到远程的metastore服务。这种方式是最常用的。



# Python连接Hive

`Python3`访问hive需要安装的依赖有：

- pip3 install thrift
- pip install PyHive
- pip install sasl
- pip install thrift_sasl

```
from pyhive import hive
conn = hive.Connection(host='localhost', port=10000, database='default')
cursor = conn.cursor()
cursor.execute('show tables')

for result in cursor.fetchall():
    print(result)
```

## java 
```
    // 此Class 位于 hive-jdbc的jar包下
    private static String driverName = "org.apache.hive.jdbc.HiveDriver";
    private static String Url = "jdbc:hive2://localhost:10000/";
    private static Connection conn;

    public static Connection getConnnection() {
        try {
            Class.forName(driverName);
            conn = DriverManager.getConnection(Url, "root", "www1234");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
            System.exit(1);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return conn;
    }

    public static PreparedStatement prepare(Connection conn, String sql) {
        PreparedStatement ps = null;
        try {
            ps = conn.prepareStatement(sql);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return ps;
    }

    public static void getAll(String tablename) {
        String sql = String.format("select behavior,count(1) as cnt from %s group by behavior order by cnt desc", tablename);
        System.out.println(sql);
        try {
            PreparedStatement ps = prepare(getConnnection(), sql);
            ResultSet rs = ps.executeQuery();
            int columns = rs.getMetaData().getColumnCount();
            while (rs.next()) {
                for (int i = 1; i <= columns; i++) {
                    System.out.print(rs.getString(i));
                    System.out.print("\t\t");
                }
                System.out.println();
            }
        } catch (SQLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }

    public static void main(String[] args) {
        String tablename = "ods.user_behavior";
        getAll(tablename);
    }
}
```



## streaming

```
  hive> FROM invites a INSERT OVERWRITE TABLE events SELECT TRANSFORM(a.foo, a.bar) AS (oof, rab) USING '/bin/cat' WHERE a.ds > '2008-08-09';
```