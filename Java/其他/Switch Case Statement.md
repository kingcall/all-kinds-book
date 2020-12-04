### Switch Case Statement in Java With Examples

Switch case statement is Java's *decision statement* which can have a number of possible execution paths. It provides an easy and more readable way to execute a set of code based on the value of the expression. For another decision statement if-else in Java you can refer this post- [if else Statement in Java With Examples](https://www.netjstech.com/2019/06/if-else-statement-in-java-with-examples.html)



**Table of contents**

1. [General form of switch statement in Java](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html#switchStmtJava)
2. [Execution path of Java switch statement](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html#JavaswitchStmtExecution)
3. [Rules for using Java switch statement](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html#JavaswitchStmtRules)
4. [Java Switch-case statement example](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html#JavaswitchStmtExp)
5. [Java Switch-case statement example with default](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html#JavaswitchStmtdefaultExp)
6. [Java Switch statement example omitting break statement](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html#SwitchStmtwithoutBreak)
7. [Java Switch statement example grouping cases](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html#SwitchStmtgroupingCases)
8. [Using Strings in Java switch Statements](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html#SwitchStmtWithString)
9. [Nested switch statements in Java](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html#NestedSwitchStmt)
10. [Java Switch-case statement efficiency](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html#SwitchStmtefficiency)



### General form of switch statement in Java

```
switch (expression) {
 case value1:
  // statement
  break;
 case value2:
  // statement
  break;
 .
 .
 .
 case valueN :
  // statement
  break;
 default:
  // default statement
}
```

In Java 12 Switch statement has been extended to be used as either a statement or an expression. Read more about switch expressions here- [Switch Expressions in Java 12](https://www.netjstech.com/2019/10/switch-expressions-in-java-12.html)

### Execution path of Java switch statement

When a value is passed in the switch expression it is compared with the value of each case statement, if match is found the code following that case statement is executed. Code execution, with in the case statement, is terminated when the **break statement** is encountered. As soon as the break statement is encountered no other case statement is executed and control comes out of the switch case statement.

If none of the values in the case statement matches the value of the expression then the **default statement** is executed. Note that *default statement is not mandatory though*. If none of the values in the case statement matches the value of the expression and default statement is also not there then control will come out of the **switch-case statement** executing nothing.

It is not mandatory to have break statement in every case statement, in case break statement is omitted in any case statement, *next case statement will also be executed even though it doesn't match the value of the expression*. That can be used to group a set of case statements but omission due to negligence will result in execution of several case statements in place of a single matching one. See **example** for more clarity.

### Rules for using Java switch statement

1. Switch statement works with byte,short, char, and int primitive data types and their wrapper classes Character, Byte, Short, and Integer. It works with [Java enum](https://www.netjstech.com/2017/03/enum-type-in-java.html) too. Starting Java 7 it also works with [String in Java](https://www.netjstech.com/2016/07/string-in-java.html).
2. Switch case statement can only be used for equality test. No other boolean expression like greater than, not equal to etc. can be used with switch statement.
3. Value of the case statement can only be a constant expression. Variable is not allowed.
4. Duplicate case values are not allowed, each case statement should have a unique value.
5. Type of each case value must be compatible with the type of expression.
6. default statement is not mandatory in Java switch statement.
7. break statement is also not mandatory. Leaving break statement with in a case statement will result in the execution of the next case statement.
8. From Java 7 string class can also be used with Switch-Case statement in Java.

### Java Switch-case statement example

```
public class SwitchCaseDemo {
 public static void main(String[] args) {
  int month = 7;
  switch(month){
   case 1: System.out.println("Month is January"); 
   break;
   case 2: System.out.println("Month is February"); 
   break;
   case 3: System.out.println("Month is March"); 
   break;
   case 4: System.out.println("Month is April"); 
   break;
   case 5: System.out.println("Month is May"); 
   break;
   case 6: System.out.println("Month is June"); 
   break;
   case 7: System.out.println("Month is July"); 
   break;
   case 8: System.out.println("Month is August"); 
   break;
   case 9: System.out.println("Month is September"); 
   break;
   case 10: System.out.println("Month is October"); 
   break;
   case 11: System.out.println("Month is November"); 
   break;
   case 12: System.out.println("Month is December"); 
   break;
  }
 }
}
```

**Output**

```
Month is July
```

Here month expression has the value 7 so it will go to the case statement which has value 7 and execute the statements with in that case statement and then break out.

**Note** that default statement is not used in this code.

### Java Switch-case statement example with default

Let us change this example to pass month as 13 and have a default statement to encounter such cases.

```
public class SwitchCaseDemo {
 public static void main(String[] args) {
  int month = 13;
  switch(month){
   case 1: System.out.println("Month is January"); 
   break;
   case 2: System.out.println("Month is February"); 
   break;
   case 3: System.out.println("Month is March"); 
   break;
   case 4: System.out.println("Month is April"); 
   break;
   case 5: System.out.println("Month is May"); 
   break;
   case 6: System.out.println("Month is June"); 
   break;
   case 7: System.out.println("Month is July"); 
   break;
   case 8: System.out.println("Month is August"); 
   break;
   case 9: System.out.println("Month is September"); 
   break;
   case 10: System.out.println("Month is October"); 
   break;
   case 11: System.out.println("Month is November"); 
   break;
   case 12: System.out.println("Month is December"); 
   break;
   default: System.out.println("Invalid month value passed");
  }
 }
}
```

**Output**

```
Invalid month value passed
```

### Java Switch statement example omitting break statement

As stated above **break statement is not mandatory** but it will result in fall-through i.e. execution of the following case statements until the break statement is encountered or the statement ends.

```
public class SwitchCaseDemo {
 public static void main(String[] args) {
  int month = 7;
  
  switch(month){
   case 1: System.out.println("Month is January");
   
   case 2: System.out.println("Month is February"); 
   
   case 3: System.out.println("Month is March"); 
   
   case 4: System.out.println("Month is April"); 
   
   case 5: System.out.println("Month is May"); 
   
   case 6: System.out.println("Month is June"); 
   
   case 7: System.out.println("Month is July"); 
   
   case 8: System.out.println("Month is August"); 
   
   case 9: System.out.println("Month is September"); 
   
   case 10: System.out.println("Month is October"); 
   
   case 11: System.out.println("Month is November"); 
   
   case 12: System.out.println("Month is December"); 
   
   default: System.out.println("Invalid month value passed");
  }
 }
}
```

**Output**

```
Month is July
Month is August
Month is September
Month is October
Month is November
Month is December
Invalid month value passed
```

In the above switch case statement break is not used with any case statement. Since month passed is 7 so control will go to the case statement having value 7 and start executing the code with in that case statement but it won't know when to break so all the following case statements (even default) are executed.

### Java Switch statement example grouping cases

Case statements **can be grouped** by omitting the break statement. **As example** if you want to display the quarter passed month falls into then you can group three case statements where break statement is used with only the last one in the group.

```
public class SwitchCaseDemo {
 public static void main(String[] args) {
  int month = 5;
  
  switch(month){
   case 1:   
   case 2:   
   case 3: System.out.println("Quarter 1"); 
     break;
   
   case 4:   
   case 5:    
   case 6: System.out.println("Quarter 2"); 
     break;
   
   case 7:   
   case 8:   
   case 9: System.out.println("Quarter 3"); 
     break;
   
   case 10:     
   case 11:    
   case 12: System.out.println("Quarter 4"); 
      break;
   
   default: System.out.println("Invalid month value passed");
  }
 }
}
```

**Output**

```
Quarter 2
```

### Using Strings in Java switch Statements

In **Java SE 7 and later**, you can use a [String](https://www.netjstech.com/2016/07/string-comparison-in-java-startswith-equals-method.html) object in the switch statement's expression.

```
public class StringSwitchDemo {

 public static void main(String[] args) {
  String dept = "Human Resources";
  String deptCd = null;
  switch(dept.toLowerCase()){
   case "account": 
    deptCd = "acct";
    break;
   case "human resources": 
    deptCd = "hr";
    break;
   case "administration":
    deptCd = "admin";
    break;
   default: System.out.println("Invalid deprtment");
  }
  System.out.println("Department - " + deptCd);
 }
}
```

**Output**

```
Department – hr
```

### Nested switch statements in Java

You can have nested switch-case statement in Java with in the outer switch statement.

```
switch(month){
 case 1:
 switch(week){
  case 1: .....
         break;
  case 2 : ....

  ...
  ...
 }
 break;
}
```

### Java Switch-case statement efficiency

If you have to make a choice between sequence of **if-else** statement and **switch-case** statement opt for switch case as it will run faster.

Reason being when the Java compiler compiles a switch-case statement it will go through all the case statement values (which are constants) and make a **jump table**. The value on which the switch is being performed is used as an index into the jump table to directly go to that case statement. In the case of switch-case statement it can be done as compiler knows that case values are all of same type and switch expression can only be used for equality test with case values.

That's all for this topic **Switch Case Statement in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!