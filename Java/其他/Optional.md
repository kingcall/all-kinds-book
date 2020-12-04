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