[TOC]

### Optional Class in Java With Examples

Optional class, added in Java 8, provides another way to handle situations when value may or may not be present. Till now you would be using null to indicate that no value is present but it may lead to problems related to null references. This new class **java.util.Optional** introduced in Java 8 can alleviate some of these problems.

**Table of contents**

1. [General structure of Optional class in Java](https://www.netjstech.com/2016/01/optional-class-in-java-8.html#OptionalStruct)
2. [How to create Optional objects](https://www.netjstech.com/2016/01/optional-class-in-java-8.html#CreateOptionalObj)
3. [How to use Optional Values](https://www.netjstech.com/2016/01/optional-class-in-java-8.html#OptionalValues)
4. [How not to use Optional Values](https://www.netjstech.com/2016/01/optional-class-in-java-8.html#OptionalValuesImproper)
5. [Using map and flatMap methods with Optional](https://www.netjstech.com/2016/01/optional-class-in-java-8.html#OptionalmapflatMap)
6. [Optional class methods added in Java 9](https://www.netjstech.com/2016/01/optional-class-in-java-8.html#OptionalMethodJava9)
7. [Optional class method added in Java 10](https://www.netjstech.com/2016/01/optional-class-in-java-8.html#OptionalMethodJava10)



### General structure of Optional class in Java

```
Class Optional<T>
```

Here T is the type of the value stored in the Optional instance. Note that Optional instance **may contain value or it may be empty**, but if it contains value then it is of type T.

Let's see an **example** how null checks may end up being the bulk of our logic. Let's say we have a class Hospital, which has a cancer ward and cancer ward has patients.

So if we need id of a patient following code will do just fine.

```
String id = hospital.getCancerWard().getPatient().getId();
```

But many hospitals don't have a separate cancer ward, what if **null reference** is returned to indicate that there is no cancer ward. That would mean getPatient() will be called on a null reference, which will result in **NullPointerException** at runtime.

In this case you'll have to add null checks to avoid null pointer exception.

```
if(hospital != null){
  Ward cancerWard = hospital.getCancerWard();
  if(cancerWard != null){
    Patient patient = cancerWard.getPatient();
    if(patient != null){
      String id = patient.getId();
    }
  }
}
```

Now let us see how Optional class in Java can help in this case and in many other cases.

### How to create Optional objects

We'll start with creating an Optional object. Optional class in Java doesn't have any constructors, there are several static methods for the purpose of creating Optional objects.

**Creating Optional instance with no value**

If you want to create an Optional instance which doesn't have any value, you can use **empty** method.

```
Optional<String> op = Optional.empty();
```

**Creating an Optional with given non-null value**

If you want to create an Optional instance with a given value, you can use **of** method.

```
Optional<String> op = Optional.of("Hello");
```

Or if you want to create instance of Optional with Ward class (as mentioned above) object

```
Ward ward = new Ward();
Optional<Ward> op = Optional.of(ward);
```

Note that value passed in Optional.of() method should not be null, in case null value is passed NullPointerException will be thrown immediately.

**Creating an Optional using ofNullable method**

There is also **Optional.ofNullable()** method which returns an Optional containing the passed value, if value is non-null, otherwise returns an empty Optional.

```
Patient patient = null;
// Will return empty optional
Optional<Patient> op = Optional.ofNullable(patient);
op.ifPresentOrElse(value->System.out.println("Value- " + value), 
          ()->System.out.println("No Value"));
```

### How to use Optional Values

Usage of Optional is more appropriate in the cases where you need some default action to be taken if there is no value.

Suppose for class Patient, if patient is not null then you return the id otherwise return the default id as 9999.
This can be written typically as-

```
String id = patient != null ? patient.getId() : "9999";
```

Using an Optional object same thing can be written using **orElse()** method which returns the value if present, otherwise returns the default value.

```
Optional<String> op1 = Optional.ofNullable(patient.getId());
String id = op1.orElse("9999");
```

**Getting value using orElseGet() method**

You can use **orElseGet()** which returns a value if value is present otherwise returns the result produced by the supplying function.

**As example-** You may want to get System property if not already there.

```
String country = op.orElseGet(()-> System.getProperty("user.country"));
```

Note that lambda expression is used here to implement a functional interface.

- Refer [Lambda Expressions in Java 8](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) to know more about lambda expressions in Java.
- Refer [Functional Interfaces in Java](https://www.netjstech.com/2015/06/functional-interfaces-and-lambda-expression-in-java-8.html) to know more about functional interface in Java.



**Throwing an exception in absence of value**

You can use **orElseThrow()** method if you want to throw an exception if there is no value.

Note that there are two overloaded orElseThrow() methods-

- **orElseThrow&(Supplier<? extends X> exceptionSupplier)**- If a value is present, returns the value, otherwise throws an exception produced by the exception supplying function.
- **orElseThrow()**- If a value is present, returns the value, otherwise throws NoSuchElementException. This method is added in Java 10.

```
op1.orElseThrow(IllegalStateException::new);
```

Note that here double colon operator (method reference) is used to create exception.

- Refer [Method reference in Java 8](https://www.netjstech.com/2015/06/method-reference-in-java-8.html) to know more about **method reference** in Java.

**Using ifPresent() method**

There are scenarios when you want to execute some logic only if some value is present or do nothing. **ifPresent()** method can be used in such scenarios.

**As example-** You want to add value to the list if there is value present.

```
op1.ifPresent(v->numList.add(v));
```

### How not to use Optional Values

There is a **get()** method provided by Optional class which returns the value, if a value is present in this Optional, otherwise throws **NoSuchElementException**. Using it directly without first ascertaining whether value is there or not is not safer than normally using value without checking for null.

```
Optional<Hospital> op = Optional.of(hospital);
op.get().getCancerWard();
```

Second wrong usage is using both **isPresent** and **get** method which is as good as null checks.

```
if (op.isPresent()){
  op.get().getCancerWard();
}
```

Better way would be to use **map** or **flatMap**.

### Using map and flatMap methods with Optional

**map(Function<? super T,? extends U> mapper)**- If a value is present, returns an Optional containing the result of applying the given mapping function to the value, otherwise returns an empty Optional.

**Optional class map method example**

```
public Optional<String> getPatientId(Patient patient){
  Optional<String> op1 = Optional.of(patient).map(Patient::getId);
  return op1;   
}
```

**Using flatMap method to flatten the structure**

If we get back to the classes mentioned in the beginning; **Hospital**, **Ward** and **Patient** then if we use **Optional** the classes will look like this -

```
public class Hospital {
  private Optional<Ward> cancerWard;

  public Optional<Ward> getCancerWard() {
    return cancerWard;
  }

  public void setCancerWard(Optional<Ward> cancerWard) {
    this.cancerWard = cancerWard;
  }
}
public class Ward {
  private Optional<Patient> patient;

  public Optional<Patient> getPatient() {
    return patient;
  }

  public void setPatient(Optional<Patient> patient) {
    this.patient = patient;
  }   
}
public class Patient {
  private String id;

  public String getId() {
    return id;
  }

  public void setId(String id) {
    this.id = id;
  }
}
```

So now if we have to write this

```
String id = hospital.getCancerWard().getPatient().getId();
```

With Optional then we have to use **flatMap()** method. There is a **map()** method too but writing something like this

```
String id = op.map(Hospital::getCancerWard)
              .map(Ward::getPatient)
              .map(Patient::getId)
              .orElse("9999");
```

Will result in compiler error as the return type of map is Optional<U> and the return type of getCancerWard() is Optional<Ward> so it will make the result of the map of type Optional<Optional<ward>>. That is why you will get compiler error with map method in this case.

Using flatMap will apply the provided Optional-bearing mapping function to it i.e. flatten the 2 level Optional into one.

Thus the correct usage in this chaining is

```
String id = op.flatMap(Hospital::getCancerWard)
              .flatMap(Ward::getPatient)
              .map(Patient::getId)
              .orElse("9999");
```

### Optional class methods added in Java 9

In Java 9 following three methods are added to optional class-

- **stream()**- If a value is present, returns a sequential Stream containing only that value, otherwise returns an empty Stream.
- **or(Supplier<? extends Optional<? extends T>> supplier)**- Returns an Optional describing the value if a value is present, otherwise returns an Optional produced by the supplying function.
- **ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)**- If a value is present, performs the given action with the value, otherwise performs the given empty-based action.

**stream method example**

stream() method in Java Optional class can be used to transform a Stream of optional elements to a Stream of present value elements.

```
Stream<Optional<Patient>> os = getPatient();
Stream<Patient> s = os.flatMap(Optional::stream);
```

**or method example**

In the example Optional<String> and Supplier<Optional<String>> are passed in the or method. Since value is present in op1 so that Optional would be returned.

```
Optional<String> op1 = Optional.of("Test");
Optional<String> op2 = op1.or(()->Optional.of("Default"));
```

**ifPresentOrElse method example**

In the ifPresentOrElse method example of the Optional class, Consumer action is there as the first argument which consumes the value of the Optional, in case optional is empty runnable task is executed which is passed as the second argument.

```
Optional<String> op1 = Optional.of("Test");
op1.ifPresentOrElse((s)->System.out.println("Value in Optional- " + s), 
        ()->System.out.println("No value in Optional"));
```

### Optional class method added in Java 10

In Java 10 following method is added to optional class-

- **orElseThrow()**- If a value is present, returns the value, otherwise throws NoSuchElementException.

That's all for this topic **Optional Class in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!

### 一、使用Optional引言

#### 1.1、代码问题引出

在写程序的时候一般都遇到过 `NullPointerException`，所以经常会对程序进行非空的判断：

```java
User user = getUserById(id);
if (user != null) {
    String username = user.getUsername();
    System.out.println("Username is: " + username); // 使用 username
}
```

为了解决这种尴尬的处境，JDK 终于在 Java8 的时候加入了 `Optional` 类，查看 `Optional` 的 javadoc 介绍：

```java
A container object which may or may not contain a non-null value. If a value is present, isPresent() will return true and get() will return the value.
```

这是一个可以包含或者不包含非 `null` 值的容器。如果值存在则 `isPresent()`方法会返回 `true`，调用 `get()` 方法会返回该对象。

#### 1.2、解决进阶

我们假设 `getUserById` 已经是个客观存在的不能改变的方法，那么利用 `isPresent` 和 `get` 两个方法，我们现在能写出下面的代码：

```java
Optional<User> user = Optional.ofNullable(getUserById(id));
if (user.isPresent()) {
    String username = user.get().getUsername();
    System.out.println("Username is: " + username); // 使用 username
}
```

好像看着代码是优美了点,但是事实上这与之前判断 `null` 值的代码没有本质的区别，反而用 `Optional` 去封装 *value*，增加了代码量。所以我们来看看 `Optional` 还提供了哪些方法，让我们更好的（以正确的姿势）使用 `Optional`。

### 二、Optional三个静态构造方法

**1）概述：**

JDK 提供三个静态方法来构造一个 `Optional`：

1. Optional.of(T value)

   ```java
       public static <T> Optional<T> of(T value) {
           return new Optional<>(value);
       }
   ```

   该方法通过一个非 `null` 的 *value* 来构造一个 `Optional`，返回的 `Optional` 包含了 *value* 这个值。对于该方法，传入的参数一定不能为 `null`，否则便会抛出 `NullPointerException`。

2. Optional.ofNullable(T value)

   ```java
       public static <T> Optional<T> ofNullable(T value) {
           return value == null ? empty() : of(value);
       }
   ```

   该方法和 `of` 方法的区别在于，传入的参数可以为 `null` —— 但是前面 javadoc 不是说 `Optional` 只能包含非 `null` 值吗？我们可以看看 `ofNullable` 方法的源码。

   原来该方法会判断传入的参数是否为 `null`，如果为 `null` 的话，返回的就是 `Optional.empty()`。

3. Optional.empty()

   ```java
       public static<T> Optional<T> empty() {
           @SuppressWarnings("unchecked")
           Optional<T> t = (Optional<T>) EMPTY;
           return t;
       }
   ```

   该方法用来构造一个空的 `Optional`，即该 `Optional` 中不包含值 —— 其实底层实现还是 **如果 Optional 中的 value 为 null 则该 Optional 为不包含值的状态**，然后在 API 层面将 `Optional` 表现的不能包含 `null`值，使得 `Optional` 只存在 **包含值** 和 **不包含值** 两种状态。

**2）分析：**

前面 javadoc 也有提到，`Optional` 的 `isPresent()` 方法用来判断是否包含值，`get()` 用来获取 `Optional` 包含的值 —— **值得注意的是，如果值不存在，即在一个Optional.empty 上调用 get() 方法的话，将会抛出 NoSuchElementException异常**。

**3）总结：**

1）Optional.of(obj): 它要求传入的 obj 不能是 null 值的, 否则还没开始进入角色就倒在了 NullPointerException 异常上了.
2）Optional.ofNullable(obj): 它以一种智能的, 宽容的方式来构造一个 Optional 实例. 来者不拒, 传 null 进到就得到 Optional.empty(), 非 null 就调用 Optional.of(obj).
那是不是我们只要用 Optional.ofNullable(obj) 一劳永逸, 以不变应二变的方式来构造 Optional 实例就行了呢? 那也未必, 否则 Optional.of(obj) 何必如此暴露呢, 私有则可。

### 三、Optional常用方法详解

#### 3.1、Optional常用方法概述

1. Optional.of(T t)

   将指定值用 Optional 封装之后返回，如果该值为 null，则抛出一个 NullPointerException 异常。

2. Optional.empty()

   创建一个空的 Optional 实例。

3. Optional.ofNullable(T t)

   将指定值用 Optional 封装之后返回，如果该值为 null，则返回一个空的 Optional 对象。

4. isPresent

   如果值存在返回true,否则返回false

5. ifPresent

   如果Optional实例有值则为其调用consumer ,否则不做处理。
   要理解ifPresent方法，首先需要了解Consumer类。简答地说，Consumer类包含一个抽象方法。该抽象方法对传入的值进行处理，但没有返回值。 Java8支持不用接口直接通过lambda表达式传入参数。
   如果Optional实例有值，调用ifPresent()可以接受接口段或lambda表达式。

6. Optional.get()

   如果该值存在，将该值用 Optional 封装返回，否则抛出一个 NoSuchElementException 异常。

7. orElse(T t)

   如果调用对象包含值，返回该值，否则返回t。

8. orElseGet(Supplier s)

   如果调用对象包含值，返回该值，否则返回 s 获取的值。

9. orElseThrow()

   它会在对象为空的时候抛出异常。

10. map(Function f)

    如果值存在，就对该值执行提供的 mapping 函数调用。

11. flatMap(Function mapper)

    如果值存在，就对该值执行提供的mapping 函数调用，返回一个 Optional 类型的值，否则就返回一个空的 Optional 对象。

#### 3.2、Optional常用方法详解

##### 3.2.1、ifPresent

```java
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
```

如果 `Optional` 中有值，则对该值调用 `consumer.accept`，否则什么也不做。
所以对于引言上的例子，我们可以修改为：

```java
Optional<User> user = Optional.ofNullable(getUserById(id));
user.ifPresent(u -> System.out.println("Username is: " + u.getUsername()));
```

##### 3.2.2、orElse

```java
    public T orElse(T other) {
        return value != null ? value : other;
    }
```

如果 `Optional` 中有值则将其返回，否则返回 `orElse` 方法传入的参数。

```java
User user = Optional
        .ofNullable(getUserById(id))
        .orElse(new User(0, "Unknown"));
        
System.out.println("Username is: " + user.getUsername());
```

##### 3.2.3、orElseGet

```java
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
```

`orElseGet` 与 `orElse` 方法的区别在于，`orElseGet` 方法传入的参数为一个 `Supplier`接口的实现 —— 当 `Optional` 中有值的时候，返回值；当 `Optional` 中没有值的时候，返回从该 `Supplier` 获得的值。

```java
User user = Optional
        .ofNullable(getUserById(id))
        .orElseGet(() -> new User(0, "Unknown"));
        
System.out.println("Username is: " + user.getUsername());
```

##### 3.2.4、orElseThrow

```java
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }
```

`orElseThrow` 与 `orElse` 方法的区别在于，`orElseThrow` 方法当 `Optional` 中有值的时候，返回值；没有值的时候会抛出异常，抛出的异常由传入的 *exceptionSupplier* 提供。

举例说明：

 在 SpringMVC 的控制器中，我们可以配置统一处理各种异常。查询某个实体时，如果数据库中有对应的记录便返回该记录，否则就可以抛出 `EntityNotFoundException` ，处理 `EntityNotFoundException` 的方法中我们就给客户端返回Http 状态码 404 和异常对应的信息 —— `orElseThrow` 完美的适用于这种场景。

```java
@RequestMapping("/{id}")
public User getUser(@PathVariable Integer id) {
    Optional<User> user = userService.getUserById(id);
    return user.orElseThrow(() -> new EntityNotFoundException("id 为 " + id + " 的用户不存在"));
}

@ExceptionHandler(EntityNotFoundException.class)
public ResponseEntity<String> handleException(EntityNotFoundException ex) {
    return new ResponseEntity<>(ex.getMessage(), HttpStatus.NOT_FOUND);
}
```

##### 3.2.5、map

```java
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```

如果当前 `Optional` 为 `Optional.empty`，则依旧返回 `Optional.empty`；否则返回一个新的 `Optional`，该 `Optional` 包含的是：函数 *mapper* 在以 *value* 作为输入时的输出值。

```java
String username = Optional.ofNullable(getUserById(id))
                        .map(user -> user.getUsername())
                        .orElse("Unknown")
                        .ifPresent(name -> System.out.println("Username is: " + name));
```

而且我们可以多次使用 `map` 操作：

```java
Optional<String> username = Optional.ofNullable(getUserById(id))
                                .map(user -> user.getUsername())
                                .map(name -> name.toLowerCase())
                                .map(name -> name.replace('_', ' '))
                                .orElse("Unknown")
                                .ifPresent(name -> System.out.println("Username is: " + name));
```

##### 3.2.6、flatMap

```java
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }
```

`flatMap` 方法与 `map` 方法的区别在于，`map` 方法参数中的函数 `mapper` 输出的是值，然后 `map` 方法会使用 `Optional.ofNullable` 将其包装为 `Optional`；而 `flatMap` 要求参数中的函数 `mapper` 输出的就是 `Optional`。

```java
Optional<String> username = Optional.ofNullable(getUserById(id))
                                .flatMap(user -> Optional.of(user.getUsername()))
                                .flatMap(name -> Optional.of(name.toLowerCase()))
                                .orElse("Unknown")
                                .ifPresent(name -> System.out.println("Username is: " + name));
```

##### 3.2.7、filter

```java
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }
```

`filter` 方法接受一个 `Predicate` 来对 `Optional` 中包含的值进行过滤，如果包含的值满足条件，那么还是返回这个 `Optional`；否则返回 `Optional.empty`。

```java
Optional<String> username = Optional.ofNullable(getUserById(id))
                                .filter(user -> user.getId() < 10)
                                .map(user -> user.getUsername());
                                .orElse("Unknown")
                                .ifPresent(name -> System.out.println("Username is: " + name));
```

### 四、Optional使用示例

#### 4.1、使用展示一

当 user.isPresent() 为真, 获得它关联的 orders的映射集合, 为假则返回一个空集合时, 我们用上面的 orElse, orElseGet 方法都乏力时, 那原本就是 map 函数的责任, 我们可以这样一行：

```java
return user.map(u -> u.getOrders()).orElse(Collections.emptyList())
 
//上面避免了我们类似 Java 8 之前的做法
if(user.isPresent()) {
  return user.get().getOrders();
} else {
  return Collections.emptyList();
}
```

map 是可能无限级联的, 比如再深一层, 获得用户名的大写形式：

```java
return user.map(u -> u.getUsername())
           .map(name -> name.toUpperCase())
           .orElse(null);
```

以前的做法：

```java
User user = .....
if(user != null) {
  String name = user.getUsername();
  if(name != null) {
    return name.toUpperCase();
  } else {
    return null;
  }
} else {
  return null;
}
```

filter() :如果有值并且满足条件返回包含该值的Optional，否则返回空Optional。

```java
Optional<String> longName = name.filter((value) -> value.length() > 6);  
System.out.println(longName.orElse("The name is less than 6 characters")); 
```

