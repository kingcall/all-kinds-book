### Constructor Overloading in Java

Like [method overloading in Java](https://www.netjstech.com/2015/04/method-overloading-in-java.html) there is also an option to have multiple constructors within the same class where the constructors differ in number and/or types of parameters, that process is known as **Constructor overloading in Java**. Depending on the parameters the appropriate overloaded constructor is called when the object is created.

### Why Constructor overloading is needed

You may need to initialize objects in different ways for that you need different constructors and that is why constructor overloading in Java is needed.

A practical example of constructor overloading in Java would be [ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html) class where there are 2 constructors related to initial capacity.

```
public ArrayList(int initialCapacity)
public ArrayList() {
  this(10);
}
```

In one of the constructor user can pass the initial capacity for the ArrayList and the ArrayList will be created with the specified capacity, in another case user can use the no-arg constructor and ArrayList would have the default initial capacity of 10.

Here is an example to understand why constructor overloading is needed. Consider the following class-

```
class ConstrOverLoading{
 private int i;
 private int j;
 //Constructor with 2 params
 ConstrOverLoading(int i, int j){
  this.i = i;
  this.j = j;
 }
}  
```

As it can be seen the ConstrOverLoading() constructor takes 2 parameters. In this case all the objects of this class must have two parameters. Following statement will result in compile time error "*The constructor ConstrOverLoading() is undefined*".

```
ConstrOverLoading obj = new ConstrOverLoading();
```

If you want to initialize the objects of the class as shown above in different ways then you do need overloaded constructors.

```
class ConstrOverLoading{
 private int i;
 private int j;
 //Constructor with 2 params
 ConstrOverLoading(int i, int j){
  this.i = i;
  this.j = j;
 }
 //Constructor with 1 param
 ConstrOverLoading(int i){
  this.i = i;
  this.j = 0;
 }
 //no-arg Constructor
 ConstrOverLoading(){
 }
 public static void main(String[] args) {
  ConstrOverLoading obj = new ConstrOverLoading(5,6);
  ConstrOverLoading obj1 = new ConstrOverLoading();
 }
}  
```

### Java constructor overloading and no-arg constructor

When you have overloaded constructors in Java having a no-arg constructor is very important. It is because of the fact- *if any constructor is defined for a class then Java won't automatically insert a default constructor. Default constructor for the class will only be inserted in case no constructor is defined*.

For example if there is a class defined as follows-

```
class ConstrOverLoading{
 private int i;
 private int j;
 //Constructor with 2 params
 ConstrOverLoading(int i, int j){
  this.i = i;
  this.j = j;
 }
}  
```

Then trying to create an object without passing any parameter like below results in a compiler error.

```
ConstrOverLoading obj = new ConstrOverLoading();
```

The code fails to compile and throws an error "The constructor ConstrOverLoading() is undefined" because of the fact that if any constructor is defined for a class then Java won't define a default constructor.

### Using this with overloaded constructors

It is a common practice to use [this](https://www.netjstech.com/2015/04/this-in-java.html) with overloaded constructors in Java and let only one of the constructor do the initialization, all the other constructors merely make a call to that constructor. Let's see it with an example.

### Constructor overloading with this - Java Example

```
class ConstrOverLoading {
 int a;
 double b;
 ConstrOverLoading(int a, double b){
  this.a = a;
  this.b = b;
 }
 ConstrOverLoading(int a){
  this(a, 0.0);
 }
 ConstrOverLoading(){
  this(0);
 }
}
```

It can be noticed here that only one of the contructor ConstrOverLoading(int a, double b) is doing any assignment other constructors are merely invoking that constructor through this(). This use of **this** helps in reducing the **duplication of code**.

**Two points to note here-**

1. this() should be the first statement within the constructor in this case.
2. this() and [super()](https://www.netjstech.com/2015/04/super-in-java.html) can't be used with in the same constructor as it is required for both to be the first statement in the class.

That's all for this topic **Constructor Overloading in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!