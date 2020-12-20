[TOC]

## 字符串常量池

作为最基础用的最多的引用数据类型，Java 设计者为String 提供了各种优化，其中就有为 String 提供了字符串常量池以提高其性能，主要就是为了降低内存开销，那么字符串常量池的具体原理是什么，我们带着以下三个问题，去理解字符串常量池：

1、字符串常量池的设计意图是什么？

2、字符串常量池在哪里？

3、如何操作字符串常量池？

从上一节[String基础](https://editor.csdn.net/md/?articleId=111314482) 到这里，我们一直在说字符串常量池或者串池，但是没有解释为什么叫字符串常量池，首先和所有的池子一样，例如线程池、数据库连接池都是为了性能，因为是存储字符串的所以叫字符串池，因为存储的字符串是不可变的，也就是常量，所以叫字符串常量池

### 一道Java 面试题的锅

我想所有 Java 程序员都曾被这个 new String 的问题困扰过，这是一道高频的 Java 面试题，但可惜的是网上众说纷纭，竟然找不到标准的答案。有人说创建了 1 个对象，也有人说创建了 2 个对象，还有人说可能创建了 1 个或 2 个对象，但谁都没有拿出干掉对方的证据，这就让我们这帮吃瓜群众们陷入了两难之中，不知道到底该信谁得。

以目前的情况来看，关于 `new String("xxx")` 创建对象个数的答案有 3 种：

1. 有人说创建了 1 个对象；
2. 有人说创建了 2 个对象；
3. 有人说创建了 1 个或 2 个对象。

**而出现多个答案的关键争议点在「字符串常量池」上**，有的说 new 字符串的方式会在常量池创建一个字符串对象，有人说 new 字符串的时候并不会去字符串常量池创建对象，而是在调用 `intern()` 方法时，才会去字符串常量池检测并创建字符串。

那我们就先来说说这个「字符串常量池」。

### 字符串常量池初识

前面我们说到，你要是认为字面量的这种方式就是创建String 对象的目的话，那你就错了，Java 提供的这种方式不单单是为了简化String 的创建，更主要的目的是为了和通过构造方法创建的这种方式进行区分，那区分出来的目的是什么呢？就是我们后面说的串池，因为它将这两种方式去分出来之后，让通过字面量的这种方式会走串池的这个设计，因为通过`new` 创建出来的String 会存储在堆里，并且有自己的空间，通过字面量的这种方式创建的字符串会被区别对待吗？会被放到公共的串池中这也就说，如果两个字面量有相同的内容(字符串)，那么其实它们返回个字符串对象的是同一个地址，**这就是节约内存的原理——没有创建新的，返回了已有对象的地址**

和其对象的创建和分配一样，通过`new` 创建出来的字符串对象是在堆上分配的，需要耗费高昂的时间和空间为代价，作为最基础的数据类型，如果大量频繁的创建字符串，会极大程度地影响程序的性能，因此 JVM 为了提高性能和减少内存开销引入了字符串常量池（Constant Pool Table）的概念，字符串常量池相当于给字符串开辟**一个常量池空间类似于缓存区**，对于直接赋值的字符串（String s="xxx"）来说，在每次创建字符串时优先使用已经存在字符串常量池的字符串，如果字符串常量池没有相关的字符串，会先在字符串常量池中创建该字符串，然后将引用地址返回变量，如下图所示：

![image-20201217215632399](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/17/21:56:33-image-20201217215632399.png)

实现该优化的基础是因为字符串是不可变的，可以不用担心数据冲突进行共享，运行时实例创建的全局字符串常量池中有一个表，总是为池中每个唯一的字符串对象维护一个引用,这就意味着它们一直引用着字符串常量池中的对象，**所以，在常量池中的这些字符串不会被垃圾收集器回收**,这个时候你要记住这一点，因为缓存虽好，但是可能引起内存泄漏的问题，除了这个问题还有一个问题那就是，如果 Pool 中对象过多，可能导致 YGC 变长，因为 YGC 的时候，需要扫描 String Pool，更多细节请看[String.intern()导致的YGC不断变长](http://troubleclear.com/post/168)

前面我们学习包装类的时候也说过包装类也是有缓存的，因为创建包装类的对象也是一个代价比较高昂的操作，需要注意的是基础类型包装类的缓存池使用一个数组进行缓存，而 String 类型，JVM 内部使用 HashTable 进行缓存，我们知道，HashTable 的结构是一个数组，数组中每个元素是一个链表。和我们平时使用的 HashTable 不同，JVM 内部的这个 HashTable 是不可以动态扩容的。关于HashTable 你可以看[深度剖析HashTable](https://blog.csdn.net/king14bhhb/article/details/110356606),当然最终的形态也是和HashTable 一致的，如下所示

![image-20201219102806212](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/19/10:28:07-image-20201219102806212.png)



以上说法可以通过如下代码进行证明：

```java
public class StringExample {
    public static void main(String[] args) {
        String s1 = "Java";
        String s2 = "Java";
        System.out.println(s1 == s2);
    }
}
```

以上程序的执行结果为：`true`，说明变量 s1 和变量 s2 指向的是同一个地址。在这里我们顺便说一下字符串常量池的再不同 JDK 版本的变化。



#### 常量池的内存布局



从**JDK 1.7 之后把永久代换成的元空间，把字符串常量池从方法区移到了 Java 堆上**。需要注意的是不论是永久代还是元空间都是对方法区的实现，在JVM 规范中并没有规定方法区的实现

在 Java 6 中，String Pool 置于 PermGen Space 中，PermGen 有一个问题，那就是它是一个固定大小的区域，虽然我们可以通过 `-XX:MaxPermSize=N` 来设置永久代的空间大小，但是不管我们设置成多少，它终归是固定的。

所以，在 Java 6 中，我们应该尽量小心使用 String.intern() 方法，否则容易导致 OutOfMemoryError。到了 Java 7，大佬们已经着手去掉 PermGen Space 了，首先，就是将 String Pool 移到了堆中。把 String Pool 放到堆中，即使堆的大小也是固定的，但是这个时候，对于应用调优工作，只需要调整堆大小就行了。

前面我们说了 String Pool 使用一个 HashTable 来实现，这个 HashTable **不可以扩容**，也就意味着极有可能出现单个 bucket 中的链表很长，导致性能降低。在 Java 6 中，这个 HashTable 固定的 bucket 数量是 1009，后来添加了选项（**-XX:StringTableSize=N**）可以配置这个值。到 Java 7(7u40)，大佬们提高了这个默认值到 60013，Java 8 依然也是使用这个值，对于绝大部分应用来说，这个值是足够用的。当然，如果你会在代码中大量使用 String#intern()，那么有必要手动设置一下这个值。

**JDK 1.8 与 JDK 1.7 最大的区别是 JDK 1.8 将永久代取消，并设立了元空间**。官方给的说明是**由于永久代内存经常不够用或发生内存泄露**，会爆出 java.lang.OutOfMemoryError: PermGen 的异常，所以把将永久区废弃而改用元空间了，**改为了使用本地内存空间**，官网解释详情：http://openjdk.java.net/jeps/122



#### 字符串常量池的控制

可以通过`-XX:StringTableSize`参数进行控制大小，可以使用`-XX:+PrintStringTableStatistics`参数，让JVM退出时打印出常量池使用情况。

这里首先我们什么都不干，就先看一下JVM 退出的时候常量池的情况

```java
public class StringPool {
    public static void main(String[] args) {

    }
}
```

```
SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     11852 =    284448 bytes, avg  24.000
Number of literals      :     11852 =    459800 bytes, avg  38.795
Total footprint         :           =    904336 bytes
Average bucket size     :     0.592
Variance of bucket size :     0.592
Std. dev. of bucket size:     0.770
Maximum bucket size     :         6
// StringTable
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000
Number of entries       :       849 =     20376 bytes, avg  24.000
Number of literals      :       849 =     57464 bytes, avg  67.684
Total footprint         :           =    557944 bytes
Average bucket size     :     0.014
Variance of bucket size :     0.014
Std. dev. of bucket size:     0.119
Maximum bucket size     :         2
```

可以这里的字面量的个数是849，但是这个值不太固定多次运行发现结果不一样，本来我想证明的是我添加一个字符串，Number of literals  也会加1的效果，但是因为初始值每次不固定，所以我就测不了，知识有限就说到这里了，如果你们有什么好的办法请一定要告知我哦。

下面我们测试一下，常量池大小对程序性能的影响，首先我们看一下代码

```java
int size=0;
@Before
public void setUp(){
    size = 4000000;
}

@Test
public   void test() {
    final List<String> lst = new ArrayList<String>(size);
    long start = System.currentTimeMillis();
    for (int i = 0; i < size; ++i) {
        final String str = "Very very very very very very very very very very very very very very very long string: " + i;
        lst.add(str.intern());

        if (i % 200000 == 0) {
            System.out.println(i + 200000 + "; time = " + (System.currentTimeMillis() - start) / 1000.0 + " sec");
            start = System.currentTimeMillis();
        }
    }
    System.out.println("Total length = " + lst.size());
}
```

下面我们看一下输出结果

| 默认大小60013                                                | 修改后大小400031                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 200000; time = 0.0 sec<br/>400000; time = 0.125 sec<br/>600000; time = 0.107 sec<br/>800000; time = 0.098 sec<br/>1000000; time = 0.103 sec<br/>1200000; time = 0.125 sec<br/>1400000; time = 0.145 sec<br/>1600000; time = 0.172 sec<br/>1800000; time = 0.344 sec<br/>2000000; time = 0.183 sec<br/>2200000; time = 0.198 sec<br/>2400000; time = 0.22 sec<br/>2600000; time = 0.238 sec<br/>2800000; time = 0.264 sec<br/>3000000; time = 0.286 sec<br/>3200000; time = 0.309 sec<br/>3400000; time = 0.346 sec<br/>3600000; time = 0.367 sec<br/>3800000; time = 0.39 sec<br/>4000000; time = 0.417 sec<br/>Total length = 4000000<br/><br/><br/>SymbolTable statistics:<br/>Number of buckets       :     20011 =    160088 bytes, avg   8.000<br/>Number of entries       :     20751 =    498024 bytes, avg  24.000<br/>Number of literals      :     20751 =    779440 bytes, avg  37.562<br/>Total footprint         :           =   1437552 bytes<br/>Average bucket size     :     1.037<br/>Variance of bucket size :     1.030<br/>Std. dev. of bucket size:     1.015<br/>Maximum bucket size     :         7<br/>StringTable statistics:<br/>Number of buckets       :     60013 =    480104 bytes, avg   8.000<br/>Number of entries       :   4003339 =  96080136 bytes, avg  24.000<br/>Number of literals      :   4003339 = 928145992 bytes, avg 231.843<br/>Total footprint         :           = 1024706232 bytes<br/>Average bucket size     :    66.708<br/>Variance of bucket size :    51.320<br/>Std. dev. of bucket size:     7.164<br/>Maximum bucket size     :        89 | 200000; time = 0.0 sec<br/>400000; time = 0.128 sec<br/>600000; time = 0.097 sec<br/>800000; time = 0.079 sec<br/>1000000; time = 0.076 sec<br/>1200000; time = 0.082 sec<br/>1400000; time = 0.083 sec<br/>1600000; time = 0.085 sec<br/>1800000; time = 0.085 sec<br/>2000000; time = 0.235 sec<br/>2200000; time = 0.071 sec<br/>2400000; time = 0.075 sec<br/>2600000; time = 0.07 sec<br/>2800000; time = 0.07 sec<br/>3000000; time = 0.069 sec<br/>3200000; time = 0.076 sec<br/>3400000; time = 0.073 sec<br/>3600000; time = 0.076 sec<br/>3800000; time = 0.079 sec<br/>4000000; time = 0.076 sec<br/>Total length = 4000000<br/><br/><br/>SymbolTable statistics:<br/>Number of buckets       :     20011 =    160088 bytes, avg   8.000<br/>Number of entries       :     20751 =    498024 bytes, avg  24.000<br/>Number of literals      :     20751 =    779440 bytes, avg  37.562<br/>Total footprint         :           =   1437552 bytes<br/>Average bucket size     :     1.037<br/>Variance of bucket size :     1.030<br/>Std. dev. of bucket size:     1.015<br/>Maximum bucket size     :         7<br/>StringTable statistics:<br/>Number of buckets       :    400031 =   3200248 bytes, avg   8.000<br/>Number of entries       :   4003341 =  96080184 bytes, avg  24.000<br/>Number of literals      :   4003341 = 928146128 bytes, avg 231.843<br/>Total footprint         :           = 1027426560 bytes<br/>Average bucket size     :    10.008<br/>Variance of bucket size :     3.495<br/>Std. dev. of bucket size:     1.870<br/>Maximum bucket size     :        20 |

可以看到修改后的的程序每次插入200000条需要的时间更短，性能更好，所以我们可以认为合适地修改串池大小可以提高我们程序的性能

### 答案揭秘(理论认证)

认为 new 方式创建了 1 个对象的人认为，new String 只是在堆上创建了一个对象，只有在使用 `intern()` 时才去常量池中查找并创建字符串。

认为 new 方式创建了 2 个对象的人认为，new String 会在堆上创建一个对象，并且在字符串常量池中也创建一个字符串。

认为 new 方式有可能创建 1 个或 2 个对象的人认为，new String 会先去常量池中判断有没有此字符串，如果有则只在堆上创建一个字符串并且指向常量池中的字符串，如果常量池中没有此字符串，则会创建 2 个对象，先在常量池中新建此字符串，然后把此引用返回给堆上的对象，如下图所示：
![image-20201219122603897](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/19/12:26:12-12:26:05-image-20201219122603897.png)

正确的答案是什么呢？还记得我们前面关于使用字符串字面量的方式创建字符串对象吗？其实到这里我们不用论证都可以猜出来，Java 特意提供了使用字面量的方式来创建字符串对象，那肯定是不希望破坏`new` 关键字的一致性——在堆上分配，所以我们认为答案是一种，而且针对字符串对象提供了intern() 方法，可以看做是对`new` 这种方式的一种扩展，也就是说非常量池中的字符串对象，也可以通过intern() 方法放入常量池，既然如此`new` 关键字肯定不会放入常量池了，否则就不用提供intern()方法了，也不用提供使用字面量创建字符创变量的方法了。

那么想法对吗，我们下面来从实现方面进行分析一下

### 技术论证

每个 java 文件编译为 class 文件后，都将产生当前类独有的常量池，我们称之为静态常量池。class 文件中的常量池包含两部分：**字面值**（literal）和**符号引用**（Symbolic Reference）。其中字面值可以理解为 java 中定义的字符串常量、final 常量等；符号引用指的是一些字符串，这些字符串表示当前类引用的外部类、方法、变量等的引用地址的抽象表示形式，在类被jvm装载并第一次使用这些符号引用时，这些符号引用将会解析为直接引用。符号常量包含：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

jvm在进行类装载时，将class文件中常量池部分的常量加载到方法区中，此时方法区中的保存常量的逻辑区域称之为运行时常量区。

方法的调用、成员变量的访问最终都是通过运行时常量池来查找具体地址的

解铃还须系铃人，回到问题的那个争议点上，new String 到底会不会在常量池中创建字符呢？我们通过反编译下面这段代码就可以得出正确的结论，代码如下：

```java
public class StringTableExample {
    public static void main(String[] args) {
        String s1 = new String("Hello World");
    }
}
```



首先我们使用 `javac StringTableExample.java` 编译代码，然后我们再使用 `javap -verbose StringTableExampl.class` 查看反编译的结果，相关信息如下：

```
Classfile /datastructure/str/StringTableExample.class
  Last modified 2020-12-19; size 590 bytes
  MD5 checksum 4e93cb3151c78dc81b015f7637c6c166
  Compiled from "StringTableExample.java"
public class datastructure.str.StringTableExample
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
// 常量池
Constant pool:
   #1 = Methodref          #6.#22         // java/lang/Object."<init>":()V
   #2 = Class              #23            // java/lang/String
   #3 = String             #24            // Hello World
   #4 = Methodref          #2.#25         // java/lang/String."<init>":(Ljava/lang/String;)V
   #5 = Class              #26            // datastructure/str/StringTableExample
   #6 = Class              #27            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Ldatastructure/str/StringTableExample;
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               args
  #17 = Utf8               [Ljava/lang/String;
  #18 = Utf8               s1
  #19 = Utf8               Ljava/lang/String;
  #20 = Utf8               SourceFile
  #21 = Utf8               StringTableExample.java
  #22 = NameAndType        #7:#8          // "<init>":()V
  #23 = Utf8               java/lang/String
  #24 = Utf8               Hello World
  #25 = NameAndType        #7:#28         // "<init>":(Ljava/lang/String;)V
  #26 = Utf8               datastructure/str/StringTableExample
  #27 = Utf8               java/lang/Object
  #28 = Utf8               (Ljava/lang/String;)V
{
  public datastructure.str.StringTableExample();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Ldatastructure/str/StringTableExample;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #2                  // class java/lang/String
         3: dup
         4: ldc           #3                  // String Hello World
         6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
         9: astore_1
        10: return
      LineNumberTable:
        line 5: 0
        line 6: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  args   [Ljava/lang/String;
           10       1     1    s1   Ljava/lang/String;
}
SourceFile: "StringTableExample.java"

```



其中 `Constant pool` 表示字符串常量池，我们在字符串编译期的字符串常量池中找到了我们创建变量 `String s1 = new String("Hello World");` 时定义的"Hello World"字符,`  #24 = Utf8 Hello World`可以看出，也就是**在编译期 new 方式创建的字符串就会被放入到编译期的字符串常量池中**，也就是说 new String的方式会首先去判断字符串常量池，如果没有就会新建字符串那么就会创建 2 个对象，如果已经存在就只会在堆中创建一个对象指向字符串常量池中的字符串。

所以看出，new 出来的字符串还是会在常量池中创建字符串，也就是说答案可能是一个也可能是两个

那么问题来了，以下这段代码的执行结果为 true 还是 false？

```
String s1 = new String("Hello World"");
String s2 = new String("Hello World"");
System.out.println(s1 == s2);
```

既然 new String 会在常量池中创建字符串，那么执行的结果就应该是 true 了。其实并不是，这里对比的变量 s1 和 s2 堆上地址，因为堆上的地址是不同的，所以结果一定是 false，如下图所示：

![字符串引用.png](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/17/15:23:25-1460000022429158.png)
从图中可以看出 s1 和 s2 的引用一定是相同的，而 s3 和 s4 的引用是不同的，对应的程序代码如下：

```java
public static void main(String[] args) {
    String s1 = "Java";
    String s2 = "Java";
    String s3 = new String("Java");
    String s4 = new String("Java");
    System.out.println(s1 == s2);
    System.out.println(s3 == s4);
}
// 输出结果
true
false
```



### 扩展知识

我们知道 String 是 final 修饰的，也就是说一定被赋值就不能被修改了。但编译器除了有字符串常量池的优化之外，还会对编译期可以确认的字符串进行优化，例如以下代码：

```
public static void main(String[] args) {
    String s1 = "abc";
    String s2 = "ab" + "c";
    String s3 = "a" + "b" + "c";
    System.out.println(s1 == s2);
    System.out.println(s1 == s3);
}
```

按照 String 不能被修改的思想来看，s2 应该会在字符串常量池创建两个字符串“ab”和“c”，s3 会创建三个字符串，他们的引用对比结果也一定是 false，但其实不是，他们的结果都是 true，这是编译器优化的功劳。

首先我们使用 `javac StringTableExample.java` 编译代码，然后我们再使用 `javap -c StringTableExampl.class` 查看反编译的结果

```
Compiled from "StringTableExample.java"
public class datastructure.str.StringTableExample {
  public datastructure.str.StringTableExample();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String abc
       2: astore_1
       3: ldc           #2                  // String abc
       5: astore_2
       6: ldc           #2                  // String abc
       8: astore_3
       9: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      12: aload_1
      13: aload_2
      14: if_acmpne     21
      17: iconst_1
      18: goto          22
      21: iconst_0
      22: invokevirtual #4                  // Method java/io/PrintStream.println:(Z)V
      25: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      28: aload_1
      29: aload_3
      30: if_acmpne     37
      33: iconst_1
      34: goto          38
      37: iconst_0
      38: invokevirtual #4                  // Method java/io/PrintStream.println:(Z)V
      41: return
}

```

从 Code 3、6 可以看出字符串都被编译器优化成了字符串“abc”了,也就是提前拼接了

## 总结

本文我们通过 `javap -v XXX` 的方式查看编译的代码发现 new String 首次会在字符串常量池中创建此字符串，那也就是说，通过 new 创建字符串的方式可能会创建 1 个或 2 个对象，如果常量池中已经存在此字符串只会在堆上创建一个变量，并指向字符串常量池中的值，如果字符串常量池中没有相关的字符，会先创建字符串在返回此字符串的引用给堆空间的变量。我们还介绍了字符串常量池在 JDK 1.7 和 JDK 1.8 的变化以及编译器对确定字符串的优化，下面是具体的知识点

JVM所使用的内存中，字符串作为一种特殊的基础数据类型，**占据了大量的内存，且字符串有着大量的重复**。由于字符串具体不可变性，因此使用`String Pool`对于同样的字符串做一个缓存，防止多次分配内存，从而提高内存利用率。

`String Pool`在JDK当中是一个类似HashTable的结构，其特点线程安全，不可扩容，但是可以rehash

可以通过`-XX:StringTableSize`参数进行控制大小如果你对程序有大量的`String.intern()`调用，可以使用`-XX:+PrintStringTableStatistics`参数，让JVM退出时打印出常量池使用情况。

StringTableSize，在 Java 6 中，是 1009；在 Java 7 和 Java 8 中，默认都是 60013，如果有必要请自行扩大这个值(因为字符串常量池是左右缓存用的，所以叫`String Pool`，因为是线上使用HashTable 实现的，所以参数叫StringTableSize)

