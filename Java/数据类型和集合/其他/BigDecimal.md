[TOC]

## BigDecimal

这篇文章我们会介绍一下Java 中的BigDecimal,并且会通过一些例子演示它的用法，例如精度的操作

Java在java.math包中提供的API类BigDecimal，用来对超过16位有效位的数进行精确的运算。双精度浮点型变量double可以处理16位有效数，但在实际应用中，可能需要对更大或者更小的数进行运算和处理。一般情况下，对于那些不需要准确计算精度的数字，我们可以直接使用Float和Double处理，但是Double.valueOf(String) 和Float.valueOf(String)会丢失精度。所以开发中，如果我们需要精确计算的结果，则必须使用BigDecimal类来操作。

### 为什么需要BigDecimal

前面学习[基本类型](https://blog.csdn.net/king14bhhb/article/details/110631164)的时候,我们可以使用float 和 double来表示浮点型数字，但是这里有一个问题那就是基本数据类型float 和 double不应该用于高精度数据的表示，例如货币，因为浮点类型的float 和 double会丢失数据的精度

```java
double d1 = 374.56;
double d2 = 374.26;
System.out.println( "d1 - d2 = " + ( d1 - d2 ));
```

这里本应输出的是0.30，但是实际上输出如下

```
d1 - d2 = 0.30000000000001137
```

这就是为什么在金融相关的程序里数据类型如此重要的原因，所以我们更好的选择是**BigDecimal** 而不是float 和 double

### Java BigDecimal 类

BigDecimal 是不可变的，任意精度的有符号十进制类型的数字，可用于货币计算

上面那个例子如果我们是使用BigDecimal来代替double 我们可以获得准确的值

```java
BigDecimal bd1 = new BigDecimal("374.56");
BigDecimal bd2 = new BigDecimal("374.26");
  
System.out.println("bd1 - bd2 = " + bd1.subtract(bd2));
```

现在输出就和预期的一样了

```
bd1 - bd2 = 0.30
```

Java 中的BigDecimal类继承自Number并且实现了[Comparable](https://blog.csdn.net/king14bhhb/article/details/110941207)接口

```
public class BigDecimal extends Number implements Comparable<BigDecimal> {

}
```

### BigDecimal 类的构造方法

BigDecimal 提供了很多的构造方法，可以使用int ,char[],BigDecimal,String,doble ,long,int来初始化BigDecimal，BigDecimal 总共提供了18种构造方法，需要注意的实如果使用double 来初始化BigDecimal或许会再次引入精度的问题，下面提供了一个例子

```java
BigDecimal bde = new BigDecimal(23.12);
System.out.println("" + bde.toString());
```

输出结果是这样的

```
23.120000000000000994759830064140260219573974609375
```

Thus it is always safe to go with a constructor that takes [String](https://www.netjstech.com/2016/07/string-in-java.html) as argument when representing a decimal value.

因此使用String 做为构造函数的参数来表示一个十进制的数字的时候总是安全的

```java
BigDecimal bde = new BigDecimal("23.12");
System.out.println("" + bde.toString());
```

**Output**

```
23.12
```

### BigDecimal 的精度

使用BigDecimal的一个理由是BigDecimal提供了精度控制(小数点后的数字的多少)和舍入模式，为了确定小数点后的保留几位数字你可以使用**setScale(int scale)** 方法，但是最好的是在使用精度的时候提供舍入模式，也就是setScale的重载方法

- setScale(int newScale, int roundingMode)
- setScale(int newScale, RoundingMode roundingMode)

接下来我们通过一个例子演示一下我们为什么要这样做，假设我们在通过一个double值构造一个BigDecimal

```java
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

从上面的输出中我们看到进度已经丢失了，输出的BigDecimal 是**23.120000000000000994759830064140260219573974609375**



并且我们看到当我们将精度设置为1 的时候并且没有提供舍入机制的时候导致Arithmetic异常被抛出

### BigDecimal 的舍入模式

如果你注意到了上面我们在讲精度设置的时候，它其实是有两个设置精度的重载方法，第二个参数代表的就是舍入模式模式的参数，BigDecimal提供了八种舍入模式，它们通过**static final int** 进行表示

```java
public final static int ROUND_UP =           0;
public final static int ROUND_DOWN =         1;
public final static int ROUND_CEILING =      2;
public final static int ROUND_FLOOR =        3;
public final static int ROUND_HALF_UP =      4;
public final static int ROUND_HALF_DOWN =    5;
public final static int ROUND_HALF_EVEN =    6;
public final static int ROUND_UNNECESSARY =  7;
```

需要注意的是在**java.math**包中也提供舍入模式的枚举值，需要注意的我们是推荐使用枚举值来代替使用int 类型的常量做舍入摸模式的参数

下面我们在设置进度的同时设置一下舍入模式，来避免Arithmetic异常

```java
@Test
public void scale() {
    BigDecimal bde = new BigDecimal(23.12);
    System.out.println("Scaled value- " + bde.setScale(1,1).toString());
}
```

但是我们说了，我们推荐使用枚举值的舍入模式，而不是直接使用int 类型的常量，接下来我们看一下`RoundingMode` 提供的枚举值

- **CEILING**- Rounding mode to round towards positive infinity.
- **DOWN**- Rounding mode to round towards zero.
- **FLOOR**- Rounding mode to round towards negative infinity.
- **HALF_DOWN**- Rounding mode to round towards "nearest neighbor" unless both neighbors are equidistant, in which case round down.
- **HALF_EVEN**- Rounding mode to round towards the "nearest neighbor" unless both neighbors are equidistant, in which case, round towards the even neighbor.
- **HALF_UP**- Rounding mode to round towards "nearest neighbor" unless both neighbors are equidistant, in which case round up.
- **UNNECESSARY** - Rounding mode to assert that the requested operation has an exact result, hence no rounding is necessary.
- **UP**- Rounding mode to round away from zero.

下面我们通过例子来总结一下这些舍入模式的舍入方式

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

**更多细节参考**- https://docs.oracle.com/javase/8/docs/api/java/math/RoundingMode.html



### BigDecimal格式化

#### NumberFormat 格式化

由于NumberFormat类的format()方法可以使用BigDecimal对象作为其参数，可以利用BigDecimal对超出16位有效数字的货币值，百分值，以及一般数值进行格式化控制。

以利用BigDecimal对货币和百分比格式化为例。首先，创建BigDecimal对象，进行BigDecimal的算术运算后，分别建立对货币和百分比格式化的引用，最后利用BigDecimal对象作为format()方法的参数，输出其格式化的货币值和百分比。

```java
@Test
public void format() {
    NumberFormat currency = NumberFormat.getCurrencyInstance(); //建立货币格式化引用
    NumberFormat percent = NumberFormat.getPercentInstance();  //建立百分比格式化引用
    percent.setMaximumFractionDigits(3); //百分比小数点最多3位

    BigDecimal loanAmount = new BigDecimal("15000.48"); //贷款金额
    BigDecimal interestRate = new BigDecimal("0.008"); //利率
    BigDecimal interest = loanAmount.multiply(interestRate); //相乘

    System.out.println("贷款金额:\t" + currency.format(loanAmount));
    System.out.println("利率:\t" + percent.format(interestRate));
    System.out.println("利息:\t" + currency.format(interest));
}
```

输出结果

```
贷款金额:	￥15,000.48
利率:	0.8%
利息:	￥120.00
```



#### DecimalFormat 格式化

BigDecimal格式化保留2为小数，不足则补0

```java
@Test
public void format2(){
    DecimalFormat df = new DecimalFormat("#.00");
    System.out.println(formatToNumber(new BigDecimal("3.435")));
    System.out.println(formatToNumber(new BigDecimal(0)));
    System.out.println(formatToNumber(new BigDecimal("0.00")));
    System.out.println(formatToNumber(new BigDecimal("0.001")));
    System.out.println(formatToNumber(new BigDecimal("0.006")));
    System.out.println(formatToNumber(new BigDecimal("0.206")));
}


/**
 * @desc 
 * 1. 0~1之间的BigDecimal小数，格式化后失去前面的0,则前面直接加上0。
 * 2. 传入的参数等于0，则直接返回字符串"0.00"
 * 3. 大于1的小数，直接格式化返回字符串
 * @return
 */
public String formatToNumber(BigDecimal obj) {
    DecimalFormat df = new DecimalFormat("#.00");
    if(obj.compareTo(BigDecimal.ZERO)==0) {
        return "0.00";
    }else if(obj.compareTo(BigDecimal.ZERO)>0&&obj.compareTo(new BigDecimal(1))<0){
        return "0"+df.format(obj).toString();
    }else {
        return df.format(obj).toString();
    }
}
```

###  BigDecimal 例子

最常见的例子就是精度是2(小数点后保留两位)，并且采用四舍五入的舍入模式(如果指定精度的笑一个数字大于等于5则向上取整，否则向下取整)

```java
@Test
public void examples() {
    BigDecimal bd1 = new BigDecimal("23.126");
    System.out.println("bd1 " + bd1.setScale(2, RoundingMode.HALF_UP).toString());
}
// 输出结果
bd1 23.13
```

因为精度设置为2之后，也就是小数点后两位的后一位数字是6大于等于5，所以向上取整，所以结果是` 23.13`

```java
BigDecimal bd1 = new BigDecimal("23.1236");
System.out.println("bd1 " + bd1.setScale(2, RoundingMode.HALF_UP).toString());
```

因为精度设置为2之后，也就是小数点后两位的后一位数字是3小于5，所以向下取整，所以结果是` 23.12`

```
BigDecimal bd1 = new BigDecimal("-15.567");
System.out.println("bd1 " + bd1.setScale(2, RoundingMode.HALF_UP).toString());
```

对于负数，也是同样的道理，所以输出是`bd1 -15.57`

### BigDecimal 的特性

#### 1. 没有重载操作符

在Java 中支持的(+, -, *, /)数学运算，BigDecimal并不支持，因为这些操作符是针对基本数据类型的，但是BigDecimal是引用类型，也就是基于对象和类的，因此BigDecimal提供了下面的方法

`add,subtract,multiply,and,divide`

例如乘法在BigDecimal的实现如下

```java
BigDecimal bd1 = new BigDecimal("15.567");

BigDecimal result = BigDecimal.valueOf(68).multiply(bd1);
System.out.println("result " + result.toString());
```

**Output**

```
result 1058.556
```

#### 2. 使用 compareTo() 来比较BigDecimals 而不是使用 equals()

需要注意如果你使用 equals()来比较两个BigDecimal数字，那只有当两个BigDecimal的值和精度都相同的时候equals()猜认为它们是相同的(因此2.0和2.00是不相同的)



```
BigDecimal bd1 = new BigDecimal("2.00");
BigDecimal bd2 = new BigDecimal("2.0");
System.out.println("bd1 equals bd2 - " + bd1.equals(bd2));
```

**Output**

```
bd1 equals bd2 - false
```

因此你应该使用**compareTo()**方法来比较两个**BigDecimal** 是否是相等的，**BigDecimal**实现了comparable接口并且提供了自己的compareTo方法，这个方法只会判断两个**BigDecimal**对象的值是否是相等的忽略了两个数字的精度(ike 2.0 和 2.00 相等的)

对于**bd1.compareTo(bd2)** 的返回值

- -1  bd1 小于 bd2.
- 0 两个相等的
- 1 bd1 大于 bd2.

```java
BigDecimal bd1 = new BigDecimal("2.00");
BigDecimal bd2 = new BigDecimal("2.0");
System.out.println("bd1 compareTo bd2 - " + bd1.compareTo(bd2));
```

**Output**

```
bd1 compareTo bd2 - 0
```

#### 3. BigDecimals 是不可变的

BigDecimal 对象是不可变的，所以是线程安全的,在进行每一次四则运算时，都会产生一个新的对象 ，所以在做加减乘除运算时要记得要保存操作后的值。

## 总结

1. 当我们在进行有着高精度的计算要求的时候不要使用double和float 因为它们有着精度丢失的问题
2. 如果使用BigDecimal的时候，不要选择double值作为初始化的值，因为它同样会引入精度的问题
3. 如果你使用BigDecimal时候设置了精度，那就同时提供舍入模式，告诉BigDecimal如何舍入从而提供你想要的精度
4. BigDecimal继承了Number类和实现了Comparable接口
5. BigDecimal 针对加减乘除提供可特定的方法，因为BigDecimal不支持(+, -, *, /)数学运算
6. BigDecimal 的对象是不可变的
7. BigDecimal因为创建对象开销的原因，所以很多操作都是比原生类型要慢一些的。