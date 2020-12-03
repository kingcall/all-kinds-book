### Access Modifiers in Java - Public, Private, Protected and Default

Access level modifiers in Java (public, private, protected, default) are used to control the visibility of the [class](https://www.netjstech.com/2015/04/class-in-java.html) or the members of the class i.e. fields and methods. These access modifiers determine whether other classes can use a particular field, invoke a particular method or create object of any given class.

**Table of contents**

1. [Types of Access Modifiers in Java](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html#AccessmodifierTypes)
2. [Access modifier with Java classes](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html#AccessmodifierClass)
3. [Java Access modifiers with fields](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html#AccessmodifierFields)
4. [Java Access modifier with methods](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html#AccessmodifierMethods)
5. [Access modifier with Constructors](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html#AccessmodifierConstructor)
6. [Class member access summary](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html#AccessSummary)



### Types of Access Modifiers in Java

- **private**- private modifier specifies that the member can only be accessed in its own class.
- **default (or package-private)**- If no specifier is used (which is known as default access) member is visible only within its own package.
- **protected**- protected modifier specifies that the member can only be accessed within its own package and by a subclass of its class in another package.
- **public**- public modifier specifies that member is visible to all classes everywhere.

Access modifiers in Java can be used with-

- Class
- Fields
- Methods
- Constructors

### Access modifier with Java classes

In Java programming language **only two** of the access modifiers can be used at the class level- **public** or **default**.

- If a class is declared with the access modifier as public, that class is visible to all classes everywhere.
- If a class has no modifier (the default), it is visible only within its own [package](https://www.netjstech.com/2016/07/package-in-java.html).

**Example of Java default access modifier**

If there is one class **DefaultClass** in package **org.netjs.examples**

```
package org.netjs.examples;

class DefaultClass {
 public void display(){
  System.out.println("display method : Default class");
 }
}
```

Then if you try to create object of this class in another class which resides in different package (**org.netjs.prog**) it will result in compile time error.

```
package org.netjs.prog;

public class Test {
 public static void main(String[] args) {
  // ERROR
  DefaultClass dc = new DefaultClass();
 }
}
```

### Java Access modifiers with fields

All the **four types** of access modifiers in Java- **public**, **protected**, **default**, **private** can be used with variables declared in the class.

- If a field is declared as public then it is visible to all classes in the same package or other packages.
- If a fields is declared with no access specifier (default) then it can be accessed by any class in the same package.
- If a field is defined as protected then it is accessible to any class in the same package or to any subclass (of the class where field is declared) in different package.
- If a field is defined private then that field can only be accessed in its own class.

Let's take an example when a field is protected-

If there is class **DefaultClass** in package **org.netjs.examples**

```
package org.netjs.examples;

public class DefaultClass {
 protected String item;
 public void display(){
  System.out.println("display method : Default class");
 }
}
```

Then in Test class in another package **org.netjs.examples** item variable won't be accessible. It will give "*field not visible*" error.

```
package org.netjs.examples;

public class Test {
  public static void main(String[] args) {
    DefaultClass dc = new DefaultClass();
    //Error
    dc.item = "laptop";
  }
}
```

If Test class **extends** DefaultClass then item variable will be accessible with the Test class object.

```
package org.netjs.examples;

public class Test extends DefaultClass {
 public static void main(String[] args) {
  Test t = new Test();
  t.item = "laptop";
 }
}
```

### Java Access modifier with methods

All the **four types** of access modifiers- **public**, **protected**, **default**, **private** can be used with methods of the class and access modifier for the methods work the same way as for the fields.

### Access modifier with Constructors

All **four types** of access modifiers in Java- **public**, **protected**, **default**, **private** can be used with constructors of the class.

In case [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html) of the class is **private** then the *object of that class can be created by that class only*. You might have seen that in Singleton design pattern.

In case no modifier is used (default) then the object of the class can be created by the classes with in the same package.

As example if there is a class **DefaultClass** within the package **org.netjs.examples**

```
public class DefaultClass {
  // Constructor
  DefaultClass(){
    System.out.println("In DefaultClass constructor ");
  }
  protected String item;
  public void display(){
    System.out.println("display method : Default class");
  }
}
```

Then trying to access constructor of **DefaultClass** in the class Test (sub class of DefaultClass) which resides in **org.netjs.prog** package will result in **compile time error**- "*The constructor DefaultClass() is not visible*"

```
package org.netjs.prog;

import org.netjs.examples.DefaultClass;

public class Test extends DefaultClass{
  Test(){
    super();
  }
  public static void main(String[] args) {
    Test t = new Test();
  }
}
```

In case **DefaultClass'** constructor is marked as **protected** then the above program will work. As *subclass in different package can access the protected constructor*.

If constructor of the class is public then its object can be created from anywhere- class residing in the same package or in a different package.

### Class member access summary

Following table shows the access levels for the class members with different access modifiers in Java.

|                                 | Private | No Modifier | Protected | Public |
| ------------------------------- | ------- | ----------- | --------- | ------ |
| Same class                      | Yes     | Yes         | Yes       | Yes    |
| Same package subclass           | No      | Yes         | Yes       | Yes    |
| Same package another class      | No      | Yes         | Yes       | Yes    |
| Different package subclass      | No      | No          | Yes       | Yes    |
| Different package another class | No      | No          | No        | Yes    |

That's all for this topic **Access Modifiers in Java - Public, Private, Protected and Default**. If you have any doubt or any suggestions to make please drop a comment. Thanks!