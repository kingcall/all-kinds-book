### Ternary Operator in Java With Examples

Java includes a conditional operator known as **ternary operator (?:)**, which can be thought of as a shorthand for an if-else statement. This operator is known as the ternary operator because it uses **three operands**.

### General form of ternary operator in Java

```
boolean-exp ? expression2 : expression3
```

If boolean expression evaluates to true then expression2 is evaluated and its value becomes the result of the operator; otherwise (if boolean expression is false) expression3 is evaluated and its value becomes the result of the operator.

As **example** if you have this statement-

```
int result = someCondition ? value1 : value2;
```

The statement should be read as: "*If someCondition is true, assign the value of value1 to result. Otherwise, assign the value of value2 to result.*"

Note that in Java Ternary operator both expression2 and expression3 should return the value of same (or compatible) type.

### Usage of Java ternary operator

Though [switch-case statement in Java](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html) provides performance improvement over [if-else statement](https://www.netjstech.com/2019/06/if-else-statement-in-java-with-examples.html) *there is no performance improvement if ternary operator is used over an if-else statement* but it makes the code more readable and terse.

Ternary operator example-

```
int val = (i > 10) ? i * 5 : i * 10;
```

Here val is assigned the value (i * 5) if i is greater than 10 otherwise val is assigned the value (i * 10).

Writing the same thing using **if-else** will be a multiple line affair.

```
if(i > 10){
  val = i * 5;
}else{
  val = i * 10;
}
```

If you were to write a method where value is returned using the same logic, with ternary operator the method will look like-

```
public int getValue(int i){
 return (i > 10) ? i * 5 : i * 10;
}
```

In this case if-else statement can also be made more compact-

```
public static int getValue(int i){
  if(i > 10)
    return i * 5;
  return i * 10;
}
```

Not its up to you to decide what looks more readable. I feel once you get a grip over ternary operator it will always be more readable.

**Another example**– This Java ternary operator example uses String expression and prints a message based on the value of the string.

```
String day = "saturday";
System.out.println(((day.equalsIgnoreCase("saturday") || day.equalsIgnoreCase("sunday"))
    ? "weekend! time to party!" : "Working day"));
```

**Output**

```
weekend! time to party!
```

### Nested ternary operator

Ternary operator in Java can be nested but doing that considerably *decreases the readability*.

**As example** if you have the logic that number of transactions in a week is-

```
upto 5 means ok, 
more than 5 and less than 10 means notify the user 
more than 10 means alarm!!
```

Then you can use the nested ternary operator

```
String noOfTransactions = (i < 5) ? "ok" : (i > 5 && i < 10) ? "notify" : "alarm";
```

Generally it is not advisable to use nested ternary operator as it defeats the purpose of making the code more readable in contrast it makes it look more complex and confusing in most of the cases.

That's all for this topic **Ternary Operator in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!