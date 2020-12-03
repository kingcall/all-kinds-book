### Lambda Expressions in Java 8

Lambda expressions in Java (also known as **closures**) are one of the most important feature added in **Java 8**. Java lambda expressions provide a clear and elegant way to represent a single abstract method interface (Functional interface) using an expression.

In order to know Lambda expressions better it is very important to have a good understanding of two terms- **lambda expression** itself and **functional interface**.

**Table of contents**

1. [What is a functional interface](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html#functionalinterface)
2. [Java lambda expressions](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html#lambdaexpressions)
3. [Why Lambda expressions in Java](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html#Whylambdaexpressions)
4. [Java lambda expression examples](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html#LambdaexpressionsExp)
5. [Target type of Lambda expressions in Java](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html#LambdaexpressionsTargetType)
6. [Block lambda expression in Java](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html#BlockLambdaexpression)



### What is a functional interface

As already stated above for better understanding of lambda expressions in Java you should also know about functional interfaces in Java. Here is a little primer about functional interfaces.

A functional interface is an interface with only one abstract method. A functional interface is also known as **SAM type** where SAM stands for (**Single Abstract Method**). An example of functional interface would be Runnable interface which has only one method run().

Please note that from Java 8 it is possible for an interface to have [default methods](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html) and [static methods](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html) thus the stress on "*only one abstract method*".

*Also a new annotation [@FunctionalInterface](https://www.netjstech.com/2015/06/functional-interface-annotation-java-8.html) has been added in Java 8, all the functional interfaces can be annotated with this annotation to ensure that they have only single abstract method.*

**functional interface Example**

```
interface IMyInterface {
  int getValue();
}
```

In interface IMyInterface there is a single abstract method **getValue()** (note that in an interface methods are implicitly abstract).



- See [Functional Interfaces in Java](https://www.netjstech.com/2015/06/functional-interfaces-and-lambda-expression-in-java-8.html) for detailed post about functional interfaces.



### Java lambda expressions

Lambda expression in Java is an instance of a functional interface and provides implementation of the single abstract method defined by the functional interface.

**General form of Lambda Expressions in Java**

A new syntax and operator has been introduced in Java for Lambda expressions.

```
(arg1, arg2, ..) -> body
```

Here you can see a new **arrow operator** (->) or **lambda operator** which divides the lambda expression into two parts.

Left side specifies **parameters** required by the lambda expression. Parameters are **optional**, if no parameters are needed an empty parenthesis can be given. Even type of the parameters is not required as compiler, in most of the cases, is able to infer type of the parameters from the context in which it is used.



Right side known as **lambda body** specifies the logic of the lambda expression.

### Why Lambda expressions in Java

Now when we have an idea about lambda expressions and functional interfaces let's try to understand how does lambda expressions help in making your code more compact.

Most used use case for anonymous class is to implement an interface that contains only one method. This is usually done when you are trying to pass functionality as an argument to another method.

**For example** if you have a List of objects of type Person and you want to sort them on the basis of first name using a Comparator. In that case you need to implement compare() method of the Comparator interface. Usually that is done by implementing Comparator as an anonnymous class. So what you are doing is to pass the functionality to compare objects as an argument to Collections.sort() method.

```
Collections.sort(personList, new Comparator<Person>() {
  public int compare(Person a, Person b) {
    return a.getFirstName().compareTo(b.getFirstName());
  }
});
```

Java lambda expression also enables you to do the same thing like-

- Passing functionality as method argument.
- Passing code as data.

Lambda expressions can only be used to implement functional interfaces (interface with single abstract method) and lambda expressions let you express these instances of single-method classes more compactly than by using anonymous class.

**For example** if you have to implement the same Comparator as used above as a lambda expression then it can be done more compactly as-

```
Collections.sort(personList, (Person a, Person b) -> 
            a.getFirstName().compareTo(b.getFirstName()));
```

Type can be inferred from the surrounding context so you don't even need to define the type of the parameters.

```
(a, b) -> a.getFirstName().compareTo(b.getFirstName());
```

Here you can see that the functionality for sorting (implementation of compare method of the Comparator) is passed as method argument.

### Java lambda expression examples

Let's see **some examples** of the lambda expressions in Java-

\1. A very simple lambda expression.

```
() -> 7; 
```

This lambda expression takes no parameter, that's why empty parenthesis and returns the constant value 7.

If we have to write the equivalent Java method then it will be something like this -

```
int getValue(){
  return 7;
}
```

\2. Lambda expressions with 2 parameters.

```
// concatenating two strings, it has 2 string parameters
// and they are concatenated in lambda body
(String s1, String s2) -> s1+s2;
```

\3. Lambda expression to test if the given number is odd or not.

```
// Lambda expression to test if the given number is odd or not
n -> n % 2 != 0;
```

\4. Lambda expression to display passed argument.

```
// Prints the passed string s to the console and returns void
(String s) -> { System.out.println(s); };
```

\5. Complete Java Lambda expression example.

```
interface IMyInterface {
  int getValue();
}

public class LambdaExpressionDemo {
  public static void main(String[] args) {
    // reference of the interface
    IMyInterface objRef;
    // Lambda expression
    objRef = () -> 7;
    System.out.println("Value is " + objRef.getValue());
    // Another lambda expression using the same interface reference 
    objRef = () -> 7 * 5;
    System.out.println("Value is " + objRef.getValue());
    // This line will give compiler error as target type 
    // can't be inferred 
    objref = () -> "11";
  }
}
```

**Output**

```
Value is 7
Value is 35
```

Here we have an interface **IMyInterface** with one method **getValue()** whose return type is int. Since there is only one abstract method in the interface so *IMyInterface is a functional interface*.

In this program it can be seen how lambda expression **() -> 7** is compatible with the abstract method. Return type is int and there is no method parameter.
If you uncomment the line **() -> "11";** it will give compiler error [The target type of this expression must be a functional interface](https://www.netjstech.com/2015/08/target-type-of-expression-must-be-functional-interface-error.html) as in this case return type is string which is **not compatible** with the return type of the abstract method, lambda expression is implementing.

Also notice that the same functional interface reference is used to execute 2 lambda expressions
**objRef = () -> 7**; and **objRef = () -> 7 \* 5**;
Since both of the lambda expressions are compatible with the target type so both of them can be assigned to the same reference (*Does it remind you of run time polymorphism*?). That's why Lambda Expression is a **Poly Expression**.

\6. Java Lambda expression with a parameter example.

```
interface IMyInterface {
  boolean test(int n);
}

public class LambdaExpressionDemo {
  public static void main(String[] args) {
    IMyInterface objRef;
    // Lambda expression
    objRef = n -> (n % 2) == 0;
    System.out.println("4 is even " + objRef.test(4)); 
    System.out.println("3 is even " + objRef.test(3)); 
  }
}
```

**Output**

```
4 is even true
3 is even false
```

Here we need to note certain points-

**n -> (n % 2) == 0**;

In this lambda expression type is not specified explicitly as int, type is inferred from the context in which the lambda expression is executed, though type can be given explicitly too.
**int n -> (n % 2) == 0**; this lambda expression is also valid.

Also parenthesis around the parameter is omitted, in case of single parameter it is not necessary to enclose the parameter with parenthesis. Again it won't be invalid to do that, so **(n) -> (n % 2) == 0**; or **(int n) -> (n % 2) == 0**; are also valid.

In case of more than one parameter also type can be inferred so
**(int x, int y) -> x+y**; or **(x, y) -> x + y**;
both of these may be valid if used in a correct context.

One important point here is we can't have lambda expression where type for only one of the parameter is explicitly declared.

```
// Invalid lambda expression
(int x, y) -> x + y; 
```





- See [Lambda Expression Examples in Java](https://www.netjstech.com/2015/06/lambda-expression-examples-in-java-8.html) for more examples of lambda expression.



### Target type of Lambda expressions in Java

Lambda expression is an instance of the functional interface thus the **functional interface specifies its target type**.

Lambda supports "**target typing**" which infers the object type from the context in which it is used. To infer that object type from the context -

- The parameter type of the abstract method and the parameter type of the lambda expression must be compatible. For Example, if the abstract method in the functional interface specifies one int parameter, then the lambda should also have one int parameter explicitly defined or implicitly inferred as int by the context.
- Its return type must be compatible with the method's type.
- Lambda expression can throw only those exceptions which are acceptable to the method.
- 

Target typing is used in a number of contexts including the following -

- Variable declarations
- Assignments
- Return statements
- Array initializers
- Method or constructor arguments
- Lambda expression bodies

### Block lambda expression in Java

Till now all the examples we have seen are single expression lambda, we also have a second type of lambda expressions known as "**block lambda**" where the right side of the lambda expression **is a block of code**.

Let's see an example, here with in the lambda expression we have a block of code for counting the words in a string without using any String function.

```
@FunctionalInterface
interface IMyInterface {
  int doCount(String str);
}

public class LambdaExpressionDemo {
  public static void main(String[] args) {
    // Lambda expression
    IMyInterface objRef = (str) -> {
      int c = 0;
      char ch[]= new char[str.length()];
      for(int i = 0; i < str.length(); i++){
          ch[i] = str.charAt(i);
          if(((i > 0) && (ch[i] != ' ') && (ch[i-1] == ' ')) || 
          ((ch[0] != ' ') && (i == 0)))
              c++;
      }
      return c;
    };    
    System.out.println("Words in the string " + objRef.doCount("Lambda Expression in Java"));    
  }
}
```

**Output**

```
Words in the string 4
```

Note here that functional interface is annotated with a @FunctionalInterface annotation, this is a new annotation added in Java 8 to be used with functional interface.



- See [Functional interface annotation in Java 8](https://www.netjstech.com/2015/06/functional-interface-annotation-java-8.html) for detailed post about @FunctionalInterface annotation.



**Points to note-**

- Lambda expression is an instance of a functional interface and provides implementation of the single abstract method defined by the functional interface.
- The lambda expression signature must be the same as the functional interface method's signature, as the target type of the lambda expression is inferred from that context.
- A lambda expression can throw only those exceptions or the subtype of the exceptions for which an exception type is declared in the functional interface method's throws clause.
- The parameter list of the lambda expression can declare a vararg parameter: (int ... i) -> {};
- **A Lambda Expression Is a Poly Expression**- The type of a lambda expression is inferred from the target type thus the same lambda expression could have different types in different contexts. Such an expression is called a poly expression.

That's all for this topic **Lambda Expressions in Java 8**. If you have any doubt or any suggestions to make please drop a comment. Thanks!