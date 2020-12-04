### Object in Java

In object-oriented terms object is an instance of the class, which gets its state and related behavior from the class. An object stores its state in fields and exposes its behavior through methods.

As explained in the post [Class in Java](https://www.netjstech.com/2015/04/class-in-java.html) when a new class is created essentially a new data type is created. This type can be used to declare objects of that type.

### Creating object in Java

Creation of an object for a class in Java is done in three steps-

- **Declaration**- Declare a variable of the class type, note that no object is defined yet. This variable can refer to an object as and when the object is created.
- **Instantiation**- Create an object and assign it to the variable created in step 1. This object creation is done using new operator.
- **Initialization**- Class name followed by parenthesis after the new operator (new classname()) means calling the [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html) of the class to initialize the object.

### Java object creation example

If we have a class called **Rectangle** as this-

```
class Rectangle{
  double length;
  double width;
  
  //methods 
  ……
  ……
}
```

Then declaring object of the class Rectangle would be done as-

```
Rectangle rectangle;
```

This statement declares an object rectangle of the class Rectangle. It does not hold any thing yet.

With the following statement a new object rectangle is created (instantiated), which means rectangle objects holds reference to the created object of Rectangle class.

```
rectangle = new Rectangle();
```

[![Declaring an object in Java](https://2.bp.blogspot.com/-f0irGv7D-0g/VS0yhO6Q2zI/AAAAAAAAACw/64Ot06ypepw/s1600/object%2Binitialization.png)](https://2.bp.blogspot.com/-f0irGv7D-0g/VS0yhO6Q2zI/AAAAAAAAACw/64Ot06ypepw/s1600/object%2Binitialization.png)

**Declaring and Instantiating an object of type Rectangle**

As it can be seen from the figure the rectangle variable holds reference to the allocated Rectangle object.

The [new operator](https://www.netjstech.com/2017/02/object-creation-using-new-operator-java.html) allocates memory for an object at run time and returns a reference to that memory which is then assigned to the variable of the class type.

In the statement **new Rectangle()**, the class name Rectangle() followed by parenthesis means that the constructor of the class Rectangle would be called to initialize the newly created object. This step is the initialization step.

- Refer [Constructor in Java](https://www.netjstech.com/2015/04/constructor-in-java.html) to know more about object initialization.

Generally this three step process of declaring, creating and initializing an object in Java is written in a single statement as following-

```
Rectangle rectangle = new Rectangle();
```

### Benefits of using class and object in Java object oriented programming

This [encapsulation](https://www.netjstech.com/2015/04/encapsulation-in-java.html) of variables and methods into a class and using them through objects provides a number of benefits including-

- **Modularity**- The application can be divided into different functional categories. Then code can be written for many independent objects implementing the specific functionality. Once created an object can be easily passed around inside the system creating a series of operations to execute functionality.
- **Information-hiding**- By encapsulating the variables and methods and providing interaction to the outside entities only through object's methods, an object hides its internal implementation from the outside world.
- **Code re-use**- While working on an application we use several third party libraries. That is an example how we use several already existing objects (already created and tested) in our application.
- **Pluggability and debugging ease**- Since a whole application comprises of several working components in form of objects, we can simply remove one object and plug in another as replacement in case a particular object is not providing the desired functionality.

That's all for this topic **Object in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!