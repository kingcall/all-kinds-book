### Difference Between Encapsulation And Abstraction in Java

Encapsulation and abstraction are two of the fundamental OOP concepts other two being [polymorphism](https://www.netjstech.com/2015/04/polymorphism-in-java.html) and [inheritance](https://www.netjstech.com/2015/04/inheritance-in-java.html). Some times people do get confused between **encapsulation** and **abstraction** as both are used to *"hide something"*. So in this post let's try to see the difference between encapsulation and abstraction in Java.

First let's try to define these two OOPS concepts encapsulation and abstraction to get a better idea-

- Encapsulation

  \- Encapsulation means keeping together the implementation (code) and the data it manipulates (variables). Having proper encapsulation ensures that the code and data both are safe from misuse by outside entity. So, in a way Encapsulation is more about

   

  data hiding

  .

  It is a [Java class](https://www.netjstech.com/2015/04/class-in-java.html) which is the foundation of encapsulation in Java.

  Following image shows the concept of encapsulation. Fields and methods are encapsulated with in the class and the methods of the class can access and manipulate fields. Outside entity can't access fields directly but need to call methods which in turn can access data.

  [![Encapsulation in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/20:23:39-Encapsulation-20201203202338481.png)](https://1.bp.blogspot.com/-V_K_ibVXekM/XCHFPDbRCCI/AAAAAAAABEs/PXdRDEbeCggSQZUY9pcqPsvoS6TX29OTwCLcBGAs/s1600/Encapsulation.png)

  **→** Refer [Encapsulation in Java](https://www.netjstech.com/2015/04/encapsulation-in-java.html) to know more about encapsulation in Java.

- Abstraction

  \- Abstraction means

   

  hiding the complexity

   

  and only showing the essential features of the object. So in a way,

   

  abstraction

   

  means abstracting/hiding the real working and we, as a user, knowing only how to use.

  A very good example from Java would be - Java Database Connectivity (JDBC) API which provides universal data access from the Java programming language. Using the [JDBC API](https://www.netjstech.com/2017/12/java-jdbc-overview-jdbc-tutorial.html), we can access virtually any data source without knowing how the driver for that particular data source is implemented. All we have is an API with a given set of methods.

  ```
  PreparedStatement pstmt = conn.prepareStatement(SQL);
  ```

  While writing this line we don't bother how connection or prepareStatement is implemented in Oracle, SQLServer, DB2 or MySQL. For us as a user those details are abstracted and we just know how to use JDBC API to interact with DBs. 

  Abstraction in Java is achieved through [interface](https://www.netjstech.com/2015/05/interface-in-java.html) and [abstract class](https://www.netjstech.com/2015/04/abstract-class-in-java.html).

  Following image shows the concept of abstraction. User interacts to the real implementation through API methods. How is API actually implemented is abstracted from the user.

  [![Abstraction in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/20:23:39-Abstraction-20201203202338845.png)](https://4.bp.blogspot.com/-_4lL8SmtMDA/XCHIKFjKkGI/AAAAAAAABFQ/pa4h2z73ad4Jo3_g2pZuu5vjkkPeClHcACLcBGAs/s1600/Abstraction.png)

  →

   

  Refer

   

  Abstraction in Java

   

  to know more about abstraction in Java.

Hope you have got some idea now about these two terms, so let's see the differences between these terms.

### Difference between Encapsulation and Abstraction in Java

1. Encapsulation

    

   is about

    

   keeping together the implementation and the data it manipulates

   . In a properly encapsulated Java class, method defines how member variables can be used. That access control to the methods and the variables is achieved through

    

   access modifiers

    

   (public, private, default & protected).

   

   Abstraction

    

   is about hiding the implementation and only giving the information about how to use it. Abstraction in Java is achieved through interfaces and abstract classes. 

2. 

3. One of the

    

   benefit of Encapsulation

    

   is that it helps in managing code that is changed frequently by keeping that in one place thus providing maintainability and flexibility. In fact that is one of the design principle;

    

   Encapsulate what varies

   .

   

   Benefit of Abstraction

    

   is to provide a contract through interface or may be a bare skeletal implementation through abstract class. Where generalized implementations are provided based on that contract but user just sees the API.

    Any number of classes can implement an interface and each class is free to provide their own implementation. That's how using interfaces, Java fully utilizes "

   one interface, multiple methods

   " aspect of polymorphism. 

4. In Encapsulation the emphasis is on keeping together the the implementation and the data it manipulates so encapsulation is more about data safety that is why encapsulation is a data hiding mechanism.
   Abstraction means hiding the complex implementation details. User just need to know what is to be called to get the work done with out knowing how work is done. So, abstraction puts more focus on hiding the implementation details.

### Example of Encapsulation in Java

Here we have a class called **Employee** and in the class there are public methods that provide access to the class’ fields (If you have noticed access modifier for all the fields is private). These methods for setting/getting the values of the fields are called *setters and getters*.

Any other class (in this example **EmployeeTest**) that wants to access the variables should access them through these getters and setters.
There is also one other method **getFullName()** in the same class which is using the fields of the class, that method can use the variables directly.

```
public class Employee {
    private String lastName;
    private String firstName;
    private String empId;
    private int age;
    public String getLastName() {
        return lastName;
    }
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
    public String getFirstName() {
        return firstName;
    }
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    public String getEmpId() {
        return empId;
    }
    public void setEmpId(String empId) {
        this.empId = empId;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    
    // method for getting full name
    public String getFullName(){
     return this.firstName + " " + this.lastName;
    }  
}
public class EmployeeTest {

 public static void main(String[] args) {
  Employee emp = new Employee();
  /*This line will give compiler error
  age field can't be used directly as it is private */
  // emp.age = 40;
     emp.setEmpId("E001");
     emp.setAge(40);
     emp.setFirstName("Ram");
     emp.setLastName("Chandra");
     System.out.println("Age- " + emp.getAge());
     System.out.println("Employee ID- " + emp.getEmpId());
     System.out.println("Full Name- " + emp.getFullName());

 }
}
```

### Example of Abstraction in Java

Here we have an interface **Ipayment** with one method **doPayment()**. There are two implementing classes **CashPayment** and **CreditCardPayment** that provide implementation for cash payment and credit card payment respectively. But as a user all you need to know is that you have to call doPayment() method.

```
public interface IPayment {
 void doPayment(double amount);
}
public class CashPayment implements IPayment {

 @Override
 public void doPayment(double amount) {
  System.out.println("Cash payment " + amount);
  
 }
}
public class CreditCardPayment implements IPayment {

 @Override
 public void doPayment(double amount) {
  System.out.println("CreditCard payment " + amount);

 }

}
public class Payment {
 private static final String CASH = "CASH";
 private static final String CC = "CC";
 public static void main(String[] args) {
  Payment payment = new Payment();
  payment.executePayment(CASH, 34.45);
  payment.executePayment(CC, 34.45);
  
 }
 
 private void executePayment(String mode, double amount){
  IPayment payRef;
  // If payment is in CASH
  if(mode.equalsIgnoreCase(CASH)){
   payRef = new CashPayment();
   payRef.doPayment(amount);  
  }
  // If payment is through credit card
  else if(mode.equalsIgnoreCase(CC)){
   payRef = new CreditCardPayment();
   payRef.doPayment(129.78);
  }
 }

}
```

In **Payment** class based on the mode of payment appropriate class is called. Of course there are better ways to do it using factory and encapsulating the logic in another class but here focus is more on knowing what is Abstraction.

That's all for this topic **Difference Between Encapsulation And Abstraction in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!