### Connection Interface in Java-JDBC

In the post [Java JDBC Steps to Connect to DB](https://www.netjstech.com/2017/12/java-jdbc-steps-to-connect-to-db.html) we have already seen a complete example using the interfaces Driver, Connection, [Statement](https://www.netjstech.com/2017/12/statement-interface-in-java-jdbc.html) and [ResultSet](https://www.netjstech.com/2017/12/resultset-interface-in-java-jdbc.html) provided by the JDBC API. In this post we’ll see Java Connection interface in detail.

### Connection interface in JDBC

Connection [interface](https://www.netjstech.com/2015/05/interface-in-java.html) resides in **java.sql** [package](https://www.netjstech.com/2016/07/package-in-java.html) and it represents a session with a specific database you are connecting to. SQL statements that you want to execute, results that are returned all that happens with in the context of a connection.

You can get a Connection object by using the **getConnection()** method of the **DriverManager** [class](https://www.netjstech.com/2015/04/class-in-java.html).

Using Connection class object-

- You can get an object of Statement.
- You can get the information about the database (DatabaseMetaData) it is connecting to.
- Connection also provides method for transaction management like commit(), rollback().



### Fields in the Connection interface

Connection interface provides a set of fields for specifying transaction isolation level-

- **TRANSACTION_NONE**- A constant indicating that transactions are not supported.
- **TRANSACTION_READ_COMMITTED**- A constant indicating that dirty reads are prevented; non-repeatable reads and phantom reads can occur.
- **TRANSACTION_READ_UNCOMMITTED**- A constant indicating that dirty reads, non-repeatable reads and phantom reads can occur.
- **TRANSACTION_REPEATABLE_READ**- A constant indicating that dirty reads and non-repeatable reads are prevented; phantom reads can occur.
- **TRANSACTION_SERIALIZABLE**- A constant indicating that dirty reads, non-repeatable reads and phantom reads are prevented.

### Frequently used methods of the Connection

Some of the frequently used methods of the Connection are as follows-

**For creating statement**

- **createStatement()**- Creates a Statement object for sending SQL statements to the database.
- **prepareStatement(String sql)**- Creates a PreparedStatement object for sending parameterized SQL statements to the database.
- **prepareCall(String sql)**- Creates a CallableStatement object for calling database stored procedures.

There are also overloaded variant of these methods where you can specify the type of ResultSet and its [concurrency](https://www.netjstech.com/2016/05/java-concurrency-interview-questions.html) level.

**For getting information about the DB**

- **getMetaData()**- Returns a DatabaseMetaData object containing metadata about the connected database.

See example of using DatabaseMetaData here- [DatabaseMetaData Interface in Java-JDBC](https://www.netjstech.com/2017/12/databasemetadata-interface-in-java-jdbc.html).

**For transaction management**

- **setAutoCommit(boolean autoCommit)**- Sets this connection's commit mode to true or false.
- **setTransactionIsolation(int level)**- Attempts to changes the transaction isolation level for this Connection object to the one given.
- **rollback()**- Undoes all changes made in the current transaction and releases any database locks currently held by this Connection object.
- **commit()**- Makes all changes made since the previous commit/rollback permanent and releases any database locks currently held by this Connection object.

See details of Transaction management using JDBC here- [Transaction in Java-JDBC](https://www.netjstech.com/2018/01/transaction-in-java-jdbc.html).

**Reference**: https://docs.oracle.com/en/java/javase/12/docs/api/java.sql/java/sql/Connection.html

That's all for this topic **Connection Interface in Java-JDBC**. If you have any doubt or any suggestions to make please drop a comment. Thanks!