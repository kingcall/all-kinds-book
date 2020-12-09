[TOC]

## Comparable 和 Comparator

###  Comparable 和 Comparator 概论

当你想在集合中存储元素的时候，这个是有**Comparable** 和**Comparator**可以选择，这个时候你到底选择哪一个呢，他们之间的差异在什么地方呢？这篇文章我们讨论一下二者之间的异同，然后说明java 中为什么会同时存在着两个接口

### Comparable interface

首先我们先看一下Comparable 接口的定义

```java
public interface Comparable<T> {
 public int compareTo(T o);
}
```

实现了这个接口的类需要在**compareTo()**  中实现逻辑，把实现了这个接口的类的排序称为自然排序， 实现了的 **compareTo**方法叫做自然比较方法，实现了这个接口的类对象可以被 **Collections.sort** 和 **Arrays.sort** 方法自然的排序，并且这些对象可以在不需要特定的 **comparator**的情况下也可以作为 sorted map  和 sorted set 的keys，

### Comparator interface

首先我们还是先看一下Comparator 接口的定义

```java
@FunctionalInterface
public interface Comparator<T> {
  int compare(T o1, T o2);
}
```

在需要使用Comparator 的场景中，需要被排序的类不需要实现Comparator 接口，Comparator 可以被其他类实现，然后作为一个匿名类或者 lambda  表达式。然后作为参数传给 **Collections.sort** 或者 **Arrays.sort** 排序方法，然后控制排序的实现。

Comparators  也可以用在一些需要排序的数据结构上，例如 sorted sets 和 sorted maps

### Comparable 的列子

接下来让我们通国一个例子来进一步搞清楚什么时候我们需要Comparator 什么时候需要Comparable ，这个例子过后，我们会罗列出 Comparable 和Comparator 之间的不同，例子之后我们很容易就可以理解这之间的不同之处

下面有一个类**Employee** ，这个类的对象我们希望先按照 first name排序，当 first name相同时再按照last name进行排序，所以这个排序对 Employee 这个类而言就是自然排序

**Employee class**

```java
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

```java
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

**输出结果**

```
====================Original List========================
Pyaremohan Mishra 35 E001
John Smith 45 E002
Ram Sharma 23 E003
Pyaremohan Mishra 60 E004
Eva Caroll 32 E005
Ram Tiwari 23 E003
====================Sorted List=======================
Eva Caroll 32 E005
John Smith 45 E002
Pyaremohan Mishra 35 E001
Pyaremohan Mishra 60 E004
Ram Sharma 23 E003
Ram Tiwari 23 E003
```

你需要注意一下几点

- Employee 类实现了Comparable 接口，然后实现了compareTo 方法
- compareTo 实现了排序的逻辑，先按照first name 然后按照 last name 排列，所以对Employee 类而言这是自然排序

- 当你使用 Collections.sort() 来排序类型是Employee 的list 的时候, 排序就自动完成了，因为Employee 已经实现了比较的方法了

### Comparator 的例子

当你不想用上面的排序规则(先按照first name 然后按照 last name 排列)对Employee类的对象排序的时候,例如现在你想当名字相同的时候，按照年龄进行降序排列，这个时候你就不能使用已经在Employee中实现了的**compareTo()** 方法了

这个时候你就可以说，当我们想用任何除了已经类中定义好的自然排序的时候我们就必须使用Comparator，这种情况下，Comparator 就会提供一个对象，这个对象里面封装了某种排序的方式

**Sorting logic**

```java
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

        System.out.println("====================Original List========================");
        for(Employee emp : empList){
            System.out.println("" + emp);
        }
        // Sorting the list
        Collections.sort(empList);

        System.out.println("====================Natural Sorted List=======================");
        for(Employee emp : empList){
            System.out.println("" + emp);
        }

        // Sorting the list
        Collections.sort(empList, new MyComparator());

        System.out.println("====================External Sorted List=======================");
        for(Employee emp : empList){
            System.out.println("" + emp);
        }
    }
}      

class MyComparator implements Comparator<Employee>{
  @Override
  public int compare(Employee o1, Employee o2) {
    int firstCmp = o1.getFirstName().compareTo(o2.getFirstName());
    if(firstCmp == 0){
    int lastCmp = o1.getLastName().compareTo(o2.getLastName());
    if(lastCmp == 0){
       return (o1.getAge() < o2.getAge() ? 1 : (o1.getAge() == o2.getAge() ? 0 : -1));
    }else{
      return lastCmp;
    }        
    }else{
      return firstCmp;
    }        
  }    
}
```

**输出结果**

```
====================Original List========================
Pyaremohan Mishra 35 E001
John Smith 45 E002
Ram Sharma 23 E003
Pyaremohan Mishra 60 E004
Eva Caroll 32 E005
Ram Tiwari 23 E003
====================Natural Sorted List=======================
Eva Caroll 32 E005
John Smith 45 E002
Pyaremohan Mishra 35 E001
Pyaremohan Mishra 60 E004
Ram Sharma 23 E003
Ram Tiwari 23 E003
====================External Sorted List=======================
Eva Caroll 32 E005
John Smith 45 E002
Pyaremohan Mishra 60 E004
Pyaremohan Mishra 35 E001
Ram Sharma 23 E003
Ram Tiwari 23 E003
```

这里我们可以看出，当名字相同的时候，按照年龄降序排列

这里有几点需要你注意

- **MyComparator** 实现了Comparator的compare 方法
- 因为其他的类包含了排序的逻辑，所以我们需要提供了Comparator 的实现了给Collections.sort 方法，例如**Collections.sort(empList, new MyComparator());**
- Comparator 的实现类可以给我们提供了扩展其他排序的能力，因为这个时候你可以选择使用Comparable 的自然排序或者是Comparator 提供的排序
- Comparator 可以使用匿名类或者Lambda 表达式的方式实现

## Comparable Vs Comparator

| 比较项   | Comparable                                                   | Comparator                                                   |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 所在包   | **Comparable** 接口 在 **java.lang** 包下                    | **Comparator** 接口 在 **java.util** 包下                    |
| 核心方法 | **Comparable** interface provides **public int compareTo(T o);** method which needs to be implemented for sorting the elements. This method compares this object with object o and returns an integer, if that integer is - **Positive** - this object is greater than o.**Zero** - this object is equal to o.**Negative** - this object is less than o. | **Comparator** interface provides **int compare(T o1, T o2);** method which needs to be implemented for sorting the elements. Here o1 and o2 objects are compared and an integer value is returned, if that integer is - **Positive**- o1 is greater than o2.**Zero**- o1 is equal to o2.**Negative**- o1 is less than o2. |
| 使用场景 | The class which has to be sorted should implement the **comparable** interface (sorting logic is in the class which has to be sorted), and that implementation **becomes the natural ordering of the class**. | Some other class can implement the **Comparator** interface not the actual class whose objects are to be sorted. That way there may be many comparators and depending on the ordering needed specific comparator can be used. As example for Employee class if different orderings are needed based on first name, last name, age etc. then we can have different comparators with the implementations for those orderings. |
| 使用方法 | When **Comparable** is used to sort the list we can use **Collections.sort(List)**. | When **Comparator** is used to sort the list, Comparator class has to be explicitly specified as a param in the sort method. **Collections.sort(List, Comparator)** |



### 扩展 Comparable和Comparator  配合使用

前面我们学习的都是要不使用Comparable ，要不使用Comparator，我们举的例子是Comparable不能满足我们的需求的时候，我们需要使用其他的排序逻辑，例如前面是的排序逻辑分别是`先根据firstName排序，当firstName相等的时候按照lastName排序`和`先根据姓名排序(firstName+lastName)排序，当姓名相等的时候按照年龄降序排列`，其实我么可以看出这个两个排序完全是逻辑上的不一样。



但是又很多时候我们会面临这样的场景，那就是排序逻辑不变，一会儿根据升序排序，一会根据降序排序，这个时候如果我们的Comparable 中的排序逻辑可以满足上面的排序，就是排序类型(升序还是降序)是不满足的，这个时候我们就可以配合Comparator，来改变原来默认的排序类型(其实就是升序)



#### Talk is cheap Show me the code

下面我们先定义了一个类，这个类按照上面的说法依然实现Comparable

```java
class Student implements Comparable<Student> {
    public String name;
    public int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Student o) {
        return (this.name + this.age).compareTo(o.name + age);
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

然后下面我们写了一个测试代码，从测试的输出结果我们在进行分析

```java
    @Test
    @Order(4)
    public void testSort() {
        List<Student> studentList = new ArrayList<>();
        studentList.add(new Student("b", 2));
        studentList.add(new Student("a", 1));
        System.out.println("====================Original List========================");
        studentList.forEach(System.out::println);

        Collections.sort(studentList);
        System.out.println("====================Natural List========================");
        studentList.forEach(System.out::println);


        Collections.sort(studentList, Comparator.naturalOrder());
        System.out.println("====================naturalOrder List========================");
        studentList.forEach(System.out::println);

        Collections.sort(studentList, Comparator.reverseOrder());
        System.out.println("====================reverseOrder List========================");
        studentList.forEach(System.out::println);
    }
```

下面是输出结果

```
====================Original List========================
Student{name='b', age=2}
Student{name='a', age=1}
====================Natural List========================
Student{name='a', age=1}
Student{name='b', age=2}
====================naturalOrder List========================
Student{name='a', age=1}
Student{name='b', age=2}
====================reverseOrder List========================
Student{name='b', age=2}
Student{name='a', age=1}
```

1. 我们的默认输出是自然排序，也就是当我们调用  Collections.sort(studentList) ，因为这个时候的排序就是使用Comparable 的自然排序
2. 接下来我们传了一个参数，Collections.sort(studentList, Comparator.naturalOrder()),但是我们发现和我们调用自然排序，也就是  Collections.sort(studentList)的结果是一样的，因为Collections.sort(studentList)本身就是自然排序
3. 接下来我们调用了Collections.sort(studentList, Comparator.reverseOrder())，达到了我们的排序的目的，就是自然排序的反向排序

> 所以当我们的自然排序是升序的时候，我们的反向排序就是降序，同理亦是如此

但是这里有一点就是当我们在使用Comparator.naturalOrder()或者Comparator.reverseOrder()的时候，我们的被排序对象依然需要实现Comparable接口，因为naturalOrder和reverseOrder只是决定了**排序的方向**，但是排序的逻辑，依然需要Comparable来实现

#### Talk is cheap Show me the Source code

接下来，我们看一下源码，我们就从`Comparator.naturalOrder()` 开始，

```java
    /**
     * Returns a comparator that compares {@link Comparable} objects in natural order.
     *  返回一个comparator 它使用Comparable的自然排序比较对象
     */
    @SuppressWarnings("unchecked")
    public static <T extends Comparable<? super T>> Comparator<T> naturalOrder() {
        return (Comparator<T>) Comparators.NaturalOrderComparator.INSTANCE;
    }
```

接下来，我们看一下` Comparators.NaturalOrderComparator.INSTANCE;`

```java
    /**
     * Compares {@link Comparable} objects in natural order.
     *
     * @see Comparable
     */
    enum NaturalOrderComparator implements Comparator<Comparable<Object>> {
        INSTANCE;
        
        @Override
        public int compare(Comparable<Object> c1, Comparable<Object> c2) {
            // 其实看到这里我们就明白了，我们看到Comparato的compare方法并没有实现自己的比较逻辑，只是用了参数的compareTo 方法，所以我么可以看到这里就要求被排序对象必须自己实现Comparable 接口
            return c1.compareTo(c2);
        }

        @Override
        public Comparator<Comparable<Object>> reversed() {
            return Comparator.reverseOrder();
        }
    }
```



看了上面的代码，我们就大概知道，这是怎么玩的了，也就是说这个的`Comparator` 本质上还是Comparable，只是改变了排序的方向，如果你不是实现`Comparable` 接口，则报错信息如下

![image-20201209175621765](C:\Users\Wenqliu\AppData\Roaming\Typora\typora-user-images\image-20201209175621765.png)



更多细节的东西大家可以单独看`Comparator` 一节，因为这一节我们主要学习Comparator 和 Comparable  使用场景，不讨论过多细节。

## 总结

1. Comparable 需要被排序的对象实现Comparable 接口，所以可以认为排序的属性是对象本身已经有了的，所以我们将由Comparable 实现的排序叫做**自然排序**

2. 当Comparable 实现的排序不能满足需求的时候，你可以使用Comparator 进行排序，这个往往是因为需要排序的对象的类的定义已经存在的时候(或者说是这个类不是你定义的)

3. 有时候我们可以使用Comparator 提供的两个方法来改变Comparable的排序类型，或者说是排序方向

   

   



