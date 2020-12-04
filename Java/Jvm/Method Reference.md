### Method Reference in Java

We generally use [lambda expressions](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) to create anonymous methods but sometimes lambda expressions are also used to call an existing method. There is another feature provided in Java 8 related to lambda expression called **method reference** in Java that provides a clearer alternative to refer to the existing method by name.

Important thing to note about Java method references is that they provide a way to refer to a method, **they don't execute the method**.

Method reference in Java relates to lambda expression as they **also require a target type context** and at the time of execution they also create an instance of [functional interface](https://www.netjstech.com/2015/06/functional-interfaces-and-lambda-expression-in-java-8.html).

How lambda expressions and method references **differ is** where lambda expressions let us define anonymous methods which can be used as an instance of functional interfaces. Method references do the same thing, but with existing methods.

**Table of contents**

1. [Kinds of Method References in Java](https://www.netjstech.com/2015/06/method-reference-in-java-8.html#MethodRefKinds)
2. [Method References to static methods](https://www.netjstech.com/2015/06/method-reference-in-java-8.html#MethodRefStaticMethods)
3. [Method References to Instance Methods](https://www.netjstech.com/2015/06/method-reference-in-java-8.html#MethodRefInstanceMethods)
4. [Reference to an instance method of an arbitrary object of a particular type](https://www.netjstech.com/2015/06/method-reference-in-java-8.html#MethodRefArbitraryObj)
5. [Reference to a Constructor](https://www.netjstech.com/2015/06/method-reference-in-java-8.html#MethodRefConstructor)



### Kinds of Method References in Java

There are four kinds of method references in Java.

| Kind                                                         | Example                              | Syntax                        |
| ------------------------------------------------------------ | ------------------------------------ | ----------------------------- |
| Reference to a static method                                 | ContainingClass::staticMethodName    | ClassName::methodName         |
| Reference to an instance method of a particular object       | containingObject::instanceMethodName | objRef::methodName            |
| Reference to an instance method of an arbitrary object of a particular type | ContainingType::methodName           | ClassName::instanceMethodName |
| Reference to a constructor                                   | ClassName::new                       | classname::new                |

Notice that the class name is separated from the method name by a double colon. The **::** is a new separator (known as **double colon operator**) that has been added in Java 8.

### Method References to static methods

The following example code shows a static method reference in Java. In the example there is a functional interface **IMyStringFunc** with a method **stringFunc**.

```
import java.util.Arrays;

@FunctionalInterface
interface IMyStringFunc<T, R>{
  R stringFunc(T t);
}
public class MethodRefDemo {
  public static void main(String[] args) {
    //String array of names
    String[] strNames = new String[]{"Ram", "shyam", "Ramesh", "John", "brad", 
           "Suresh"};
    // method reference to the static method sortName
    IMyStringFunc<String[],String[]> stringFunc = SortClass::sortName;
    // Here calling strngFunc will refer to the implementing method
    // sortName()
    String[] sortedNames = stringFunc.stringFunc(strNames);
    for(String name : sortedNames){
      System.out.println(name);
    }
  }
}

class SortClass{
  // A static method that sorts an array
  static String[] sortName(String[] names) {
    //Sorting array using sort method (case sensitive)
    Arrays.sort(names);
    return names;
  }
}
```

**Output**

```
John
Ram
Ramesh
Suresh
brad
shyam
```

In the code in this line ***SortClass::sortName;\*** existing method sortName() is referred using method reference. This works because sortName is compatible with the IMyStringFunc functional interface. Thus, the expression **SortClass::sortName** evaluates to a reference to an object in which method **sortName** provides the implementation of abstract method stringFunc in functional interface **IMyStringFunc**.

### Method References to Instance Methods

To pass a reference to an instance method of a particular object is similar to static method reference, instead of class we need to use object

```
objRef::methodName
```

Let's see the same example again with object

```
import java.util.Arrays;
@FunctionalInterface
interface IMyStringFunc<T, R>{
  R stringFunc(T t);
}

public class MethodRefDemo {
  public static void main(String[] args) {
    // creating an object
    SortClass sc = new SortClass();
    //String array of names
    String[] strNames = new String[]{"Ram", "shyam", "Ramesh", "John", "brad", 
            "Suresh"};  
    // method reference to the instance method sortName
    IMyStringFunc<String[],String[]> refObj = sc::sortName;
    String[] sortedNames = refObj.stringFunc(strNames);
    for(String name : sortedNames){
      System.out.println(name);
    }
  }
}

class SortClass{
  // A non-static method that sorts an array
  String[] sortName(String[] names) {
    //Sorting array using sort method (case sensitive)
    Arrays.sort(names);
    return names;
  }
}
```

### Reference to an instance method of an arbitrary object of a particular type

There may be a situation when you want to specify an instance method that can be used with any object of a given class without any particular object. In that case name of the class is used even when a non-static method is specified.

```
ClassName::instanceMethodName
```

Let's take one example where we have a Person class and we have a list of Person object. We want to call getFirstName() method on all those person objects.

**Person Class**

```
public class Person {
  private String firstName;
  private String lastName;
  private int age;
  private char gender;
  public Person(String firstName, String lastName, int age, char gender){
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;
    this.gender = gender;
  }
    
  public String getFirstName() {
    return firstName;
  }

  public String getLastName() {
    return lastName;
  }

  public int getAge() {
    return age;
  }
  public char getGender() {
    return gender;
  }
    
  public String toString(){
    StringBuffer sb = new StringBuffer();
    sb.append(getFirstName()).append(" ");
    sb.append(getLastName()).append(" ");
    sb.append(getAge()).append(" ");
    sb.append(getGender());
    return sb.toString();      
  }
}
@FunctionalInterface
interface IMyStringFunc<T, R>{
  R stringFunc(T t);
}

public class MethodRefDemo {
  public static void main(String[] args) {
    List<Person> personList = createList();
    List allNames = MethodRefDemo.listAllNames (personList, Person::getFirstName);
    System.out.println("" + allNames);
  }
    
  //Utitlity method to create list
  private static List<Person> createList(){
    List<Person> tempList = new ArrayList<Person>();
    IMyFunc createObj = Person::new;
    Person person = createObj.getRef("Ram","Tiwari", 50, 'M');
    tempList.add(person);
    person = createObj.getRef("Prem", "Chopra", 13, 'M');
    tempList.add(person);
    person = createObj.getRef("Tanuja", "Trivedi", 30, 'F');
    tempList.add(person);
    person = createObj.getRef("Manoj", "Sharma", 40, 'M');
    tempList.add(person);
    person = createObj.getRef("John", "Trevor", 70, 'M');
    tempList.add(person);
    person = createObj.getRef("Alicia", "Sliver", 17, 'F');
    tempList.add(person);
    System.out.println("List elements are - ");
    System.out.println(tempList);
    return tempList;
  }

  private static List<String> listAllNames(List<Person> person, 
         IMyStringFunc<Person, String> func){
    List<String> result = new ArrayList<String>();
    person.forEach(x -> result.add(func.stringFunc(x)));
    return result;
  }
}
```

Notice this line **Person::getFirstName** here we are calling getFirstName method on all the objects of the list not any one particular object.

### Reference to a Constructor

A constructor can be referenced in the same way as a static method by using new.

```
public class Person {
  private String firstName;
  private String lastName;
  private int age;
  private char gender;
  public Person(String firstName, String lastName, int age, char gender){
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;
    this.gender = gender;
  }
    
  public String getFirstName() {
    return firstName;
  }

  public String getLastName() {
    return lastName;
  }

  public int getAge() {
    return age;
  }
  public char getGender() {
    return gender;
  }
    
  public String toString(){
    StringBuffer sb = new StringBuffer();
    sb.append(getFirstName()).append(" ");
    sb.append(getLastName()).append(" ");
    sb.append(getAge()).append(" ");
    sb.append(getGender());
    return sb.toString();
  }
}
@FunctionalInterface
interface IMyFunc {
  Person getRef(String firstName, String lastName, int age, char gender);
}

public class LambdaDemo {
  public static void main(String[] args) {
    List<Person> personList = createList();
    System.out.println("Name - " + personList.get(0).getFirstName());
  }
  // Utitlity method to create list
  private static List<Person> createList(){
    List<Person> tempList = new ArrayList<Person>();
    IMyFunc createObj = Person::new;
    Person person = createObj.getRef("Ram","Tiwari", 50, 'M');
    tempList.add(person);
    person = createObj.getRef("Prem", "Chopra", 13, 'M');
    tempList.add(person);
    person = createObj.getRef("Tanuja", "Trivedi", 30, 'F');
    tempList.add(person);
    person = createObj.getRef("Manoj", "Sharma", 40, 'M');
    tempList.add(person);
    person = createObj.getRef("John", "Trevor", 70, 'M');
    tempList.add(person);
    person = createObj.getRef("Alicia", "Sliver", 17, 'F');
    tempList.add(person);
    System.out.println("List elements are - ");
    System.out.println(tempList);
    return tempList;
  }
}
```

**Output**

```
List elements are - 
[Ram Tiwari 50 M, Prem Chopra 13 M, Tanuja Trivedi 30 F, Manoj Sharma 40 M, John Trevor 70 M, Alicia Sliver 17 F]
Name - Ram
```

In the program notice that the getRef method of the IMyFunc returns a reference of type Person and has **String, String, int and char** as parameters. Also notice that in the Person class there is a constructor which specifies **String, String, int and char** as parameters.

Using this line **Person::new** a constructor reference to a Person constructor is created. Functional interface method also takes the same params thus the constructor of the Person class is referred through it.

That's all for this topic **Method Reference in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!