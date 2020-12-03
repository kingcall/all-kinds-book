### Type Casting in Java With Conversion Examples

Type casting in Java is used to cast one type (primitive or object) to another type. Whenever you try to assign data of one type to variable of another type, type conversion happens in Java.

Type conversion in Java may be classified into following scenarios.

- [Widening primitive conversions](https://www.netjstech.com/2017/03/type-casting-in-java.html#Wideningconversion)
- [Widening reference conversions](https://www.netjstech.com/2017/03/type-casting-in-java.html#WideningRefconversion)
- [Narrowing primitive conversions](https://www.netjstech.com/2017/03/type-casting-in-java.html#Narrowingconversion)
- [Narrowing reference conversions](https://www.netjstech.com/2017/03/type-casting-in-java.html#NarrowingRefconversion)

Out of these four; widening primitive type conversions and widening reference type conversions happen automatically and type casting is not required. Type casting in Java is required in the case of narrowing primitive type conversions and narrowing reference conversions.

### Widening primitive conversions

If destination type (type to which you are converting) is larger than the source type, you are widening the type of your source type. This type conversion will happen with out problem and automatically i.e. type casting is not required in this case.

**As example-**

```
int i = 10;
float f = i; // Assigning int to float
```

i which is an int can safely be assigned to float variable as float is compatible with int and also wider than int.

- Refer [Automatic numeric type promotion in Java](https://www.netjstech.com/2015/04/java-automatic-numeric-type-promotion.html) to know more about widening primitive conversions.

### Widening reference conversions

Same way you can have a widening reference conversion. That is applicable in an [inheritance](https://www.netjstech.com/2015/04/inheritance-in-java.html) scenario where a **parent-child** relationship exists.

For example if there is a parent class **A** and a child class **B** that extends class A then reference type A can safely hold reference type of class B.

```
A a;
B b = new B();
a = b; // Widening conversion from sub type to super type
```

### Type casting in Java

Though automatic conversion is convenient and helpful, it may not always be possible. Especially in case of narrowing conversions. Again, narrowing conversion can be of two types-

- Narrowing primitive conversions
- Narrowing reference conversions

### Narrowing primitive conversions

As the name suggests if you try to fit a value into a source that is narrower than the original type of the value then it is a **narrowing conversion**.

**As example**– If we do the exact opposite of what we did in the example of widening conversion and try to assign a float value to an int.

```
int i;
float f = 19.6f;
i = f; // Compile-time error (Type mismatch: cannot convert from float to int)
```

As you see, you get a compile-time error if you try to assign a float value to an int as it is a narrowing conversion. In case of narrowing conversion you need to explicitly type cast it to make it work.

### General form of type casting in Java

```
(type) value;
```

Here **type** is the type to which the value has to be converted.

So in the above example we have to add an **explicit cast** to int.

### Java type casting example

```
int i;
float f = 19.6f;
i = (int)f;
System.out.println("value " + i); 
```

**Output**

```
value 19
```

Here you can see that the fractional part is truncated.

### Narrowing reference conversions

A [super](https://www.netjstech.com/2015/04/super-in-java.html) type can hold reference to an object of itself or the sub-types. But doing the opposite, when you want a conversion from super-type to sub-type, you will need **type casting**.

Since the conversion is from super-type to sub-type it is called **narrowing reference conversion**.

One important thing to always remember is; an object can only be type cast to its own class or one of its super-type, if you try to cast to any other object you may either get a compile-time error or a class-cast exception (run-time).

**Narrowing reference conversion example**

If we take the same example as used in widening reference conversion where there is a **class A** and a child **class B** that extends class A then reference type A can safely hold reference type of class B. But now we’ll try the opposite too.

```
A a;
B b = new B()
a = b; // OK widening conversion
b = a; // Compile-time error as it is a narrowing conversion
```

What you need to do to avoid compile-time error is-

```
b = (B) a;
```

### Why type casting in Java required

You may have a scenario where child class has methods of its own apart from inheriting methods from the super class or [overriding methods](https://www.netjstech.com/2015/04/method-overriding-in-java.html) of the super class.

Being a good programmer and always trying to achieve [polymorphism](https://www.netjstech.com/2015/04/polymorphism-in-java.html) you will make sure that the reference of the sub-class is held by the super-class object. But one problem is, then you can’t call those methods which are exclusive to sub-class.

In order to call those methods you need **casting to the type**. Let’s try to understand it with an example.

### Type casting Java example code

Here I have a class hierarchy where **Payment** is an [interface](https://www.netjstech.com/2015/05/interface-in-java.html) and there are two classes **CashPayment** and **CardPayment** implementing the Payment interface.

**Payment interface**

```
public interface Payment {
 public boolean proceessPayment(double amount);
}
```

**CashPayment class**

```
import org.netjs.examples.interfaces.Payment;

public class CashPayment implements Payment {
 @Override
 public boolean proceessPayment(double amount) {
  System.out.println("Cash payment done for Rs. " + amount);
  return true;
 }
}
```

**CardPayment class**

```
import org.netjs.examples.interfaces.Payment;

public class CardPayment implements Payment {

 @Override
 public boolean proceessPayment(double amount) {
  System.out.println("Card Payment done for Rs. " + amount);
  return true;
 }
 
 public void printSlip(){
  System.out.println("Printing slip for payment" );
 }
}
```

In **CardPayment** class, note that, there is an extra method **printSlip()** which is exclusive to this class. Now when you do some payments using the class as given below-

```
import org.netjs.examples.interfaces.Payment;

public class PaymentDemo {
 public static void main(String[] args) {
  PaymentDemo pd = new PaymentDemo();
  Payment payment = new CashPayment();
  pd.doPayment(payment, 100);
  payment = new CardPayment();
  pd.doPayment(payment, 300);
  //int i = 10;
  //float f = i;
  
  int i;
  float f = 19.6f;
  i = (int)f;
  System.out.println("value " + i);
 }
 
 public void doPayment(Payment pd, int amt){
  pd.proceessPayment(amt);
  pd.printSlip();
 }
}
```

This method call **pd.printSlip();** gives following compile-time error as the Payment object has no idea of the printSlip() method, thus the error-

*The method printSlip() is undefined for the type Payment*

If you want to call printSlip() method you need a cast back to CardPayment class type. Beware that there are two child classes and you don’t need that casting for CashPayment class object. Which means you need to use **instanceof operator** in conjunction with the casting.

- Refer [instanceof Operator in Java](https://www.netjstech.com/2017/03/instanceof-operator-in-java.html) to know more about instanceof operator in Java.

With these corrections the code will now look like-

```
import org.netjs.examples.interfaces.Payment;

public class PaymentDemo {

 public static void main(String[] args) {
  PaymentDemo pd = new PaymentDemo();
  Payment payment = new CashPayment();
  pd.doPayment(payment, 100);
  payment = new CardPayment();
  pd.doPayment(payment, 300);
  //int i = 10;
  //float f = i;
  
  int i;
  float f = 19.6f;
  i = (int)f;
  System.out.println("value " + i);
 }
 
 public void doPayment(Payment pd, int amt){
  pd.proceessPayment(amt);
  
  if (pd instanceof CardPayment){
   CardPayment cardPay = (CardPayment)pd;
   cardPay.printSlip();
  }
 }
}
```

**Output**

```
Cash payment done for Rs. 100.0
Card Payment done for Rs. 300.0
Printing slip for payment
value 19
```

Here you can see how **instanceof** operator is used to make sure that the object is indeed of type CardPayment and then casting is done to the CardPayment type, which makes it possible to call the printSlip() method.

Reference:[ https://docs.oracle.com/javase/specs/jls/se8/html/jls-5.html](https://docs.oracle.com/javase/specs/jls/se8/html/jls-5.html)

That's all for this topic **Type Casting in Java With Conversion Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!