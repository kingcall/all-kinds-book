### Transaction Management in Java-JDBC

This post provides detail about JDBC transaction management with examples for starting a transaction in JDBC, committing and rolling back a transaction, setting savepoint for transaction rollback.

**Table of contents**

1. [What is a transaction](https://www.netjstech.com/2018/01/transaction-in-java-jdbc.html#Transaction)
2. [Transaction management in JDBC](https://www.netjstech.com/2018/01/transaction-in-java-jdbc.html#TransactionMgmt)
3. [Transaction Rollback in JDBC](https://www.netjstech.com/2018/01/transaction-in-java-jdbc.html#TransactionRollback)
4. [JDBC transaction management example](https://www.netjstech.com/2018/01/transaction-in-java-jdbc.html#JDBCTransactionExp)
5. [Transaction Isolation Levels in JDBC](https://www.netjstech.com/2018/01/transaction-in-java-jdbc.html#JDBCTransactionIsolation)
6. [Setting Savepoint for transaction rollback](https://www.netjstech.com/2018/01/transaction-in-java-jdbc.html#JDBCTransactionSavepoint)



### What is a transaction

A transaction is a *single logical unit of work*, to complete a transaction all of the actions belonging to the unit of work have to be completed. If any of the action with in the unit of work fails all of the work done as part of the transaction has to be undone and DB should return to its state before the transaction begin.

**As Example**– A classic example for explaining transaction is transferring amount from one account to another. While transferring amount, actions are as follows-

1. Amount is deducted from one account.
2. Amount is added to another account.

As part of a transaction *both of these actions should happen or none*.

If first action happens and amount is deducted from the account and then some error happens and that amount is not added to the second account then your *DB is left in an inconsistent state*.

If both actions happen as a single unit of work in a transaction then any error after the first step will result in **rollback** of the action. Which means the action where amount was deducted from the account is undone and DB returns to its state before the transaction begin.

### Transaction management in JDBC

When a connection is created in JDBC it is in auto-commit mode by default meaning each individual SQL statement is treated as a transaction and is automatically committed right after it's execution is completed.

If you want to run two or more statements grouped into a transaction in JDBC then you need to disable the **auto-commit mode**. You can do it using the **setAutoCommit()** method.

```
connection.setAutoCommit(false);
```

By setting the auto-commit as false you are ensuring that the statements are not automatically committed after execution. You can commit it once all the statements grouped into a transaction are completed without any error.

In order to commit the transaction you will have to explicitly call the **commit method**.

```
connection.commit();
```

commit method call will make all changes made as part of the transaction permanent and releases any database locks currently held by this [Connection](https://netjs.blogspot.com/2017/12/connection-interface-in-java-jdbc.html) object.

### Transaction Rollback in JDBC

If any action which is part of the transaction fails then you need to *rollback the whole transaction*. You can do it by calling the **rollback()** method.

```
connection.rollback();
```

Calling rollback() will undo all changes made in the current transaction and releases any database locks currently held by this Connection object.

To sum it up following steps are to be followed for transaction management in JDBC -

1. For your connection object you need to set auto-commit mode as false.
2. Once all the actions with in the transaction are completed with out any error, make changes permanent by calling the commit method.
3. If there is any error in any action, rollback the transaction by calling the rollback() method.

### JDBC transaction management example

Let’s see an example of transaction management in JDBC. We’ll use the same example of transferring amount from one account to another.

DB used is **MySQL**, schema is **netjs** and there is a table **account** which has fields **acct_num** and **amount** which will be used in SQL queries.

In the code there are two methods **deductAmount()** and **addAmount()** which are used to deduct amount form the account and add the same amount to another account respectively. These two methods are grouped into a transaction.

```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class TransactionDemo {

 public static void main(String[] args) {
  TransactionDemo td = new TransactionDemo();
  td.transferAmount(1, 7, 100);
 }
 
 /**
  * 
  * @param fromAcct
  * @param toAcct
  * @param amount
  */
 public void transferAmount(int fromAcct, int toAcct, int amount){
  Connection connection = null;
  try {
   connection = getConnection();
   // Disabling auto commit
   connection.setAutoCommit(false);
   /** Transaction starts **/
   deductAmount(connection, fromAcct, amount);
   addAmount(connection, toAcct, amount);
   /** Transaction ends **/
   // Commit transaction
   connection.commit();
  }catch (SQLException e) {
     e.printStackTrace();
     if(connection != null){
       try {
         System.out.println("Rollingback the transaction");
         connection.rollback();
       } catch (SQLException e1) {
         // TODO Auto-generated catch block
         e1.printStackTrace();
       }
     }
      
  }finally{
       if(connection != null){
        //closing connection 
        try {
          connection.close();
        } catch (SQLException e) {
           // TODO Auto-generated catch block
           e.printStackTrace();
        }
      } // if condition
  }// finally
 }
 
 private void deductAmount(Connection connection, int acctNum, int amount) 
    throws SQLException{

  
  String updateSQL = "update account as t1 JOIN "
    + " (select acct_num, (balance - ?) as bal from account"
    + " where acct_num = ?) As acct "
    + " ON  t1.acct_num = acct.acct_num"
    + " set t1.balance = acct.bal"
    + " where t1.acct_num = ?";

  PreparedStatement prepStmt = null;
  try {
   prepStmt = connection.prepareStatement(updateSQL);
   prepStmt.setInt(1, amount);
   prepStmt.setInt(2, acctNum);
   prepStmt.setInt(3, acctNum);
   int count = prepStmt.executeUpdate();
   System.out.println("Count of rows updated " + count);
   if(count == 0){
    throw new SQLException("Account num not found " + acctNum);
   }
  }finally{
    if(prepStmt != null){
     prepStmt.close();
    }
  }
 }
 
 private void addAmount(Connection connection, int acctNum, int amount) throws SQLException{
  String updateSQL = "update account as t1 JOIN "
    + " (select acct_num, (balance + ?) as bal from account"
    + " where acct_num = ?) As acct "
    + " ON  t1.acct_num = acct.acct_num"
    + " set t1.balance = acct.bal"
    + " where t1.acct_num = ?";

  PreparedStatement prepStmt = null;
  try {
   prepStmt = connection.prepareStatement(updateSQL);
   prepStmt.setInt(1, amount);
   prepStmt.setInt(2, acctNum);
   prepStmt.setInt(3, acctNum);
   int count = prepStmt.executeUpdate();
   System.out.println("Count of rows updated " + count);
   if(count == 0){
    throw new SQLException("Account num not found " + acctNum);
   }
  }finally{
    if(prepStmt != null){
     prepStmt.close();
    }
  }
  
 }
 
 /**
  * Method for getting the connection
  * @return
  */
 public static Connection getConnection(){
  Connection connection = null;
   try {
    // Loading driver
    Class.forName("com.mysql.jdbc.Driver");
    
    // Creating connection
    connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/netjs", 
                 "root", "admin");
    
   } catch (ClassNotFoundException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
   } catch (SQLException e) {
       // TODO Auto-generated catch block
       e.printStackTrace();
   }
  return connection;
 }

}
```

### Transaction Isolation Levels in JDBC

Connection interface provides a set of fields for specifying transaction isolation level-

- **TRANSACTION_NONE**- A constant indicating that transactions are not supported.
- **TRANSACTION_READ_COMMITTED**- A constant indicating that dirty reads are prevented; non-repeatable reads and phantom reads can occur.
- **TRANSACTION_READ_UNCOMMITTED**- A constant indicating that dirty reads, non-repeatable reads and phantom reads can occur.
- **TRANSACTION_REPEATABLE_READ**- A constant indicating that dirty reads and non-repeatable reads are prevented; phantom reads can occur.
- **TRANSACTION_SERIALIZABLE**- A constant indicating that dirty reads, non-repeatable reads and phantom reads are prevented.

Connection interface also provides a method **setTransactionIsolation(int level)** for setting the transaction isolation level.

**As example** – If you want to set the transaction isolation level to read committed.

```
connection.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
```

### Setting Savepoint for transaction rollback

You can also set a save point in your transaction and rollback to that savepoint rather than rolling back the whole transaction. There is an overloaded variant of rollback method that takes a Savepoint argument.

Connection interface provides two overloaded methods for setting savepoint -

- **setSavepoint()**- Creates an unnamed savepoint in the current transaction and returns the created Savepoint object.
- **setSavepoint(String name)**- Creates a savepoint with the given name in the current transaction and returns the created Savepoint object.

If you want to release the created savepoint you can do it using the **releaseSavepoint()** method. As example to release the created Savepoint sp1.

```
connection.releaseSavepoint(sp1)
```

**JDBC transaction - Setting savepoint example**

If we take the same example as used above for transaction and add a new functionality to add the transaction information to a new table **transdata** (From Account, To Account, Amount).

There is savepoint crated after the transfer is done and then move further to insert the transaction information in the table transdata. In case there is some error while inserting record in transdata you just want to log that information and rollback to the savepoint created after the transfer is done rather than rollingback the whole transaction.

```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Savepoint;

public class TransactionDemo {
  public static void main(String[] args) {
    TransactionDemo td = new TransactionDemo();
    td.transferAmount(2, 3, 100);
  }
 
  /**
  * @param fromAcct
  * @param toAcct
  * @param amount
  */
  public void transferAmount(int fromAcct, int toAcct, int amount){
    Connection connection = null;
    Savepoint sp1 = null;
    try {
      connection = getConnection();
      // Disabling auto commit
     connection.setAutoCommit(false);
     /** Transaction starts **/
     deductAmount(connection, fromAcct, amount);
     addAmount(connection, toAcct, amount);
     /** Transaction ends **/
     // setting save point here
     sp1 = connection.setSavepoint("TrasferDoneSavePoint");
     // inserting transaction data
     insertTransaction(connection, fromAcct, toAcct, amount);
     // Commit transaction
     connection.commit();      
    }catch (SQLException e) {
      e.printStackTrace();
      if(connection != null){
        try {
          // if savepoint is not reached, rollback the transaction
          if(sp1 == null){
            System.out.println("Rollingback the transaction");
            connection.rollback();
          }else{
            System.out.println("Rollingback to savepoint");
            // rollback to savepoint
            connection.rollback(sp1);
            // Commit till the savepoint
            connection.commit();
          }
        } catch (SQLException e1) {
          // TODO Auto-generated catch block
          e1.printStackTrace();
        }
      }  
    }finally{
      if(connection != null){
        //closing connection 
        try {
          connection.close();
        } catch (SQLException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
      } // if condition
    }// finally
  }
 
  /**
  * @param connection
  * @param acctNum
  * @param amount
  * @throws SQLException
  */
  private void deductAmount(Connection connection, int acctNum, int amount) throws SQLException{
    String updateSQL = "update account as t1 JOIN "
      + " (select acct_num, (balance - ?) as bal from account"
      + " where acct_num = ?) As acct "
      + " ON  t1.acct_num = acct.acct_num"
      + " set t1.balance = acct.bal"
      + " where t1.acct_num = ?";

    PreparedStatement prepStmt = null;
    try {
      prepStmt = connection.prepareStatement(updateSQL);
      prepStmt.setInt(1, amount);
      prepStmt.setInt(2, acctNum);
      prepStmt.setInt(3, acctNum);
      int count = prepStmt.executeUpdate();
      System.out.println("Count of rows updated " + count);
      if(count == 0){
        throw new SQLException("Account num not found " + acctNum);
      }
    }finally{
      if(prepStmt != null){
        prepStmt.close();
      }
    }
  }
 
 /**
  * 
  * @param connection
  * @param acctNum
  * @param amount
  * @throws SQLException
  */
 private void addAmount(Connection connection, int acctNum, int amount) throws SQLException{
  String updateSQL = "update account as t1 JOIN "
    + " (select acct_num, (balance + ?) as bal from account"
    + " where acct_num = ?) As acct "
    + " ON  t1.acct_num = acct.acct_num"
    + " set t1.balance = acct.bal"
    + " where t1.acct_num = ?";

  PreparedStatement prepStmt = null;
  try {
   prepStmt = connection.prepareStatement(updateSQL);
   connection.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
   prepStmt.setInt(1, amount);
   prepStmt.setInt(2, acctNum);
   prepStmt.setInt(3, acctNum);
   int count = prepStmt.executeUpdate();
   System.out.println("Count of rows updated " + count);
   if(count == 0){
    throw new SQLException("Account num not found " + acctNum);
   }
  }finally{
    if(prepStmt != null){
     prepStmt.close();
    }
  }  
 }
 
 /**
  * 
  * @param connection
  * @param fromAcct
  * @param toAcct
  * @param amount
  * @throws SQLException
  */
 private void insertTransaction(Connection connection, int fromAcct, int toAcct, int amount) throws SQLException{
  String insertSQL = "Insert into transdata (from_acct, to_acct, amount) values (?, ?, ?)";
  PreparedStatement prepStmt = null;
  try {
   prepStmt = connection.prepareStatement(insertSQL);
   prepStmt.setInt(1, fromAcct);
   //prepStmt.setInt(2, toAcct);
   //prepStmt.setInt(3, amount);
   int count = prepStmt.executeUpdate();
   System.out.println("Count of rows inserted " + count);
   if(count == 0){
    throw new SQLException("Problem in inserting - " + " From Account: " + fromAcct + " To Account: "  +  toAcct) ;
   }
  }finally{
    if(prepStmt != null){
     prepStmt.close();
    }
  }
 }
 
  /**
  * Method for getting the connection
  * @return
  */
  public static Connection getConnection(){
    Connection connection = null;
    try {
      // Loading driver
      Class.forName("com.mysql.jdbc.Driver");
    
      // Creating connection
      connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/netjs", "root", "admin");    
    } catch (ClassNotFoundException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (SQLException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    return connection;
  }
}
```

That's all for this topic **Transaction Management in Java-JDBC**. If you have any doubt or any suggestions to make please drop a comment. Thanks!