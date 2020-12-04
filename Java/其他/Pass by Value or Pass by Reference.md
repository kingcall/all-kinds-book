### Java Pass by Value or Pass by Reference

Parameter passing in Java is pass by value or pass by reference is one question which causes a bit of confusion so let's try to clarify it in this post.

Before going into whether Java is pass by value or pass by reference let's first see what these 2 terms actually mean.

- **Pass by value**- In this case value of the actual parameter is copied into memory (stack) used by the parameters of the method, so there are two independent variables with their own memory.
- **Pass by reference**- In this case the memory address of the actual parameter is copied in the parameter of the method. Thus anytime method is using its parameter it is actually referencing the actual parameter.

### Passing primitive types as method parameter

At least there is no confusion with [primitive values](https://www.netjstech.com/2017/03/primitive-data-types-in-java.html) as it is easy to see that **primitives are passed by value** in Java. Let's see it with an example-

```
public class A {
 
 void displayData(int i){  
  //value of the passed primitive variable changed
  i++;
  System.out.println("Value of i inside method "  + i);
 }  
 public static void main(String args[]){ 
  A a = new A();
  int i = 5;
  // Sending variable i as param
  a.displayData(i);  
  System.out.println("Value of i inside main "  + i);
 }  
}
```

**Output of the program**

```
Value of i inside method 6
Value of i inside main 5
```

In the code when the displayData() method is called-

- Copy of variable i is made.
- That copy is sent to the displayData() method.
- Any change made to variable i with in displayData() method is local to that method and that copy of the variable.

Thus it can be seen that the changing of the passed int value in the method **doesn't affect the original value**.

### Passing class object as method parameter

When [object](https://www.netjstech.com/2015/04/object-in-java.html) is passed as a method parameter, people have some confusion whether it's pass by value or pass by reference in Java. Before going in to it let's be very clear that in **Java every thing is pass by value whether primitives or objects**. It does make sense as Java has no pointers thus no way to send memory addresses of the variables.

Still, if an object is passed as parameter in Java and any instance variable in that object is changed in the method that change is global i.e. *same value is reflected every where*. Let's see an example

```
public class A {
 private int i;
 //constructor
 A(int i){
  this.i = i;
 }

 void displayData(A obj){ 
  // object manipulated 
  obj.setI(7);
  System.out.println("Value of i inside method "  + obj.getI());
 }  
 public static void main(String args[]){ 
  A a = new A(5);
  System.out.println("Value of i inside main before method call "  + a.getI());
  a.displayData(a);  
  System.out.println("Value of i inside main after method call "  + a.getI());
 }
 //getter
 public int getI() {
  return i;
 }
//setter
 public void setI(int i) {
  this.i = i;
 }  
}
```

**Output of the program**

```
Value of i inside main before method call 5
Value of i inside method 7
Value of i inside main after method call 7
```

It can be seen from the output that changes done to the object in the method **displayData()** are reflected globally. When value of **i** is printed after the method call it is changed. **If java is pass by value then why that**?

It is because when we say

```
A a = new A();
```

The new operator allocates memory for the created object and returns a reference to that memory which is then assigned to the variable of the class type.

So, it can be said **“a”** is a variable which is holding a value and that value happens to be the reference to the memory address.

- Refer [Object Creation Using new Operator in Java](https://www.netjstech.com/2017/02/object-creation-using-new-operator-java.html) to know more about what happens while an object is created.

When the method is called with

```
a.displayData(a); 
```

copy of the reference value is created and passed into the other object which means in method **void displayData(A obj)**, obj parameter holds the copy of the reference value thus a and obj both are references to the same memory given to the object. **Refer below image to have clarity**.

[![pass by value or pass by reference](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/19:57:12-pass%252Bby%252Bvalue%252Bpass%252Bby%252Breference.png)](https://1.bp.blogspot.com/-v3vg3DKW8JA/VTfKfryh2ZI/AAAAAAAAAEk/agSmdD-r0aA/s1600/pass%2Bby%2Bvalue%2Bpass%2Bby%2Breference.png)

So to make it clear, in the case of object *Java copies and passes the reference by value*, not the actual reference itself. That is why any manipulation of the object in the method will alter the object, since the references point to the original objects. At the same time if reference itself is changed for the object that change is not reflected to the original object, which would have happened in case of pass by reference.

Let's write a Java program to understand this statement "If reference itself is changed for the obejct that change is not reflected to the original object" in detail.

```
public class A {
 private int i;
 A(int i){
  this.i = i;
 }
 void displayData(A obj){
  // change the reference
  obj = new A(8);
  System.out.println("Value of i inside method "  + obj.getI());
 }  
 public static void main(String args[]){ 
  A a = new A(5);
  System.out.println("Value of i inside main before method call "  + a.getI());
  a.displayData(a);  
  System.out.println("Value of i inside main after method call "  + a.getI());
 }
 
 // getter
 public int getI() {
  return i;
 }
 // setter
 public void setI(int i) {
  this.i = i;
 }  
}
```

In this program the reference of the object which was copied and passed as value to the parameter in method **displayData()** has been changed

```
obj = new A(8);
```

but it can be seen from the output that it doesn't change the original reference as it retains the original value of the variable i.

```
Value of i inside main before method call 5
Value of i inside method 8
Value of i inside main after method call 5
```

I hope I am able to make it clear that in Java, whether primitives or objects, everything is pass by value. If you have any doubt or any suggestions to make please drop a comment. Thanks!