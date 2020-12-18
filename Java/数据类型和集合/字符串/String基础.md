[TOC]

## Java String 初识

自从Java发布以来，基本数据类型就是Java语言的一部分，分别是***byte, short, int, long, char, float, double, boolean***.当前前面我们也学习了基本类型的包装类，知道了每种基本类型都有它的包装类型，**JAVA是面向对象的语言，很多类和方法中的参数都需使用对象(例如集合)，但基本数据类型却不是面向对象的，这就造成了很多不便**，所以有了包装类型以及自动模糊拆箱装箱。

String可以说是Java中使用最多最频繁、最特殊的类，因为同时也是字面常量，而字面常量包括基本类型、String类型、空类型。**其实如何判断一个类型是不是基本类型，其实只要看该类型名称首字母是否是大写的**(这是因为java 的类库严格遵循了驼峰命名的习惯，如果你对enum 有疑问，请查看[Java枚举—枚举初识](https://blog.csdn.net/king14bhhb/article/details/111224216))

因为String 的广泛使用，所以Java也针对String 做了很多的优化，例如线程安全的StingBuffer,快速拼接的StringBulider,还有JVM 的串池，正是因为如此，也衍生出了很多关于String类型的面试题，因为**事出必有妖**，谁让它特殊呢

### 一. String 的`说明书`

还是按照国际惯例，先看一下String 的`说明书`，其实往往很多时候你的困惑都在`说明书`里写着呢，但是在此之前我们还是先看一下它的继承关系，因为我们说了它是引用类型的



![image-20201217102443989](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/17/10:24:44-image-20201217102443989.png)

从这个继承关系我们看出它是可以序列化的，也是可以相互比较的，好的，接下来开启我们的String 之旅，`林尽水源，便得一山，山有小口，仿佛若有光。便舍船，从口入。初极狭，才通人。复行数十步，豁然开朗`,String 的说明书就是桃花源的入口



```java
/**
 * The {@code String} class represents character strings. All string literals in Java programs, such as {@code "abc"}, are
 * implemented as instances of this class.
 * {@code String}类表示字符串。Java程序中的所有字符串文本，例如{@code“abc”}，都是作为这个类的实例实现的。
 * Strings are constant; their values cannot be changed after they are created. 
 * Strings 是常量，它们的值在创建之后不能改变
 * String buffers support mutable strings. Because String objects are immutable they can be shared. For example:
 * String buffers 是可变的字符串，因为String 对象的不可变性，所以它们可以共享
 * The class {@code String} includes methods for examining individual characters of the sequence, 
 * for comparing strings, for searching strings, for extracting substrings, and for creating a
 * copy of a string with all characters translated to uppercase or to  lowercase. 
 * {@code String}类包含用于检查序列中单个字符的方法，这些方法主要用于comparing、searching、extracting substrings、创建一个副本
 * Case mapping is based on the Unicode Standard version specified by the {@link java.lang.Character Character} class.
 * 大小写映射基于在 java.lang.Character类中 指定的Unicode标准版本。
 * The Java language provides special support for the string concatenation operator (&nbsp;+&nbsp;), and for conversion of other objects to strings.
 * java 语言针对字符串的拼接和将其他对象转化成字符串提供了特殊的支持，例如字符串的拼接的”+“ 操作符
 * String concatenation is implemented through the {@code StringBuilder}(or {@code StringBuffer}) class and its {@code append} method.
 * 字符串的拼接是通过{@code StringBuilder}或者 {@code StringBuffer}的{@code append} 方法实现的
 * String conversions are implemented through the method {@code toString}, defined by {@code Object} and nherited by all classes in Java. 
 * 字符串转化是通过 {@code toString} 方法的，这个方法是定义在{@code Object} 类中的，并且被java所有的类继承了
 * For additional information on string concatenation and conversion, see Gosling, Joy, and Steele, <i>The Java Language Specification</i>.
 * 有关字符串连接和转换的其他信息可以查看The Java Language Specification，书我已经放在网上了(http://troubleclear.com/document/2981)
 * <p> Unless otherwise noted, passing a <tt>null</tt> argument to a constructor or method in this class will cause a {@link NullPointerException} to be thrown.
 * 除非另有说明，传递一个null 值给构造器或者其他方法会抛出NullPointerException 
 * <p>A {@code String} represents a string in the UTF-16 format
 * in which <em>supplementary characters</em> are represented by <em>surrogate
 * pairs</em> (see the section <a href="Character.html#unicode">Unicode
 * Character Representations</a> in the {@code Character} class for
 * more information).
 * String 是使用 UTF-16 编码的，补充字符由代理项对表示
 * Index values refer to {@code char} code units, so a supplementary character uses two positions in a {@code String}.
 * 指定下标的位置对应着一个字符，因此补充字符在{@code String}中使用两个位置。
 * <p>The {@code String} class provides methods for dealing with Unicode code points (i.e., characters), in addition to those for dealing with Unicode code units (i.e., {@code char} values).
 * 除了{@code char}处理Unicode code字符外 {@code String} 也提供了这样的方法
 */

public final class String  implements java.io.Serializable, Comparable<String>, CharSequence {
 ... ... 
}
```



#### 1. String 的主要内部构成

下面的变量value和hash其实很简单，但是这里单独提出来还是希望大家能留意一下，早年前出去面试，别人问看过java 的源代码吗，那我肯定回答看过啊，那别人又说既然看过源代码那就很多看过String 的源代码了，那你讲讲，当时那尴尬 :smile:

##### value

这就是string 的真实存储，也就是说String 其实是以字符数组进行存储的

```java
/** The value is used for character storage. */
private final char value[];
```



##### hash

字符串的hash值

```java
/** Cache the hash code for the string */
private int hash; // Default to 0
```



#### 2. String 的创建方法

1. **构造方法**:我们知道String 不是基本类型而是对象类型，那么我们自然可以按照创建对象类型的方式去创建一个字符串对象，也就是使用`new` 关键字，其实就是 这就是String的构造方法，例如`String str = new String(“abc”)`，String 提供了十来种构造方法
2. **字面量**：如果你对什么是字面量不清楚的话，没有关系，但是你可以想一下，你是如何创建基础数据类型变量的，例如`int i=10`,对于String 类型我们也可以这样创建例如`String str = “abc”;`

但是你要是认为字面量的这种方式就是创建String 对象的目的话，那你就错了，Java 提供的这种方式不单单是为了简化String 的创建，更主要的目的是为了和通过构造方法创建的这种方式进行区分，那区分出来的目的是什么呢？就是我们后面说的串池，因为它将这两种方式区分出来之后，让通过字面量的这种方式会走串池的这个设计，因为通过`new` 创建出来的String 会存储在堆里，并且有自己的空间，通过字面量的这种方式创建的字符串会被区别对待吗，会被放到公共的串池中

这也就说，如果两个字面量有相同的内容(字符串)，那么它们会占用串池中同一块存储空间，也就是指向同一个内容

```java
String str1 = "abc";
String str2 = "abc";
```

当你通过分配一个字符串字面量的方式创建一个字符串对象的时候，串池会先检测是否存在这样的字符串内容，如果存在则返回已经存在的字符串的引用给变量，如果不存在则该字面量加入到串池然后返回其引用，所以上面两个字符串变量共享了同一个在串池中的字面量，文章后面还会简单介绍一下串池，后面针对串池我还会单独写一篇文章



### 二 . String的使用

#### 1. String的不可变性

不可变性在注释中已经说明了，下面我们具体看一下

```java
/**
 * The {@code String} class represents character strings. All
 * string literals in Java programs, such as {@code "abc"}, are
 * implemented as instances of this class.
 * <p>
 * Strings are constant; their values cannot be changed after they
 * are created. String buffers support mutable strings.
 * Because String objects are immutable they can be shared. For example:
 * ...
 */

public final class String { 
	private final char value[];
}
```

String对象一旦在堆中创建出来，就无法再修改。那是因为String对象放在char数组中，该数组由final关键字修饰，不可变。这里有一点需要注意的是不可变的原因不是因为String类被final 修饰了，而是因为它的底层存储是一个不可变的数组,程序无法对它做出修改

像下面这个字符串类`BuerSting`它就是可变的，这里的可变是指的是它原来的值变掉了

```java
public final class BuerSting {
    private char value[];
    private int hash; // Default to 0

    public BuerSting(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }
    public BuerSting(String original) {
        this.value = original.toCharArray();
        this.hash = original.hashCode();
    }

    public void setValue(char[] value) {
        this.value = value;
    }
    public void setValue(String value) {
        this.value = value.toCharArray();
    }

    @Override
    public String toString() {
        return "BuerSting{" +
                "value=" + Arrays.toString(value) +
                '}';
    }

    public static void main(String[] args) {
        BuerSting s = new BuerSting("aaa");
        System.out.println(s);
        s.setValue("sss");
        System.out.println(s);
    }
}
// 输出结果
BuerSting{value=[a, a, a]}
BuerSting{value=[s, s, s]}
```

通过上面我们看到了，字符串其实是可以定义为可变的，但是java 为什么还是把它设计成不可变的呢，这也和串池有关，那是因为通过串池的这种设计方式，可能导致多个字符串变量，持有同一个字符串，那么如果是可变的话，通过一个变量改变了字符串的内容的话，就会导致其他字符串的变量的引用的内容发生变化，所以这就是字符串为什么设计成不可变的。

```java
@Test
public void immutable() {
    String str = "hello";
    str.concat("world");
    System.out.println("Value of str:" + str);
}
// 输出结果
Value of str:hello
```

可以看出当你通过字符串的方法去修改字符串的时候，它只会返回新的字符串新的字符串包含了修改后的内容，原来的内容不会变

到这里你是不是觉得完了，其实还没有，因为这个时候你还可以通过继承的方式干点坏事

##### final class

1. 你可通过继承String 类，然后提供可以修改内容的方法，这个时候String类型的不可变性就会被打破
2. 还有就是你可以重写hashcode 方法和equals 方法，这个时候就会导致串池里的内容重复或者是不同内容有相同的hash值



##### 线程安全

因为是不可变对象，所以String 是线程安全的

#### 2. 定义一个字符串

```java
public class MyString {
    public static void main(String[] args) {
        String str1 = "i";
        String str2 = "i";
        String str2 = new String("i");
        System.out.println(str1 == str2);
        System.out.println(str1 == str3);
        System.out.println(str1 == str3.intern());
    }
}
// 运行结果
true
false
true
```

上面三句代码怎么理解呢？这里需要先引入一个概念，**字符串常量池**。

**字符串常量池**是一块特殊的独立内存空间，放在Java Heap中 { 在Jdk7.0之前字符串常量池存放在PermGen中，Jdk7.0的时候移动到了Java Heap（在堆中仍然是独立的），Jdk8.0的时候去掉了PermGen，用Metaspace进行取代 } ，Java内存模型不是本章讨论的重点,关于**字符串常量池**在内存中的位置可以看关于JVM 的文章

str1和str2引用的字符串字面量就在字符串常量池中，而str3引用的对象在Java Heap中。怎么，还不太好理解？举个例子

> 工作一天，到下班时间了，准备看会儿金瓶.，算了，《三国演义》，打开小说网站，在线阅读；过了半个小时，女票回家了，看《三国演义》也是她想做的事儿，我看网址发给她，好，她也开始看了，再过半个小时，我爸回来了，他也是三国迷，但是他不喜欢在线看，因此在书店买了一本看。

上面提到的小说网站就是一个字符串常量池，包含了很多字符串字面量，如《三国演义》、《西游记》、《红楼梦》等，每个字符串字面量在常量池中保持独一份，无论谁进网站看《三国演义》都是同样的网址和同样的内容。

我和女票就是str1和str2，我们看的都是同一个网站的《三国演义》，不仅内容一样，引用的地址也一样（字符串常量池中保留了唯一的“helloworld”），因此str1 == str2 运行结果为true

而我爸就是str3，与我和女票都不一样，虽然看的内容也是《三国演义》，但是通过实体书籍来看，引用地址不一样，同时一本书籍不支持多个人同时看（字符串对象在java heap中，且每次new都会新建一个对象），因此str1 == str3 运行结果为false。

所以我们回过头再来看一下，一个字符串字面量总是引用String类的同一个实例，因为被String.intern()方法限定了，**同样我们可以调用该方法将堆中的String对象放到字符串常量池中，这样做可以提升内存使用效率，同时可以让所用使用者共享唯一的实例**。

```javascript
System.out.println(str1 == str3.intern());
//运行结果为true
```

那么该方法的实现逻辑是怎么样的呢，我们看一下源码

```java
/** 
 * Returns a canonical representation for the string object. 
 * 返回字符串对象的规范表示形式。
 * A pool of strings, initially empty, is maintained privately by the  class {@code String}. 
 * 一个字符串池，一开始是空的，由字符串类私有维护
 * When the intern method is invoked, if the pool already contains a  string equal to this {@code String} object as determined by 
 * the {@link #equals(Object)} method, then the string from the pool is  returned. 
 * 当intern方法被调用的时候，如果串池中已经包含被equals方法认定相等的字符串，则串池中的字符串被返回
 * Otherwise, this {@code String} object is added to the  pool and a reference to this {@code String} object is returned. 
 * 否则当前String对象被加入串池，并且返回该字符串的引用
 * It follows that for any two strings {@code s} and {@code t},  {@code s.intern() == t.intern()} is {@code true}  if and only if {@code s.equals(t)} is {@code true}. 
 * 因此，对于任意两个字符串{@codes s}和{@code t}，{@codes s.intern（）==t.intern（）}是{@code true}当且仅当{@codes s.equals（t）}是{@code true}。
 * All literal strings and string-valued constant expressions are  interned. String literals are defined in section 3.10.5 of the <cite>The Java&trade; Language Specification</cite>. 
 * 所有的文字字符串和字符串值常量表达式都是内部的
 * @return  a string that has the same contents as this string, but is 
 *          guaranteed to be from a pool of unique strings. 
 */
 public native String intern();
```

我们发现这是一个native方法，看一下注释，发现这个方法大致流程是：

1、当执行intern()时，会先判断字符串常量池中是否含有相同（通过equals方法）的字符串字面量，如果有直接返回字符串字面量；

2、如果不含，则将该字符串对象添加到字符串常量池中，同时返回该对象在字符串常量池的引用。返回的引用需要赋值才可，否则还是会指向堆中的地址，即：

```
String str4 = new String("helloChina");
System.out.println(str4.intern() == str4);//false
str4 = str4.intern();
String str5 = "helloChina";
String str6 = "helloZhonghua"
System.out.println(str4 == str5);//true
```

下面我们看一下内存结构
![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:12:13-1677914-20190622105249234-2108048903.png)

#### 3. 再次赋值给已定义的字符串

```
str6 = "helloHuaxia";
```

我们开始已经说了String是由final关键字修饰，不可变，那么此时在内存中如何体现呢，可以看出是不该改变原来的字符串`helloZhonghua` 而是创建了新的字符串`helloHuaxia`

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:12:13-1677914-20190622105259274-1149192402.png)

#### 4. String 对 “+” 的处理

```
String str7 = "good good" + " study";
String str8 = "good good study";
System.out.println(str7 == str8);
```

通过编译工具后得到

```
String str7 = "good good study";
String str8 = "good good study";
```

因此我们可以发**现编译器在编译期间就是进行变量合并**，而不会在常量池中创建三个对象 “good good”，“ study”，"good good study"。str7 == str8 运行结果 true。但如果这样

```java
String str9 = "good good ";
String str10 = str9 + "study";
system.out.println(str8 == str10);//false
```

这时运行结果为false，**通过String变量 + 字符常量方式得到的结果会在堆中，不在常量池中**，当然可以通过intern()方法放进常量池中，**同时不仅“+”如此，调用substring()，toUpperCase()，trim()等返回的都是String在堆中的地址**。

#### 5. String常用的方法

```java
//str1 == "hello,world ";

//获取长度
str1.length()//12;

//截取位置2到5之间的字符串（包括位置2，不包括位置5，从0开始）
str1.substring(2,5);//"llo"

//判断是否含有字符串“ello”
str1.contains("ello");//true,通过indexOf实现

//获取ello在str1中的开始位置
str1.indexOf("ello");//1

//将字符串转化为字符串数据
str1.split(",");//["hello","world"]

//去掉字符串两侧空格
str1.trim();//"hello,world"
```

##  总结

本文从String的不可变性，String创建时字面量和String对象的不同，字符串字面量常量池，字符串的内存结构，常用的String相关方法的描述，若有不对之处，请批评指正，望共同进步，谢谢！

1. Java 语言针对字符串提供了特殊的支持

   1. 字符串的拼接和将其他对象转化成字符串提供了特殊的支持
   2. 创建String 对象提供了特殊支持，那就是通过字面量的这种方式，而这种方式是为了实现串池的设计与使用
   3. String 对象是不可变的，所以是线程安全的
   4. "+" 操作符被重写了，用来拼接字符串
   5. Java7 之后 String 对象可以被用在switch case  中

2. Java为了优化字符串的性能，提供了字符串常量池

3. String 不可变的原因是底层存储的字节数组是final 修饰的

4. 提供了一个文档[The Java Language Specification](http://troubleclear.com/document/2981) 大家有需要可以去查看

   





