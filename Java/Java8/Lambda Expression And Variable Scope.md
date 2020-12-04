### Java Lambda Expression And Variable Scope

The body of a [lambda expression in Java](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) **does not define a new scope**; the lambda expression scope is the same as the enclosing scope.

Let's see it with **an example**, if there is already a declared variable i and lambda expression body declares a local variable i too, that will result in compiler error "*Lambda expression's local variable i cannot re-declare another local variable defined in an enclosing scope*"

[![Java lambda expression and variable scope](https://2.bp.blogspot.com/-YsovR2NfWfs/VW8vjKLn_xI/AAAAAAAAAKc/kvgQbeKfVik/s1600/lambda%2Bexpression%2Bvariable%2Bscope.png)](https://2.bp.blogspot.com/-YsovR2NfWfs/VW8vjKLn_xI/AAAAAAAAAKc/kvgQbeKfVik/s1600/lambda%2Bexpression%2Bvariable%2Bscope.png)

### Effectively Final in Java

When a lambda expression uses an assigned local variable from its enclosing space there is an **important restriction**; Lambda expression in Java may only use local variable whose value doesn't change. That restriction is referred as **"variable capture"** which is described as; lambda expression capture values, not variables. The local variables that a lambda expression may use are known as **"effectively final"**.

An [effectively final variable](https://www.netjstech.com/2016/03/effectively-final-in-java-8.html) is one whose value does not change after it is first assigned. There is **no need to explicitly declare such a variable as final**, although doing so would not be an error.

Let's see it with **an example**, we have a local variable i which is initialized with the value 7, with in the lambda expression we are trying to change that value by assigning a new value to i. This will result in compiler error- "*Local variable i defined in an enclosing scope must be final or effectively final*"

[![lambda expression and variable scope](https://3.bp.blogspot.com/-L5JmtGM1YW8/VW8vlgt3MOI/AAAAAAAAAKk/9q2VFOlvzj0/s1600/lambda%2Bexpression%2Bvariable%2Bscope1.png)](https://3.bp.blogspot.com/-L5JmtGM1YW8/VW8vlgt3MOI/AAAAAAAAAKk/9q2VFOlvzj0/s1600/lambda%2Bexpression%2Bvariable%2Bscope1.png)

### this() and super() with lambda expression in Java

[this](https://www.netjstech.com/2015/04/this-in-java.html) and [super](https://www.netjstech.com/2015/04/super-in-java.html) references with in a lambda expression are the same as in the enclosing context. Since a lambda expression does not define a new scope, the **this keyword** with in a lambda expression signifies the **this parameter** of the method where the lambda expression is residing.

```
@FunctionalInterface
interface IFuncInt {
  int func(int num1, int num2);
  public String toString();
}

public class LambdaVarDemo {
  public static void main(String[] args){                
    LambdaVarDemo lambdaDemo = new LambdaVarDemo();
    lambdaDemo.getResult();
  }
    
  public void getResult(){
    IFuncInt funcInt = (num1, num2) -> {
      System.out.println("calling toString " + this.toString());
      return num1 + num2;        
    };
    System.out.println("Result is " + funcInt.func(6, 7));
  }
    
  @Override
  public String toString() {
    System.out.println("in class LambdaVarDemo toString()" );
    return super.toString();
  }
}
```

**Output**

```
in class LambdaVarDemo toString()
calling toString org.netjs.examples1.LambdaVarDemo@12c9b19
Result is 13
```

Here it can be seen that the expression this.toString() calls the toString method of the LambdaVarDemo object, not the toString() method of the IFuncInt instance. Please note that a functional interface may have the Object class public methods too in addition to the abstract method.

That's all for this topic **Java Lambda Expression And Variable Scope**. If you have any doubt or any suggestions to make please drop a comment. Thanks!