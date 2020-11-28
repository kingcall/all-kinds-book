String可以说是Java中使用最多最频繁、最特殊的类，因为同时也是字面常量，而字面常量包括基本类型、String类型、空类型。

## 一. String的使用

#### 1. String的不可变性

```
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

String对象一旦在堆中创建出来，就无法再修改。因为String对象放在char数组中，该数组由final关键字修饰，不可变。

#### 2. 定义一个字符串

```
/**
 * 定义一个字符串
 */
String str1 = "helloworld";
String str2 = "helloworld";
//也可以,但基本不这样写
String str3 = new String("helloworld");
System.out.println(str1 == str2);
System.out.println(str1 == str3);

//运行结果 true, false
```

上面三句代码怎么理解呢？这里需要先引入一个概念，**字符串常量池**。

**字符串常量池**是一块特殊的独立内存空间，放在Java Heap中 { 在Jdk7.0之前字符串常量池存放在PermGen中，Jdk7.0的时候移动到了Java Heap（在堆中仍然是独立的），Jdk8.0的时候去掉了PermGen，用Metaspace进行取代 } ，Java内存模型不是本章讨论的重点。

str1和str2引用的字符串字面量就在字符串常量池中，而str3引用的对象在Java Heap中。

怎么，还不太好理解？举个例子

> 工作一天，到下班时间了，准备看会儿金瓶.，算了，《三国演义》，打开小说网站，在线阅读；过了半个小时，女票回家了，看《三国演义》也是她想做的事儿，我看网址发给她，好，她也开始看了，再过半个小时，我爸回来了，他也是三国迷，但是他不喜欢在线看，因此在书店买了一本看。

上面提到的小说网站就是一个字符串常量池，包含了很多字符串字面量，如《三国演义》、《西游记》、《红楼梦》等，每个字符串字面量在常量池中保持独一份，无论谁进网站看《三国演义》都是同样的网址和同样的内容。

我和女票就是str1和str2，我们看的都是同一个网站的《三国演义》，不仅内容一样，引用的地址也一样（字符串常量池中保留了唯一的“helloworld”），因此str1 == str2 运行结果为true

而我爸就是str3，与我和女票都不一样，虽然看的内容也是《三国演义》，但是通过实体书籍来看，引用地址不一样，同时一本书籍不支持多个人同时看（字符串对象在java heap中，且每次new都会新建一个对象），因此str1 == str3 运行结果为false。

一个字符串字面量总是引用String类的同一个实例，因为被String.intern()方法限定了，同样我们可以调用该方法将堆中的String对象放到字符串常量池中，这样做可以提升内存使用效率，同时可以让所用使用者共享唯一的实例。

```
System.out.println(str1 == str3.intern());
//运行结果为true
```

那么该方法的实现逻辑是怎么样的呢，我们看一下源码

```
/** 
 * Returns a canonical representation for the string object. 
 * <p> 
 * A pool of strings, initially empty, is maintained privately by the 
 * class {@code String}. 
 * <p> 
 * When the intern method is invoked, if the pool already contains a 
 * string equal to this {@code String} object as determined by 
 * the {@link #equals(Object)} method, then the string from the pool is 
 * returned. Otherwise, this {@code String} object is added to the 
 * pool and a reference to this {@code String} object is returned. 
 * <p> 
 * It follows that for any two strings {@code s} and {@code t}, 
 * {@code s.intern() == t.intern()} is {@code true} 
 * if and only if {@code s.equals(t)} is {@code true}. 
 * <p> 
 * All literal strings and string-valued constant expressions are 
 * interned. String literals are defined in section 3.10.5 of the 
 * <cite>The Java&trade; Language Specification</cite>. 
 * 
 * @return  a string that has the same contents as this string, but is 
 *          guaranteed to be from a pool of unique strings. 
 */
 public native String intern();
```

我们发现这是一个native方法，看一下注释，发现str3.intern()方法大致流程是：

当执行intern()时，会先判断字符串常量池中是否含有相同（通过equals方法）的字符串字面量，如果有直接返回字符串字面量；如果不含，则将该字符串对象添加到字符串常量池中，同时返回该对象在字符串常量池的引用。

返回的引用需要赋值才可，否则还是会指向堆中的地址，即：

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

我们开始已经说了String是由final关键字修饰，不可变，那么此时在内存中如何体现呢？

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:12:13-1677914-20190622105259274-1149192402.png)

#### 4. String 对 “+” 的处理

```
String str7 = "good good" + " study";
String str8 = "good good study";
system.out.println(str7 == str8);
```

通过编译工具后得到

```
String str7 = "good good study";
String str8 = "good good study";
```

因此我们可以发现编译器在编译期间就是进行变量合并，而不会在常量池中创建三个对象 “good good”，“ study”，"good good study"。str7 == str8 运行结果 true。

但如果这样

```
String str9 = "good good ";
String str10 = str9 + "study";
system.out.println(str8 == str10);//false
```

这时运行结果为false，通过String变量 + 字符常量方式得到的结果会在堆中，不在常量池中，当然可以通过intern()方法放进常量池中，同时不仅“+”如此，调用substring()，toUpperCase()，trim()等返回的都是String在堆中的地址。

#### 5. String常用的方法

```
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

## 二. 总结

本文从String的不可变性，String创建时字面量和String对象的不同，字符串字面量常量池，字符串的内存结构，常用的String相关方法的描述，若有不对之处，请批评指正，望共同进步，谢谢！