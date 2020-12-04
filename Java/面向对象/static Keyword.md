### static Keyword in Java With Examples

When we want to define a [class](https://www.netjstech.com/2015/04/class-in-java.html) member that can be used independently of any [object](https://www.netjstech.com/2015/04/object-in-java.html) of that class we use **static keyword** in Java. When a class member is defined as **static it is associated with the class**, rather than with any object.

**The static keyword in Java can be used with-**

- [variable](https://www.netjstech.com/2015/04/static-in-java.html#staticvariable) (also called class variable)
- [method](https://www.netjstech.com/2015/04/static-in-java.html#staticmethod) (also called class method)
- [static block](https://www.netjstech.com/2015/04/static-in-java.html#staticblock)
- [static nested class](https://www.netjstech.com/2015/04/static-in-java.html#staticnested)

### Static Variable in Java

A variable declared as **static** in Java is associated with the class rather than with any object. When objects of its class are created, copy of static variable is not created per object. All objects of the class **share the same static variable**. A static variable can be **accessed directly using the class** and doesn't need any object.

Since static variables are associated with the class, not with instances of the class, they are part of the **class data in the method area**.

### Usage of Static variable

One common use of static keyword in Java is to create a constant value that's at a class level and applicable to all created objects, it is a common practice to declare constants as **public final static**. **static** so that it is at class level, shared by all the objects, [final](https://www.netjstech.com/2015/04/final-in-java.html) so that the value is not changed anytime.

### static field example

```
public class Employee {
 int empId;
 String name;
 String dept;
 // static constant
 static final String COMPANY_NAME = "XYZ";
 Employee(int empId, String name, String dept){
  this.empId = empId;
  this.name = name;
  this.dept = dept;
 }
 
 public void displayData(){
  System.out.println("EmpId = " + empId + " name= " + name + " dept = " + 
  dept + " company = " + COMPANY_NAME);
 }
 public static void main(String args[]){  
  Employee emp1 = new Employee(1, "Ram", "IT");
  Employee emp2 = new Employee(2, "Krishna", "IT");
  emp1.displayData();
  emp2.displayData();
 }
}
```

**Output**

```
EmpId = 1 name= Ram dept = IT company = XYZ
EmpId = 2 name= Krishna dept = IT company = XYZ
```

**Another usage** is when there is a requirement to have a counter at the class level. Since static variable is associated to the class and will have the same value for every instance of the class, where as value of non static variable will vary from object to object. So any counter which should be at the class level has to be declared as static.

```
class Counter{
 // at class level and gets memory only once
 static int count = 0;
 
 Counter(){
  count++;
 }
}
```

### Static Method in Java

Same as static variable, **static method** is also associated with the class rather than objects of the class. Static method can be called directly using the class name like **ClassName.static_method()** rather than with the object of the class.

**main()** method is the most common example of static method, [main() method is declared static](https://www.netjstech.com/2015/04/why-main-method-is-static-in-java.html) because main() must be called before any object of the class exists.

### Usage of Static method

The most common usage of static method is to declare all the utility methods (like date formatting, conversion of data types) as static. Reason being these utility methods are not part of actual business functionality so should not be associated with individual objects.

### Example of Static method

```
public class StaticDemo {
 static void staticMethod(int i){
  System.out.println("in static method " + i);
 }
 
 public static void main(String[] args) {  
  StaticDemo.staticMethod(5);
 }
}
```

It can be seen here that the staticMethod is accessed with the class itself.

**StaticDemo.staticMethod(5);**

**Please note that** it won't be an error if class object is used to access a static method, though compiler will warn to use class instead.

### Restrictions with static method in Java

1. static method can directly call other static methods only.

   ```
   public class StaticDemo {
    // Non Static method
    void staticMethod(int i){
     System.out.println("in static method " + i);
    }
    
    public static void main(String[] args) {  
     staticMethod(5); // Compiler error
    }
   }
   ```

   It will throw compiler error "

   [Can not make static reference to the non-static method](https://www.netjstech.com/2015/06/static-reference-to-non-static-method-field-error.html)

   ".

2. static method can directly access static data only

   ```
   public class StaticDemo {
    // non static field
    int i;
    
    public static void main(String[] args) {  
     i = 10; // Compiler error
    }
   }
   ```

   It will throw compiler error "

   Cannot make a static reference to the non-static field

   ".

3. Static methods cannot refer to [this](https://www.netjstech.com/2015/04/this-in-java.html) or [super](https://www.netjstech.com/2015/04/super-in-java.html) in any way.

### Static Block in Java

A [static block](https://www.netjstech.com/2016/02/static-block-in-java.html) in a class is executed only once, when the class is first loaded, even before the main method.

### Usage of Static Block in Java

A static final variable that is not initialized at the time of declaration is known as static blank final variable in Java. It can be initialized only in static block.
If some computation is needed, in order to initialize a static variable, that can be done in static block.

```
public class StaticDemo {
 // static blank final variable
 static final int i;
 static int b;
 static {
  System.out.println("in static block");
  i = 5;
  b = i * 5;
  System.out.println("Values " + i + " " +  b);
 }
 
 public static void main(String args[]){  
  System.out.println(" in main method ");
 }
}
```

**Output of the program**

```
in static block
Values 5 25
in main method
```

It can be seen that main method is called only after executing the static block. In static block the static blank final variable is initialized. Also Another static variable too is initialized with in the block after some computation.

### static nested class

Top level class can't be declared as static but a nested class can be declared as static, such a nested class is known as static nested class in Java.

- Refer [Nested Class And Inner Class in Java](https://www.netjstech.com/2017/05/nested-class-inner-class-java.html) to know more about static nested class in Java.

**Points to note about static keyword in Java-**

- **static** keyword in Java can be used with variables, methods and nested class in Java. There can also be a static block.
- **static** variables and methods are associated with class itself rather than with any object.
- **static** variables will have a single copy that will be used by all the objects.
- static members of a class can be accessed even before object of a class is created.
- static fields are not thread safe so proper synchronization should be done in case of several threads accessing the static variable.
- static methods can directly call other static methods only and directly access static fields only.
- static methods can be [overloaded](https://www.netjstech.com/2015/04/method-overloading-in-java.html) but can not be [overridden](https://www.netjstech.com/2015/04/method-overriding-in-java.html).
- static block is executed in a class at the time of class loading even before the main method.
- With [static import](https://www.netjstech.com/2015/05/static-import-in-java.html) it is possible to access any static member (static field or method) of the class with out qualifying it with class name.

That's all for this topic **static Keyword in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!