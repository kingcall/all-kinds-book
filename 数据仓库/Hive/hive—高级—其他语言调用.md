## python
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
```



## streaming

```
  hive> FROM invites a INSERT OVERWRITE TABLE events SELECT TRANSFORM(a.foo, a.bar) AS (oof, rab) USING '/bin/cat' WHERE a.ds > '2008-08-09';
```