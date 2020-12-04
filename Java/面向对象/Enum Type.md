### Enum Type in Java

An enum type in Java is a special data type that helps you to define a list of predefined constants which can be accessed using a variable. In the Java programming language, you define an enum type by using the enum keyword.

As example, if you want to declare a enum **Day** that has all the days as predefined constants.

```
enum Day {
    SUNDAY, MONDAY, TUESDAY, WEDNESDAY,
    THURSDAY, FRIDAY, SATURDAY 
}
```

Since enum is a data type so you can declare a variable of that type.

```
Day d;
d = Day.FRIDAY;
System.out.println("Day of the week " + d);
```

**Output**

```
Day of the week FRIDAY
```

**Note:** You **do not** instantiate an enum using new operator, even though enumeration defines a class type. You declare an enumeration variable as you will declare a varibale of [primitive data types](https://www.netjstech.com/2017/03/primitive-data-types-in-java.html).

Enum was added in Java 1.5 along with [generics](https://www.netjstech.com/2017/01/generics-in-java.html), [varargs](https://www.netjstech.com/2015/05/varargs-in-java.html), autoboxing and other features.

### Java Enum Example

Here we have a complete code where enum is created for days of the week and then we loop through that enum values and print all the constants defined in the enum.

```
enum Day {
    SUNDAY, MONDAY, TUESDAY, WEDNESDAY,
    THURSDAY, FRIDAY, SATURDAY 
}

public class EnumDemo {

 public static void main(String[] args) {
   Day[] allDays = Day.values();
   for(Day day : allDays){
     System.out.println(day);
   }
 }
}
```

### Benefits & Usage of enum in Java

Apart from the obvious benefit of using enum when you need to represent a fixed set of constants there are other benefits and usages of enum.

1. Provides type safety

   – If you use private static final constants you can always assign any random value to a variable instead of these predefined constants.

   **As example**– If I have constants like below I can still assign dayOfTheWeek as “Funday”.

   ```
   private static final String SUNDAY = "Sunday";
   private static final String MONDAY = "Monday";
   private static final String TUESDAY = "Tuesday";
      
   final String dayOfTheWeek;
   dayOfTheWeek = "Funday";
   ```

   With enums it is not possible. If you have an enum like this -

   ```
   enum Day {
       SUNDAY, MONDAY, TUESDAY, WEDNESDAY,
       THURSDAY, FRIDAY, SATURDAY 
   }
   ```

   How much ever you want your day to be a funday you can’t do it now.

   ```
   Day day = Day.FUNDAY; // Compile-time error
   ```

2. **Enums have their own name space**– Enums in Java have their own name space. So a enum Day as created above will be created in its own name space and has its own type. You are not permitted to use any value other than the one specified as predefined constants with in the enum.

3. Comparison using == operator

   \- Two enumeration constants can be compared for equality by using the

    

   = =

    

   relational operator.

    

   As example

   \- This statement compares the value in day variable with the TUESDAY constant:

   ```
   Day day = Day.FRIDAY;
      
   if(day == Day.TUESDAY){
    System.out.println("Days are equal");
   }else{
    System.out.println("Days are not equal");
   }
   ```

4. Used in switch statement

   \- Java Enum can also be used in a

    

   switch statement

   . All case statements have to use the constants from the enum that is used in the switch statement.

   **As example-**

   ```
   Day day = Day.FRIDAY;
    
    switch(day){
     case MONDAY: 
      System.out.println("Week started");
      break;
     case TUESDAY: case WEDNESDAY: case THURSDAY:
      System.out.println("Keep toiling");
      break;
     case FRIDAY:
      System.out.println("Countdown begins");
      break;
     case SATURDAY: case SUNDAY:
      System.out.println("Weekend !!!");
      break;
   }
   ```

### enum declaration defines a class

Java enumeration is a class type, enum can have [constructors](https://www.netjstech.com/2015/04/constructor-chaining-in-java-calling-one-constructor-from-another.html), instance variables, methods. Enum can even implement [interfaces](https://www.netjstech.com/2015/05/marker-interface-in-java.html).

**As example**– Here I have created an enum **Days** which defines constants, a constructor, a variable day, a method getDay() and even [main method](https://www.netjstech.com/2015/04/why-main-method-is-static-in-java.html) so you can run this enum as any other Java class.

```
public enum Days {
 SUNDAY(1), 
 MONDAY(2), 
 TUESDAY(3), 
 WEDNESDAY(4),
 THURSDAY(5), 
 FRIDAY(6), 
 SATURDAY(7);
 private int  day;
 private Days(int day){
  this.day = day;
 }
 
 int getDay(){
   return day;
 }
 public static void main(String[] args) {
  Days[] allDays = Days.values();
  for(Days d : allDays){
   System.out.println(d + " " + d.getDay());
  }
 }
}
```

**Output**

```
SUNDAY 1
MONDAY 2
TUESDAY 3
WEDNESDAY 4
THURSDAY 5
FRIDAY 6
SATURDAY 7
```

Some important points to keep in mind while creating an Enum in Java are as given below.

1. Java requires that the constants be defined first, prior to any fields or methods. So you can’t have code like below where field day is declared first, as it will result in error.

   ```
   public enum Days {
    private int day;
    SUNDAY(1), 
    MONDAY(2), 
    TUESDAY(3), 
    WEDNESDAY(4),
    THURSDAY(5), 
    FRIDAY(6), 
    SATURDAY(7);
    ...
    ...
   }
   ```

2. Each enumeration constant is an object of its enumeration type. When you create an object of any class its constructor is called for initialization same way constructor is called when each enumeration constant is created.

   If you have noticed constructor of the Days enum takes one int parameter and all the constants have an integer associated with them i.e. SUNDAY(1), MONDAY(2).

   When these constants are created constructor is called and day gets its value from the integer.

3. Same like any other object each enumeration constant has its own copy of instance variables defined by the enumeration.

   That’s why **d.getDay()** gives the correct day of the week for every constant.

4. The constructor for an enum type must be package-private or private access. It automatically creates the constants that are defined at the beginning of the enum body. You cannot invoke an enum constructor yourself.

5. All enums implicitly extend **java.lang.Enum**. Because a class can only extend one parent in Java an enum cannot extend anything else.

6. An enum can implement one or more interfaces.

### Java Enum - values() and valueOf() methods

In the above example code I have already used **values()** method to iterate over the values of an enum type.

These two methods **values()** and **valueOf()** are implicitly declared for all enumerations in Java.

**General form of values() and valueOf() methods**

```
public static T valueOf(String)

public static T[] values()
```

Here T is the enum type whose constant is to be returned

The **values()** method returns an array of the enumeration constants.

The **valueOf()** method returns the constant whose value corresponds to the string passed in String argument.

```
enum Day {
    SUNDAY, MONDAY, TUESDAY, WEDNESDAY,
    THURSDAY, FRIDAY, SATURDAY 
}

public class EnumDemo {

 public static void main(String[] args) {
  
  // Using values method
   Day[] allDays = Day.values();
   for(Day day : allDays){
     System.out.println(day);
   }
   // Using valueOf() method
   Day day = Day.valueOf("TUESDAY");
   System.out.println("Day - " + day);  
 }
}
```

**Output**

```
SUNDAY
MONDAY
TUESDAY
WEDNESDAY
THURSDAY
FRIDAY
SATURDAY
Day - TUESDAY
```

**Points to remember**

1. Java enumeration is a class type.
2. Enumerations can have Constructors, instance Variables, methods and can even implement Interfaces.
3. You do not instantiate an enum using new operator, even though enumeration defines a class type.
4. All Enumerations by default inherit java.lang.Enum class. So they can’t extend any other class.
5. Enum constants are implicitly static and final and can not be changed once created.

That's all for this topic **Enum Type in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!