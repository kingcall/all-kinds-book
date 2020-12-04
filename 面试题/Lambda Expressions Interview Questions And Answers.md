### Java Lambda Expressions Interview Questions And Answers

In this post some of the Java Lambda Expressions interview questions and answers are listed. This compilation will help the Java developers in preparing for their interviews especially when asked interview questions about Java 8.

1. **What is lambda expression?**

   Lambda expression in itself is an anonymous method i.e. a method with no name which is used to provide implementation of the method defined by a functional interface.

   A new syntax and operator has been introduced in Java for Lambda expressions.

   General form of lambda expression

   ```
   (optional) (Arguments) -> body
   ```

   Here you can see a new arrow operator or lambda operator which divides the lambda expression into two parts. Left side specifies parameters required by the lambda expression. Parameters are optional, if no parameters are needed an empty parenthesis can be given. Right side known as lambda body specifies the logic of the lambda expression.

   Read more about lambda expressions in Java

    

   here

   .

2. ------

3. **Give some examples of lambda expressions.**

   A very simple example of lambda expression would be-

   ```
   () -> 5
   ```

   This lambda expression takes no parameter (that's why empty parenthesis) and returns the constant value 5.

   The above lambda expression is equivalent to the following Java code-

   ```
   int getValue(){
       return 5;
   }
   ```

   **Another example** which has 2 parameters-

   ```
   // concatenating two strings, it has 2 string parameters and they are concatenated in lambda body
   (String s1, String s2) -> s1+s2;
   ```

   See more lambda expressions examples

    

   here

   .

4. ------

5. **What is a functional interface?**

   A functional interface is an interface with only one abstract method. A functional interface is also known as SAM type where SAM stands for (Single Abstract Method).

   An example of functional interface with in Java would be Runnable interface which has only one method run().

   ```
   public interface Runnable {
     public void run();
   }
   ```

   Read more about functional interface in Java

    

   here

   .

6. ------

7. **How do you use lambda expression with functional interface?**

   Lambda expression provides implementation of the abstract method defined by the functional interface.

   Read more about functional interface in Java

    

   here

   .

8. ------

9. **How target type is inferred for the lambda expression?**

   Lambda expression doesn't have a type of its own. A lambda expression provides implementation of the abstract method defined by the functional interface. Thus the functional interface specifies its target type.

   Lambda supports "target typing" which infers the object type from the context in which it is used.

   To infer that object type from the context -

   - The parameter type of the abstract method and the parameter type of the lambda expression must be compatible. **For Example**, if the abstract method in the functional interface specifies one int parameter, then the lambda should also have one int parameter explicitly defined or implicitly inferred as int by the context.
   - Its return type must be compatible with the method's type.
   - Lambda expression can throw only those exceptions which are acceptable to the method.

   Read more about lambda expressions in Java

    

   here

   .

10. ------

11. **Explain target typing with an example.**

    Let's say there is a functional interface IMyFunc

    ```
    interface IMyFunc {
        int getValue(int num);
    }
    ```

    And a class LambdaDemo with a lambda expression which implements this functional interface.

    ```
    public class LambdaDemo {
      public static void main(String[] args) {
        // lambda expression
        IMyFunc myFactorialFunc = (num) -> {
          int fact = 1;
          for(int i = 1; i <= num; i++){
            fact = i * fact;
          }
          return fact;
        };
        System.out.println("Factorial of 7 is " + myFactorialFunc.getValue(7));
      }
    }
    ```

    Here you can see that the lambda expression is assigned to the variable of type IMyFunc(functional interface). Now when the method myFactorialFunc.getValue(7) is called it will know that the lambda expression is providing the implementation of the method getValue() because -

    - The parameter type of the abstract method and the parameter type of the lambda expression are compatible.
    - getValue() method's return type is compatible with the return type of the lambda expression.

    So you can see in this example how target type of the lambda expression is inferred from the context.

    See more lambda expressions examples

     

    here

    .

12. ------

13. **(int x, int y) -> x+y; or (x, y) -> x + y; which one of these is a valid lambda expression?**

    Both of them are valid lambda expressions if used in a correct context.

    With the first one **(int x, int y) -> x+y;** we know that the parameters must be of type int.

    In case of (x, y) -> x + y; if used in a correct context type can be inferred from the context in which the lambda expression is executed.

    See more lambda expressions examples

     

    here

    .

14. ------

15. **(int x, y) -> x + y; is this a valid lambda expression?**

    You can't have lambda expression where type for only one of the parameter is explicitly declared so this lambda expression is invalid.

    See more lambda expressions examples

     

    here

    .

16. ------

17. **What is block lambda expression?**

    A block lambda is the lambda expression where the right side of the lambda expression is a block of code.

    As example -

    ```
    IMyFunc myFactorialFunc = (num) -> {
      int fact = 1;
      for(int i = 1; i <= num; i++){
        fact = i * fact;
      }
      return fact;
    };
    ```

    Where as

     

    (String s1, String s2) -> s1+s2;

     

    is a single expression lambda.

18. ------

19. **Why lambda expression is called a poly expression?**

    The type of a lambda expression is inferred from the target type thus the same lambda expression could have different types in different contexts.

    **As example-**

    If there is a functional interface **IMyInterface**

    ```
    interface IMyInterface {
      int getValue();
    }
    ```

    Then both of the following lambda expressions can be of type IMyInterface.

    ```
    // Lambda expression
    objRef = () -> 7;
    
    // Another lambda expression using the same interface reference 
    objRef = () -> 7 * 5;
    ```

    **Another example-**

    Let’s say there are these 2 functional interfaces.

    ```
    @FunctionalInterface
    interface TestInterface {
      int addValues(int val1, int val2);
    }
    ```

    ```
    @FunctionalInterface
    interface StrInterface {
      String concat(String str1, String str2);
    }
    ```

    Then you can write a lambda expression as follows-

    ```
    TestInterface obj1;
    obj1 = (x, y) -> x + y;
    System.out.println("" + obj1.addValues(2, 3));
        
    StrInterface obj2;
    obj2 = (x, y) -> x + y;
    System.out.println("" + obj2.concat("Hello", "Lambda"));
    ```

    If you notice here same lambda expression (x, y) -> x + y; is used both of the times but the type of the parameters and the return type determines which functional interface should be referred.

    Read more about functional interface in Java

     

    here

    .

20. ------

21. **Can we have a generic functional interface?**

    Since lambda expression doesn't have type parameters of its own so it can't be generic. But the functional interface that specifies the target type for the lambda expression can be generic.

    **As example-**

    A generic functional interface-

    ```
    @FunctionalInterface
    interface GenInterface<T, U, R> {
      R addValues(T t, U u);
    }
    ```

    Lambda expression implementing the given functional interface with Integer types.

    ```
    GenInterface<Integer, Integer, Integer> obj1;
    obj1 = (x, y) -> x + y;
    System.out.println("" + obj1.addValues(2, 3));
    ```

    Lambda expression implementing the given functional interface with String types.

    ```
    GenInterface<String, String, String> obj2;
    obj2 = (x, y) -> x + y;
    System.out.println("" + obj2.addValues("Hello", "Lambda")); 
    ```

    Read more about functional interface in Java

     

    here

    .

22. ------

23. **What about inbuilt functional interfaces?**

    With Java 8 many new functional interfaces are being defined, in fact there is a whole new package **java.util.function** added with many functional interfaces. The interfaces in this package are general purpose functional interfaces used by the JDK, and are available to be used by user code as well.

    Some of the inbuilt functional interfaces -

    ```
    public interface Consumer<T> {
      void accept(T t);
    }
     
    public interface Supplier {
      T get();
    }
    
    public interface Function {
      R apply(T t);
    }
    ```

    Read more about functional interface in Java

     

    here

    .

24. ------

25. **Comparator method is a functional interface but I see a lot of other methods in Comparator method then how is it a Single Abstract method interface?**

    From Java 8 it is possible for an interface to have default methods and static methods so, in a functional interface there may be other default and static methods but there must be only one abstract method.

    A functional interface can specify Object class public methods too in addition to the abstract method. That interface will still be a valid functional interface. The public Object methods are considered implicit members of a functional interface as they are automatically implemented by an instance of functional interface.

    Read more about functional interface in Java

     

    here

    .

26. ------

27. **What is functional interface annotation?**

    Java 8 also introduces an annotation @FunctionalInterface to be used with functional interface. Annotating an interface with @FunctionalInterface indicates that an interface type declaration is intended to be a functional interface.

    Read more about functional interface annotation in Java

     

    here

    .

28. ------

29. **Is it mandatory to mark functional interface with @FunctionalInterface annotation?**

    It is not mandatory to mark functional interface with @FunctionalInterface annotation, it is more of a best practice to do that and also gives a surety that no other abstract method will be added accidentally to the functional interface. Because it will result in compiler error if any other abstract method is added to a functional interface which is annotated with @FunctionalInterface annotation.

    Read more about functional interface annotation in Java

     

    here

    .

30. ------

31. **How can we write runnable as lambda expression?**

    Refer

     

    Lambda Expression Runnable example

     

    to see how to write runnable as lambda expression.

    ------

32. **How can we write comparator as lambda expression?**

    Refer

     

    Lambda Expression Comparator example

     

    to see how to write comparator as lambda expression.

    ------

33. **How can we write callable as lambda expression?**

    Refer

     

    Lambda Expression Callable example

     

    to see how to write runnable as lambda expression.

    ------

34. **What is effective final in Java? What is variable capture?**

    When a lambda expression uses an assigned local variable from its enclosing space there is an important restriction. A lambda expression may only use local variable whose value doesn't change. That restriction is referred as "variable capture".

    The local variables that a lambda expression may use are known as "effectively final". An effectively final variable is one whose value does not change after it is first assigned.

    Refer

     

    effectively final in Java 8

     

    to know more about effectively final in Java.

    ------

35. **Can lambda expression throw exception? Is there any restriction in lambda expression exception handling?**

    A lambda expression can throw an exception but lambda expression must throw exception compatible with those specified in the throws clause of the functional interface method.

    If a lambda expression body throws an exception, the throws clause of the functional interface method must declare the same exception type or its supertype.

    Refer

     

    Lambda expression and exception handling

     

    to know more about exception handling in lambda expressions in Java.

    ------

36. **What is method reference in Java 8?**

    Lambda expressions can be used to call an existing method. Java 8 provides another feature called method reference that provides a clearer alternative to refer to the existing method by name.

    General form of Method reference- **ClassName (or object)::methodName**

    class name or instance is separated from the method name by a double colon. The :: is a new separator (known as double colon operator) that has been added in Java 8.

    Refer

     

    Method reference in Java 8

     

    to know more about Method Reference in Java 8.

    ------

37. **Give an example of method reference.**

    A very simple example of method reference would be how we call System.out.println.

    Suppose you are using a forEach statement to print all the elements of the list then the lambda expression for the same would be written as-

    ```
    myList.forEach(x -> System.out.println(x))
    ```

    Using method reference it can be written as -

    ```
    list.forEach(System.out::println);
    ```

    Refer

     

    Method reference in Java 8

     

    to know more about Method Reference in Java 8.

    ------

38. **What are the types of method reference?**

    There are four kinds of method references:

    | Kind                                                         | Example                              | Syntax                        |
    | ------------------------------------------------------------ | ------------------------------------ | ----------------------------- |
    | Reference to a static method                                 | ContainingClass::staticMethodName    | ClassName::methodName         |
    | Reference to an instance method of a particular object       | containingObject::instanceMethodName | objRef::methodName            |
    | Reference to an instance method of an arbitrary object of a particular type | ContainingType::methodName           | ClassName::instanceMethodName |
    | Reference to a constructor                                   | ClassName::new                       | classname::new                |