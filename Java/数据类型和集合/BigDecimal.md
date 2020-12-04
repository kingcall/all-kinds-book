### [BigDecimal in Java With Examples](https://www.netjstech.com/2017/04/bigdecimal-in-java.html)

This post gives an overview of BigDecimal in Java and the examples of BigDecimal class usage in scale manipulation, rounding and format conversion where more precision is required.

### Why BigDecimal is needed

While reading about [primitive date types in Java](https://www.netjstech.com/2017/03/primitive-data-types-in-java.html) we get to know that we should use float and double primitive types for decimal numbers. But there is one problem with these primitive types float and double that these types should never be used for precise value, such as currency.

**Reference**: https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html

**As exmaple**

```
double d1 = 374.56;
double d2 = 374.26;
System.out.println( "d1 - d2 = " + ( d1 - d2 ));
```

Here you may expect the output to be .30 but the actual output is-

```
d1 - d2 = 0.30000000000001137
```

That’s why in financial applications where **scale and rounding mode** for the numbers is an important aspect while doing calculations, it’s a better choice to go with **BigDecimal** in Java.

### Java BigDecimal class

BigDecimals are immutable, arbitrary-precision signed decimal numbers which can be used for monetary calculations.

In the example used above if we use BigDecimal instead of double then value would be calculated precisely.

```
BigDecimal bd1 = new BigDecimal("374.56");
BigDecimal bd2 = new BigDecimal("374.26");
  
System.out.println("bd1 - bd2 = " + bd1.subtract(bd2));
```

Now the output will be as expected-

```
bd1 - bd2 = 0.30
```

BigDecimal class in Java extends Number and implements [Comparable](https://www.netjstech.com/2015/10/difference-between-comparable-and-comparator-java.html).

```
public class BigDecimal extends Number implements Comparable<BigDecimal> {

}
```

### Constructors in BigDecimal class

BigDecimal class in Java provides many constructors where a BigDecimal object can be initialized using int, char[], BigDecimal, String, double, long. In total 18 [constructors](https://www.netjstech.com/2015/04/constructor-chaining-in-java-calling-one-constructor-from-another.html) are there.

One thing to note here is using a double value to initialize a BigDecimal may give the precision problem again. As seen in the example below-

```
BigDecimal bde = new BigDecimal(23.12);
System.out.println("" + bde.toString());
```

Output you get from printing this BigDecimal is-

```
23.120000000000000994759830064140260219573974609375
```

Thus it is always safe to go with a constructor that takes [String](https://www.netjstech.com/2016/07/string-in-java.html) as argument when representing a decimal value.

```
BigDecimal bde = new BigDecimal("23.12");
System.out.println("" + bde.toString());
```

**Output**

```
23.12
```

### Scaling in BigDecimal

One of the biggest reason to use BigDecimal is to be able to provide **scale** (How many numbers will be there after the decimal) and to provide rounding mode.

In order to specify the number of digits after the decimal point you can use the **setScale(int scale)** method.

But it is always better to provide the rounding mode too along with scale. For that there are two [overloaded methods](https://www.netjstech.com/2015/04/method-overloading-in-java.html).

- setScale(int newScale, int roundingMode)
- setScale(int newScale, RoundingMode roundingMode)

Let’s see an example to show why you should that, let’s assume you are using a double value to construct a BigDecimal object then you will loose some precision as shown in an example in the Constructor section.

```
BigDecimal bde = new BigDecimal(23.12);
System.out.println("Value- " + bde.toString());
System.out.println("Scaled value- " + bde.setScale(1).toString());
```

**Output**

```
Value- 23.120000000000000994759830064140260219573974609375
Exception in thread "main" java.lang.ArithmeticException: Rounding necessary
 at java.base/java.math.BigDecimal.commonNeedIncrement(BigDecimal.java:4495)
 at java.base/java.math.BigDecimal.needIncrement(BigDecimal.java:4702)
 at java.base/java.math.BigDecimal.divideAndRound(BigDecimal.java:4677)
 at java.base/java.math.BigDecimal.setScale(BigDecimal.java:2811)
 at java.base/java.math.BigDecimal.setScale(BigDecimal.java:2853)
 at org.netjs.Programs.App.main(App.java:15)
```

From the output you can see that some precision is lost as the value of bde is **23.120000000000000994759830064140260219573974609375**. When setting scale as 1 it is not known what is the rounding mechanism so Arithmetic [exception](https://www.netjstech.com/2015/05/overview-of-java-exception-handling.html) is thrown instead.

### Rounding modes in Java BigDecimal

There are eight rounding modes provided by the BigDecimal class as **static final int**. If you have noticed above in scaling section there are two overloaded methods where the second one takes **RoundingMode** as parameter. RoundingMode is an [Enum](https://www.netjstech.com/2017/03/enum-type-in-java.html) provided in the [package](https://www.netjstech.com/2016/07/package-in-java.html) **java.math**.

It is **always recommended** to use RoundingMode enum in place of int constants. According to Java docs-

*“Using the integer fields in this class (such as ROUND_HALF_UP) to represent rounding mode is largely obsolete; the enumeration values of the RoundingMode enum, (such as RoundingMode.HALF_UP) should be used instead.”*

**RoundingMode Enum Constants**

Eight rounding modes provided are-

- **CEILING**- Rounding mode to round towards positive infinity.
- **DOWN**- Rounding mode to round towards zero.
- **FLOOR**- Rounding mode to round towards negative infinity.
- **HALF_DOWN**- Rounding mode to round towards "nearest neighbor" unless both neighbors are equidistant, in which case round down.
- **HALF_EVEN**- Rounding mode to round towards the "nearest neighbor" unless both neighbors are equidistant, in which case, round towards the even neighbor.
- **HALF_UP**- Rounding mode to round towards "nearest neighbor" unless both neighbors are equidistant, in which case round up.
- **UNNECESSARY** - Rounding mode to assert that the requested operation has an exact result, hence no rounding is necessary.
- **UP**- Rounding mode to round away from zero.

Here is a summary table showing the results of these rounding operations for all rounding modes.

|              | Result of rounding input to one digit with the given rounding mode |        |           |         |           |             |             |                             |
| ------------ | ------------------------------------------------------------ | ------ | --------- | ------- | --------- | ----------- | ----------- | --------------------------- |
| Input Number | `UP`                                                         | `DOWN` | `CEILING` | `FLOOR` | `HALF_UP` | `HALF_DOWN` | `HALF_EVEN` | `UNNECESSARY`               |
| 5.5          | 6                                                            | 5      | 6         | 5       | 6         | 5           | 6           | throw `ArithmeticException` |
| 2.5          | 3                                                            | 2      | 3         | 2       | 3         | 2           | 2           | throw `ArithmeticException` |
| 1.6          | 2                                                            | 1      | 2         | 1       | 2         | 2           | 2           | throw `ArithmeticException` |
| 1.1          | 2                                                            | 1      | 2         | 1       | 1         | 1           | 1           | throw `ArithmeticException` |
| 1.0          | 1                                                            | 1      | 1         | 1       | 1         | 1           | 1           | 1                           |
| -1.0         | -1                                                           | -1     | -1        | -1      | -1        | -1          | -1          | -1                          |
| -1.1         | -2                                                           | -1     | -1        | -2      | -1        | -1          | -1          | throw `ArithmeticException` |
| -1.6         | -2                                                           | -1     | -1        | -2      | -2        | -2          | -2          | throw `ArithmeticException` |
| -2.5         | -3                                                           | -2     | -2        | -3      | -3        | -2          | -2          | throw `ArithmeticException` |
| -5.5         | -6                                                           | -5     | -5        | -6      | -6        | -5          | -6          | throw `ArithmeticException` |

**Reference**- https://docs.oracle.com/javase/8/docs/api/java/math/RoundingMode.html

### Java BigDecimal Examples

In most of the cases conventional logic is to have a **scale of 2** (2 digits after the decimal) and rounding up if the digit **after the scale is >= 5**.

```
BigDecimal bd1 = new BigDecimal("23.1256");
System.out.println("bd1 " + bd1.setScale(2, RoundingMode.HALF_UP).toString());
```

Here scale is set to 2 and digit after the 2 decimal digits is 5 so the last digit(after scaling) will be rounded up and the output is 23.13.

```
BigDecimal bd1 = new BigDecimal("23.1236");
System.out.println("bd1 " + bd1.setScale(2, RoundingMode.HALF_UP).toString());
```

Here digit after the 2 decimal digits is 3 (less than 5) so the last digit(after scaling) will not be rounded up and the output is 23.12.

```
BigDecimal bd1 = new BigDecimal("-15.567");
System.out.println("bd1 " + bd1.setScale(2, RoundingMode.HALF_UP).toString());
```

Same logic applies to negative numbers too so here the output is -15.57.

### Features of BigDecimal in Java

1. No Overloaded operators

   – In Java arithmetic operators (+, -, *, /) are not permitted with objects so these operators are not used with BigDecimal numbers, you will have to use method calls instead. BigDecimal class has methods

    

   add

   ,

    

   subtract

   ,

    

   multiply

    

   and

    

   divide

    

   for the arithmetic operations.

   ```
   BigDecimal bd1 = new BigDecimal("15.567");
   
   BigDecimal result = BigDecimal.valueOf(68).multiply(bd1);
   System.out.println("result " + result.toString());
   ```

   **Output**

   ```
   result 1058.556
   ```

2. Use compareTo() to compare BigDecimals not equals()

   \- Don’t use equals method to compare 2 BigDecimal numbers as this method considers two BigDecimal objects equal only if they are equal in value and scale (thus 2.0 is not equal to 2.00 when compared by this method.

   ```
   BigDecimal bd1 = new BigDecimal("2.00");
   BigDecimal bd2 = new BigDecimal("2.0");
   System.out.println("bd1 equals bd2 - " + bd1.equals(bd2));
   ```

   **Output**

   ```
   bd1 equals bd2 - false
   ```

   You should use **compareTo()** method instead, **BigDecimal** class implements comparable and provides its own implementation of compareTo() method.

   Two BigDecimal objects that are equal in value but have a different scale (like 2.0 and 2.00) are considered equal by this method.

   For a statement like **bd1.compareTo(bd2)** this method returns -

   - -1 if bd1 is less than bd2.
   - 0 if both are equal.
   - 1 if bd1 is greater than bd2.

   ```
   BigDecimal bd1 = new BigDecimal("2.00");
   BigDecimal bd2 = new BigDecimal("2.0");
   System.out.println("bd1 compareTo bd2 - " + bd1.compareTo(bd2));
   ```

   **Output**

   ```
   bd1 compareTo bd2 - 0
   ```

3. **BigDecimals are immutable**- BigDecimal objects are **immutable**, so any operation won't result in the original object being modified. You can take example of the **setScale** method, usual convention is that methods named **setX** mutate field X. But setScale returns an object with the proper scale; the returned object may or may not be newly allocated.

**Points to remember**

- The BigDecimal class should be used when we need accurate precision rather than approximation.
- BigDecimal class in Java provide methods to provide scale and rounding options for the result.
- BigDecimal class extends Number class like other [wrapper classes](https://www.netjstech.com/2017/03/typewrapper-classes-in-java.html).
- BigDecimal class has specific methods for addition, subtraction, multiplication, and division.
- BigDecimal objects are immutable.
- With BigDecimal object creation overhead is involved so operations are slightly slower compared to primitive types.

That's all for this topic **BigDecimal in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!