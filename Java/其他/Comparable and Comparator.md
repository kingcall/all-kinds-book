### Difference Between Comparable and Comparator in Java

While sorting elements in collection classes, these two interfaces **Comparable** and **Comparator** in Java come into picture. Both of these interfaces are used to sort collection elements. An obvious question which comes to mind is *why two different interfaces?*

In this post we'll see the **difference between Comparable and Comparator interfaces in Java** and why both of them are required. Before going into the differences between these two, let's have a brief introduction of both.

### Comparable interface in Java

Comparable interface in Java is defined as follows-

```
public interface Comparable<T> {
 public int compareTo(T o);
}
```

Classes implementing this interface need to provide sorting logic in **compareTo()** method.

The ordering imposed by the implementation of this [interface](https://www.netjstech.com/2015/05/interface-in-java.html) is referred to as the class' natural ordering, and the implemented **compareTo** method is referred to as its natural comparison method.
Classes implementing this interface can be sorted automatically by **Collections.sort** (and Arrays.sort) method. Objects that implement this interface can be used as keys in a sorted map (like [TreeMap](https://www.netjstech.com/2015/11/treemap-in-java.html target=)) or as elements in a sorted set (like [TreeSet](https://www.netjstech.com/2015/10/treeset-in-java.html target=)) without the need to specify a **comparator**.

### Comparator interface in Java

In the case of Comparator interface, [class](https://www.netjstech.com/2015/04/class-in-java.html) that is to be sorted **doesn't implement it**. Comparator can be implemented by some other class, as an anonymous class or as [lambda expression](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) (from Java 8). Comparators can be passed to a sort method (such as **Collections.sort** or **Arrays.sort**) to allow precise control over the sort order. Comparators can also be used to control the order of certain data structures (such as sorted sets or sorted maps).

**General form of Comparator interface in Java-**

```
@FunctionalInterface
public interface Comparator<T> {
  int compare(T o1, T o2);
}
```

Note the annotation **@FunctionalInterface**, it's available from Java 8 and Comparator being a functional interface can be implemented using lambda expression.

- Refer [Functional interface annotation in Java 8](https://www.netjstech.com/2015/06/functional-interface-annotation-java-8.html) to know more about **Functional interface annotation**.

### Java Example code using Comparable

Let's see an example to further clarify when do we need Comparable and when Comparator interface is needed in Java. After the example we'll list out the differences between Comparable and Comparator in Java as it will be easy to understand the differences after the example.

Let's say we have an **Employee** class. Objects of this Employee class are stored in an ArrayList and we want to sort it first on first name and then on last name. So this order is the natural order for the Employee class, employees are first sorted on first name and then last name.

**Employee class**

```
public class Employee implements Comparable<Employee>  {
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
  
  @Override
  public String toString() {        
    return getFirstName() + " " + getLastName() + " " + getAge() + " " + getEmpId();
  }
  @Override
  public int compareTo(Employee o) {
    int firstCmp = this.getFirstName().compareTo(o.getFirstName());
    return firstCmp != 0 ? firstCmp :  this.getLastName().compareTo(o.getLastName());
  }
}
```

**Class where sorting of the list will be done**

```
public class SortObjectList {
  public static void main(String[] args) {
    List<Employee> empList = new ArrayList<Employee>();
    // Storing elements in the arraylist
    empList.add(getData("E001", "Mishra", "Pyaremohan", 35));
    empList.add(getData("E002", "Smith", "John", 45));
    empList.add(getData("E003", "Sharma", "Ram", 23));
    empList.add(getData("E004", "Mishra", "Pyaremohan", 60));
    empList.add(getData("E005", "Caroll", "Eva", 32));
    empList.add(getData("E003", "Tiwari", "Ram", 23));
      
    System.out.println("Original List");
    for(Employee emp : empList){
      System.out.println("" + emp);
    }
    // Sorting the list
    Collections.sort(empList);
    
    System.out.println("Sorted List");
    for(Employee emp : empList){
      System.out.println("" + emp);
    }    
  }
              
  // Stub method 
  private static Employee getData(String empId, String lastName, String firstName, int age){
    Employee employee = new Employee();
    employee.setEmpId(empId);
    employee.setLastName(lastName);
    employee.setFirstName(firstName);
    employee.setAge(age);
    return employee;
  }    
}
```

**Output**

```
Original List
Pyaremohan Mishra 35 E001
John Smith 45 E002
Ram Sharma 23 E003
Pyaremohan Mishra 60 E004
Eva Caroll 32 E005
Ram Tiwari 23 E003
Sorted List
Eva Caroll 32 E005
John Smith 45 E002
Pyaremohan Mishra 35 E001
Pyaremohan Mishra 60 E004
Ram Sharma 23 E003
Ram Tiwari 23 E003
```

Some of the points you should notice here are-

- Here note that Employee class is implementing Comparable interface providing implementation for the **compareTo()** method.
- compareTo() method is implemented to sort on first name and then on last name so that is the natural ordering for the Employee class.
- When you are using Collections.sort() method to sort the List having objects of type Employee, sorting is done automatically as comparison order is already defined for that class.

### Java Example code using Comparator

Now if you want to sort employees using following rule-

If names are same, they are sorted on the basis of age in descending order.

With this sorting order we can't use the already implemented **compareTo()** method of the Employee class. So you can say that if we want any ordering other than the defined natural ordering for the class then we have to use Comparator. In that case Comparator will provide an object that [encapsulates](https://www.netjstech.com/2015/04/encapsulation-in-java.html) an ordering.

**Sorting logic**

```
public class SortObjectList {
  public static void main(String[] args) {
    List<Employee> empList = new ArrayList<Employee>();
    // Storing elements in the arraylist
    empList.add(getData("E001", "Mishra", "Pyaremohan", 35));
    empList.add(getData("E002", "Smith", "John", 45));
    empList.add(getData("E003", "Sharma", "Ram", 23));
    empList.add(getData("E004", "Mishra", "Pyaremohan", 60));
    empList.add(getData("E005", "Caroll", "Eva", 32));
    empList.add(getData("E003", "Tiwari", "Ram", 23));
    
    System.out.println("Original List");
    for(Employee emp : empList){
      System.out.println("" + emp);
    }
    // Sorting the list
    Collections.sort(empList, new MyComparator());
            
    System.out.println("Sorted List");
    for(Employee emp : empList){
      System.out.println("" + emp);
    }    
  }
                
  // Stub method 
  private static Employee getData(String empId, String lastName, String firstName, int age){
    Employee employee = new Employee();
    employee.setEmpId(empId);
    employee.setLastName(lastName);
    employee.setFirstName(firstName);
    employee.setAge(age);
    return employee;
  }        
}

class MyComparator implements Comparator<Employee>{
  @Override
  public int compare(Employee o1, Employee o2) {
    int firstCmp = o1.getFirstName().compareTo(o2.getFirstName());
    if(firstCmp == 0){
    int lastCmp = o1.getLastName().compareTo(o2.getLastName());
    if(lastCmp == 0){
      return (o2.getAge() < o1.getAge() ? -1 :
             (o2.getAge() == o1.getAge() ? 0 : 1));
    }else{
      return lastCmp;
    }        
    }else{
      return firstCmp;
    }        
  }    
}
```

**Output**

```
Original List
Pyaremohan Mishra 35 E001
John Smith 45 E002
Ram Sharma 23 E003
Pyaremohan Mishra 60 E004
Eva Caroll 32 E005
Ram Tiwari 23 E003
Sorted List
Eva Caroll 32 E005
John Smith 45 E002
Pyaremohan Mishra 60 E004
Pyaremohan Mishra 35 E001
Ram Sharma 23 E003
Ram Tiwari 23 E003
```

Here it can be seen that the name which is same is sorted by age in descending order.

Some of the points you should notice here are-

- A Comparator class **MyComparator** is implementing the Comparator interface and providing implementation for the **compare()** method.
- Since another class is used for providing the comparison logic, now we need to provide the Comparator class in Collections.sort method.
  **Collections.sort(empList, new MyComparator());**
- Having another class implementing the Comparator interface is giving us flexibility to define different sort orderings. You can still use the natural ordering as defined by Comparable interface or you can use the sorting logic implemented by Comparator class.

Comparator class can also be implemented as an Anonymous class or as a Lambda expression.

- See an example of how comparator can be implemented as a lambda expression here - [Lambda expression examples in Java 8](https://www.netjstech.com/2015/06/lambda-expression-examples-in-java-8.html)

I hope now it is clear why two interfaces are provided for sorting and how they are used. So let's see the differences now-

### Comparable Vs Comparator in Java

| Comparable                                                   | Comparator                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Comparable** interface is in **java.lang** package.        | **Comparator** interface is in **java.util** package.        |
| **Comparable** interface provides **public int compareTo(T o);** method which needs to be implemented for sorting the elements. This method compares this object with object o and returns an integer, if that integer is - **Positive** - this object is greater than o.**Zero** - this object is equal to o.**Negative** - this object is less than o. | **Comparator** interface provides **int compare(T o1, T o2);** method which needs to be implemented for sorting the elements. Here o1 and o2 objects are compared and an integer value is returned, if that integer is - **Positive**- o1 is greater than o2.**Zero**- o1 is equal to o2.**Negative**- o1 is less than o2. |
| The class which has to be sorted should implement the **comparable** interface (sorting logic is in the class which has to be sorted), and that implementation becomes the natural ordering of the class. | Some other class can implement the **Comparator** interface not the actual class whose objects are to be sorted. That way there may be many comparators and depending on the ordering needed specific comparator can be used. As example for Employee class if different orderings are needed based on first name, last name, age etc. then we can have different comparators with the implementations for those orderings. |
| When **Comparable** is used to sort the list we can use **Collections.sort(List)**. | When **Comparator** is used to sort the list, Comparator class has to be explicitly specified as a param in the sort method. **Collections.sort(List, Comparator)** |

That's all for this topic **Difference Between Comparable and Comparator in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!