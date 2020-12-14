# Java EnumSet

#### In this tutorial, we will learn about the Java EnumSet class and its various methods with the help of examples.

The `EnumSet` class of the Java collections framework provides a set implementation of elements of a single enum.

Before you learn about EnumSet, make sure to know about [Java Enums](https://www.programiz.com/java-programming/enums).

It implements the [Set interface](https://www.programiz.com/java-programming/set).



![Java EnumSet class implements the Java Set interface.](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/09:34:24-java-enumset.png)



------

## Creating EnumSet

In order to create an enum set, we must import the `java.util.EnumSet` package first.

Unlike other set implementations, the enum set does not have public constructors. We must use the predefined methods to create an enum set.

### 1. Using allOf(Size)

The `allof()` method creates an enum set that contains all the values of the specified enum type Size. For example,

```
import java.util.EnumSet;

class Main {
    // an enum named Size
    enum Size {
        SMALL, MEDIUM, LARGE, EXTRALARGE
    }
    
    public static void main(String[] args) {

        // Creating an EnumSet using allOf()
        EnumSet<Size> sizes = EnumSet.allOf(Size.class);

        System.out.println("EnumSet: " + sizes);
    }

}
```

**Output**

```
EnumSet: [SMALL, MEDIUM, LARGE, EXTRALARGE]
```

Notice the statement,

```
EnumSet<Size> sizes = EnumSet.allOf(Size.class);
```

Here, Size.class denotes the Size enum that we have created.

------

### 2. Using noneOf(Size)

The `noneOf()` method creates an empty enum set. For example,

```
import java.util.EnumSet;

class Main {

     // an enum type Size
    enum Size {
        SMALL, MEDIUM, LARGE, EXTRALARGE
    }

    public static void main(String[] args) {

        // Creating an EnumSet using noneOf()
        EnumSet<Size> sizes = EnumSet.noneOf(Size.class);

        System.out.println("Empty EnumSet: " + sizes);
    }
}
```

**Output**

```
Empty EnumSet : []
```

Here, we have created an empty enum named sizes.

**Note**: We can only insert elements of enum type Size in the above program. It's because we created our empty enum set using Size enum.

------

### 3. Using range(e1, e2) Method

The `range()` method creates an enum set containing all the values of an enum between e1 and e2 including both values. For example,

```
import java.util.EnumSet;

class Main {

    enum Size {
        SMALL, MEDIUM, LARGE, EXTRALARGE
    }

    public static void main(String[] args) {

        // Creating an EnumSet using range()
        EnumSet<Size> sizes = EnumSet.range(Size.MEDIUM, Size.EXTRALARGE);

        System.out.println("EnumSet: " + sizes);
    }
}
```

**Output**

```
EnumSet: [MEDIUM, LARGE, EXTRALARGE]
```

------

### 4. Using of() Method

The `of()` method creates an enum set containing the specified elements. For example,

```
import java.util.EnumSet;

class Main {

    enum Size {
        SMALL, MEDIUM, LARGE, EXTRALARGE
    }

    public static void main(String[] args) {

        // Using of() with a single parameter
        EnumSet<Size> sizes1 = EnumSet.of(Size.MEDIUM);
        System.out.println("EnumSet1: " + sizes1);

        EnumSet<Size> sizes2 = EnumSet.of(Size.SMALL, Size.LARGE);
        System.out.println("EnumSet2: " + sizes2);
    }
}
```

**Output**

```
EnumSet1: [MEDIUM]
EnumSet2: [SMALL, LARGE]
```

------

## Methods of EnumSet

The `EnumSet` class provides methods that allow us to perform various elements on the enum set.

------

## Insert Elements to EnumSet

- `add()` - inserts specified enum values to the enum set
- `addAll()` inserts all the elements of the specified collection to the set

For example,

```
import java.util.EnumSet;

class Main {

    enum Size {
        SMALL, MEDIUM, LARGE, EXTRALARGE
    }

    public static void main(String[] args) {

        // Creating an EnumSet using allOf()
        EnumSet<Size> sizes1 = EnumSet.allOf(Size.class);

        // Creating an EnumSet using noneOf()
        EnumSet<Size> sizes2 = EnumSet.noneOf(Size.class);

        // Using add method
        sizes2.add(Size.MEDIUM);
        System.out.println("EnumSet Using add(): " + sizes2);

        // Using addAll() method
        sizes2.addAll(sizes1);
        System.out.println("EnumSet Using addAll(): " + sizes2);
    }
}
```



**Output**

```
EnumSet using add(): [MEDIUM]
EnumSet using addAll(): [SMALL, MEDIUM, LARGE, EXTRALARGE]
```

In the above example, we have used the `addAll()` method to insert all the elements of an enum set sizes1 to an enum set sizes2.

It's also possible to insert elements from other collections such as `ArrayList`, `LinkedList`, etc. to an enum set using `addAll()`. However, all collections should be of the same enum type.

------

## Access EnumSet Elements

To access elements of an enum set, we can use the `iterator()` method. In order to use this method, we must import the `java.util.Iterator` package. For example,

```
import java.util.EnumSet;
import java.util.Iterator;

class Main {

    enum Size {
        SMALL, MEDIUM, LARGE, EXTRALARGE
    }

    public static void main(String[] args) {

        // Creating an EnumSet using allOf()
        EnumSet<Size> sizes = EnumSet.allOf(Size.class);

        Iterator<Size> iterate = sizes.iterator();

        System.out.print("EnumSet: ");
        while(iterate.hasNext()) {
            System.out.print(iterate.next());
            System.out.print(", ");
        }
    }
}
```

**Output**

```
EnumSet: SMALL, MEDIUM, LARGE, EXTRALARGE,
```

**Note**:

- `hasNext()` returns `true` if there is a next element in the enum set
- `next()` returns the next element in the enum set

------

## Remove EnumSet Elements

- `remove()` - removes the specified element from the enum set
- `removeAll()` - removes all the elements from the enum set

For example,

```
import java.util.EnumSet;

class Main {

    enum Size {
        SMALL, MEDIUM, LARGE, EXTRALARGE
    }

    public static void main(String[] args) {

        // Creating EnumSet using allOf()
        EnumSet<Size> sizes = EnumSet.allOf(Size.class);
        System.out.println("EnumSet: " + sizes);

        // Using remove()
        boolean value1 = sizes.remove(Size.MEDIUM);
        System.out.println("Is MEDIUM removed? " + value1);

        // Using removeAll()
        boolean value2 = sizes.removeAll(sizes);
        System.out.println("Are all elements removed? " + value2);
    }
}
```

**Output**

```
EnumSet: [SMALL, MEDIUM, LARGE, EXTRALARGE]
Is MEDIUM removed? true
Are all elements removed? true
```

------

## Other Methods

| Method       | Description                                                  |
| :----------- | :----------------------------------------------------------- |
| `copyOf()`   | Creates a copy of the `EnumSet`                              |
| `contains()` | Searches the `EnumSet` for the specified element and returns a boolean result |
| `isEmpty()`  | Checks if the `EnumSet` is empty                             |
| `size()`     | Returns the size of the `EnumSet`                            |
| `clear()`    | Removes all the elements from the `EnumSet`                  |

------

## Clonable and Serializable Interfaces

The `EnumSet` class also implements `Cloneable` and `Serializable` interfaces.

**Cloneable Interface**

It allows the `EnumSet` class to make a copy of instances of the class.

**Serializable Interface**

Whenever Java objects need to be transmitted over a network, objects need to be converted into bits or bytes. This is because Java objects cannot be transmitted over the network.

The `Serializable` interface allows classes to be serialized. This means objects of the classes implementing `Serializable` can be converted into bits or bytes.

------

## Why EnumSet?

The `EnumSet` provides an efficient way to store enum values than other set implementations (like `HashSet`, `TreeSet`).

An enum set only stores enum values of a specific enum. Hence, the JVM already knows all the possible values of the set.

This is the reason why enum sets are internally implemented as a sequence of bits. Bits specifies whether elements are present in the enum set or not.

The bit of a corresponding element is turned on if that element is present in the set.





EnumSet是与枚举类型一起使用的专用 Set 集合，EnumSet 中所有元素都必须是枚举类型。与其他Set接口的实现类HashSet/TreeSet(内部都是用对应的HashMap/TreeMap实现的)不同的是，EnumSet在内部实现是位向量(稍后分析)，它是一种极为高效的位运算操作，由于直接存储和操作都是bit，因此EnumSet空间和时间性能都十分可观，足以媲美传统上基于 int 的“位标志”的运算，重要的是我们可像操作set集合一般来操作位运算，这样使用代码更简单易懂同时又具备类型安全的优势。注意EnumSet不允许使用 null 元素。试图插入 null 元素将抛出 NullPointerException，但试图测试判断是否存在null 元素或移除 null 元素则不会抛出异常，与大多数collection 实现一样，EnumSet不是线程安全的，因此在多线程环境下应该注意数据同步问题，ok~，下面先来简单看看EnumSet的使用方式。



## EnumSet用法

创建EnumSet并不能使用new关键字，因为它是个抽象类，而应该使用其提供的静态工厂方法，EnumSet的静态工厂方法比较多，如下：

```java
创建一个具有指定元素类型的空EnumSet。
EnumSet<E>  noneOf(Class<E> elementType)       
//创建一个指定元素类型并包含所有枚举值的EnumSet
<E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType)
// 创建一个包括枚举值中指定范围元素的EnumSet
<E extends Enum<E>> EnumSet<E> range(E from, E to)
// 初始集合包括指定集合的补集
<E extends Enum<E>> EnumSet<E> complementOf(EnumSet<E> s)
// 创建一个包括参数中所有元素的EnumSet
<E extends Enum<E>> EnumSet<E> of(E e)
<E extends Enum<E>> EnumSet<E> of(E e1, E e2)
<E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3)
<E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4)
<E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4, E e5)
<E extends Enum<E>> EnumSet<E> of(E first, E... rest)
//创建一个包含参数容器中的所有元素的EnumSet
<E extends Enum<E>> EnumSet<E> copyOf(EnumSet<E> s)
<E extends Enum<E>> EnumSet<E> copyOf(Collection<E> c)123456789101112131415161718
```

代码演示如下：

```java
package zejian;

import java.util.ArrayList;
import java.util.EnumSet;
import java.util.List;

/**
 * Created by wuzejian on 2017/5/12.
 *
 */
enum Color {
    GREEN , RED , BLUE , BLACK , YELLOW
}


public class EnumSetDemo {

    public static void main(String[] args){

        //空集合
        EnumSet<Color> enumSet= EnumSet.noneOf(Color.class);
        System.out.println("添加前："+enumSet.toString());
        enumSet.add(Color.GREEN);
        enumSet.add(Color.RED);
        enumSet.add(Color.BLACK);
        enumSet.add(Color.BLUE);
        enumSet.add(Color.YELLOW);
        System.out.println("添加后："+enumSet.toString());

        System.out.println("-----------------------------------");

        //使用allOf创建包含所有枚举类型的enumSet，其内部根据Class对象初始化了所有枚举实例
        EnumSet<Color> enumSet1= EnumSet.allOf(Color.class);
        System.out.println("allOf直接填充："+enumSet1.toString());

        System.out.println("-----------------------------------");

        //初始集合包括枚举值中指定范围的元素
        EnumSet<Color> enumSet2= EnumSet.range(Color.BLACK,Color.YELLOW);
        System.out.println("指定初始化范围："+enumSet2.toString());

        System.out.println("-----------------------------------");

        //指定补集，也就是从全部枚举类型中去除参数集合中的元素，如下去掉上述enumSet2的元素
        EnumSet<Color> enumSet3= EnumSet.complementOf(enumSet2);
        System.out.println("指定补集："+enumSet3.toString());

        System.out.println("-----------------------------------");

        //初始化时直接指定元素
        EnumSet<Color> enumSet4= EnumSet.of(Color.BLACK);
        System.out.println("指定Color.BLACK元素："+enumSet4.toString());
        EnumSet<Color> enumSet5= EnumSet.of(Color.BLACK,Color.GREEN);
        System.out.println("指定Color.BLACK和Color.GREEN元素："+enumSet5.toString());

        System.out.println("-----------------------------------");

        //复制enumSet5容器的数据作为初始化数据
        EnumSet<Color> enumSet6= EnumSet.copyOf(enumSet5);
        System.out.println("enumSet6："+enumSet6.toString());

        System.out.println("-----------------------------------");

        List<Color> list = new ArrayList<Color>();
        list.add(Color.BLACK);
        list.add(Color.BLACK);//重复元素
        list.add(Color.RED);
        list.add(Color.BLUE);
        System.out.println("list:"+list.toString());

        //使用copyOf(Collection<E> c)
        EnumSet enumSet7=EnumSet.copyOf(list);
        System.out.println("enumSet7:"+enumSet7.toString());

        /**
         输出结果：
         添加前：[]
         添加后：[GREEN, RED, BLUE, BLACK, YELLOW]
         -----------------------------------
         allOf直接填充：[GREEN, RED, BLUE, BLACK, YELLOW]
         -----------------------------------
         指定初始化范围：[BLACK, YELLOW]
         -----------------------------------
         指定补集：[GREEN, RED, BLUE]
         -----------------------------------
         指定Color.BLACK元素：[BLACK]
         指定Color.BLACK和Color.GREEN元素：[GREEN, BLACK]
         -----------------------------------
         enumSet6：[GREEN, BLACK]
         -----------------------------------
         list:[BLACK, BLACK, RED, BLUE]
         enumSet7:[RED, BLUE, BLACK]
         */
    }

}
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273747576777879808182838485868788899091929394959697
```

`noneOf(Class<E> elementType)`静态方法，主要用于创建一个空的EnumSet集合，传递参数elementType代表的是枚举类型的类型信息，即Class对象。`EnumSet<E> allOf(Class<E> elementType)`静态方法则是创建一个填充了elementType类型所代表的所有枚举实例，奇怪的是EnumSet提供了多个重载形式的of方法，最后一个接受的的是可变参数，其他重载方法则是固定参数个数，EnumSet之所以这样设计是因为可变参数的运行效率低一些，所有在参数数据不多的情况下，强烈***不建议\***使用传递参数为可变参数的of方法，即`EnumSet<E> of(E first, E... rest)`，其他方法就不分析了，看代码演示即可。至于EnumSet的操作方法，则与set集合是一样的，可以看API即可这也不过多说明。什么时候使用EnumSet比较恰当的，事实上当需要进行位域运算，就可以使用EnumSet提到位域，如下：

```
public class EnumSetDemo {
    //定义位域变量
    public static final int TYPE_ONE = 1 << 0 ; //1
    public static final int TYPE_TWO = 1 << 1 ; //2
    public static final int TYPE_THREE = 1 << 2 ; //4
    public static final int TYPE_FOUR = 1 << 3 ; //8
    public static void main(String[] args){
        //位域运算
        int type= TYPE_ONE | TYPE_TWO | TYPE_THREE |TYPE_FOUR;
    }
}1234567891011
```

诸如上述情况，我们都可以将上述的类型定义成枚举然后采用EnumSet来装载，进行各种操作，这样不仅不用手动编写太多冗余代码，而且使用EnumSet集合进行操作也将使代码更加简洁明了。

```
enum Type{
    TYPE_ONE,TYPE_TWO,TYPE_THREE,TYPE_FOUR 
}

public class EnumSetDemo {
    public static void main(String[] args){
    EnumSet set =EnumSet.of(Type.TYPE_ONE,Type.TYPE_FOUR);
    }
}123456789
```

其实博主认为EnumSet最有价值的是其内部实现原理，采用的是位向量，它体现出来的是一种高效的数据处理方式，这点很值得我们去学习它。

## EnumSet实现原理剖析

关于EnumSet实现原理可能会有点烧脑，内部执行几乎都是位运算，博主将尽力使用图片来分析，协助大家理解。

### 理解位向量

在分析EnumSet前有必要先了解以下位向量，顾名思义位向量就是用一个bit位(0或1)标记一个元素的状态，用一组bit位表示一个集合的状态，而每个位对应一个元素，每个bit位的状态只可能有两种，即0或1。位向量能表示的元素个数与向量的bit位长度有关，如一个int类型能表示32个元素，而一个long类型则可以表示64个元素，对于EnumSet而言采用的就long类型或者long类型数组。比如现在有一个文件中的数据，该文件存储了N=1000000个无序的整数，需要把这些整数读取到内存并排序再重新写回文件中，该如何解决？最简单的方式是用int类型来存储每个数，并把其存入到数组(int a[m])中，再进行排序，但是这种方式将会导致存储空间异常大，对数据操作起来效率也能成问题，那有没更高效的方式呢？的确是有的，那就是运用位向量，我们知道一个int型的数有4个字节，也就是32位，那么我们可以用N/32个int型数组来表示这N个数：

```
a[0]表示第1~32个数（0~31）
a[1]表示第33~64个数（32~63）
a[2]表示第65~96个数（64~95）
...... 以此类推1234
```

这样，每当输入一个数字m，我们应该先找到该数字在数组的第？个元素，也就是a[?]，然后再确定在这个元素的第几个bit位，找到后设置为1，代表存在该数字。举个例子来说，比如输入40，那么40/32为1余8，则应该将a[1]元素值的第9个bit位置为1(1的二进制左移8位后就是第9个位置)，表示该数字存在，40数字的表示原理图过程如下：

![img](https://img-blog.csdn.net/20170513085533480?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

大概明白了位向量表示方式后，上述过程的计算方式，通过以下方式可以计算该数存储在数组的第?个元素和元素中第?个bit位置,为了演示方便，我们这里假设整第?个元素中的?为P，余值设置S

```
//m 除以 2^n 则商(P)表示为 m >> n 
//等同于 m / 2^5 取整数 即：40 / 32 = 1 ，那么P=1就是数组中第2个元素，即a[1]

//位操作过程如下，40的二进制
00000000 00000000 00000000 00101000

//右移5位即 n=5 ， m >> 5 ，即结果转为10进制就是P=1
00000000 00000000 00000000 00000001123456789
```

在这里我们使用的int类型，即32位，所有2^5=32，因此n=5，由此计算出 P的值代表的是数组的第 P 个元素，接着利用下述方式计算出余数（S），以此设置该元素值的第（S+1）个bit位为1

```java
//m 除以2^n 的余数(S)表示为 m & (2^n-1) 
//等同于： m % 2^5 取余数 即：40 % 32 = 8

//m=40的二进制
00000000 00000000 00000000 00101000

//2^n-1（31）的二进制
00000000 00000000 00000000 00011111

// m & (2^n-1) 即40与31进行与操作得出余数 即 S=8
00000000 00000000 00000000 00001000 

//下面是将a[1]元素值的第(8+1)个bit设置为1，为什么是(8+1)不是8？因为1左移8位就在第9个bit位了，过程如下：

//1的二进制如下：
00000000 00000000 00000000 00000001

//1 << 8 利用余数8对1进行左移动
00000000 00000000 00000001 0000000 

//然后再与a[1]执行或操作后就可以将对应的bit位设置为1
//a[P] |= 1 << S 见下述java实现的代码12345678910111213141516171819202122
```

通过上述二进制位运算过程(关于位运算可以看博主的另一篇博文~[java位运算](http://blog.csdn.net/javazejian/article/details/51181320))就可以计算出整数部分P和余数部分S，并成功设置bit位为1，现在利用java来实现这个运算过程如下：

```java
//定义变量
private int[] a; //数组存储元素的数组
private int BIT_LENGTH = 32;//默认使用int类型
private int P; //整数部分
private int S; //余数
private int MASK =  0x1F;// 2^5 - 1
private int SHIFT = 5; // 2^n SHIFT=n=5 表示2^5=32 即bit位长度321234567
```

计算代码

```java
/**
 * 置位操作,添加操作
 * @param i
 */
public void set(int i){
     P = i >> SHIFT; //结果等同  P = i / BIT_LENGTH; 取整数 ①
     S = i & MASK;   //结果等同  S = i % BIT_LENGTH; 取余数 ②

     a[P] |= 1 << S;  //赋值设置该元素bit位为1               ③
     //将int型变量j的第k个比特位设置为1， 即j=j|(1<<k),上述3句合并为一句
     //a[i >> SHIFT ] |= (1 << (i & MASK));               ④
 }123456789101112
```

计算出P和S后，就可以进行赋值了，其中 a[P]代表数组中第P个元素，`a[P] |= 1 << S` 整句意思是把a[P]元素的第S+1位设置为1，注意从低位到高位设置，即从右到左，①②③合并为④，代码将更佳简洁。当然有添加操作，那么就会有删除操作，删除操作过程与添加类似，只不过删除是把相对应的bit位设置0，代表不存在该数值。

```java
/**
* 置0操作，相当于清除元素
* @param i
*/
public void clear(int i){
   P =  i >> SHIFT; //计算位于数组中第？个元素 P = i / BIT_LENGTH;
   S =  i & MASK;   //计算余数  S = i % BIT_LENGTH;
   //把a[P]元素的第S+1个(从低位到高位)bit位设置为0
   a[P] &= ~(1 << S);

   //更优写法
   //将int型变量j的第k个比特位设置为0，即j= j&~(1<<k)
   //a[i>>SHIFT] &= ~(1<<(i &MASK));
}1234567891011121314
```

与添加唯一不同的是，计算出余数S，利用1左移S位，再取反(~)操作，最后进行与(&)操作，即将a[P]元素的第S+1个(从低位到高位)bit位设置为0，表示删除该数字，这个计算过程大家可以自行推算一下。这就是位向量表示法的添加和清除方法，然后我们可以利用下述的get方法判断某个bit是否存在某个数字：

```
/**
 * 读取操作，返回1代表该bit位有值，返回0代表该bit位没值
 * @param i
 * @return
 */
public int get(int i){
    //a[i>>SHIFT] & (1<<(i&MASK));
    P = i >> SHIFT;
    S = i &  MASK;
    return Integer.bitCount(a[P] & (1 << S));
}1234567891011
```

其中Integer.bitCount()是返回指定 int 值的二进制补码(计算机数字的二进制表示法都是使用补码表示的)表示形式的 1 位的数量。位向量运算整体代码实现如下：

```java
package com.zejian;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;

/**
 * Created by zejian on 2017/5/13.
 * Blog : http://blog.csdn.net/javazejian [原文地址,请尊重原创]
 * 位向量存储数据
 */
public class BitVetory {
    private int count;
    private int[] a; //数组
    private int BIT_LENGTH = 32;//默认使用int类型
    private int P; //整数部分
    private int S; //余数
    private int MASK =  0x1F;// 2^5 - 1
    private int SHIFT = 5; // 2^n SHIFT=n=5 表示2^5=32 即bit位长度32

    /**
     * 初始化位向量
     * @param count
     */
    public BitVetory(int count) {
        this.count = count;
        a = new int[(count-1)/BIT_LENGTH + 1];
        init();
    }

    /**
     * 将数组中元素bit位设置为0
     */
    public void init(){
        for (int i = 0; i < count; i++) {
            clear(i);
        }
    }

    /**
     * 获取排序后的数组
     * @return
     */
    public List<Integer> getSortedArray(){
        List<Integer> sortedArray = new ArrayList<Integer>();

        for (int i = 0; i < count; i++) {
            if (get(i) == 1) {//判断i是否存在
                sortedArray.add(i);
            }
        }
        return sortedArray;
    }
    /**
     * 置位操作,设置元素
     * @param i
     */
    public void set(int i){
        P = i >> SHIFT; //P = i / BIT_LENGTH; 取整数
        S = i & MASK; //S = i % BIT_LENGTH; 取余数
        a[P] |= 1 << S;

        //将int型变量j的第k个比特位设置为1， 即j=j|(1<<k),上述3句合并为一句
        //a[i >> SHIFT ] |= (1 << (i & MASK));
    }

    /**
     * 置0操作，相当于清除元素
     * @param i
     */
    public void clear(int i){
        P =  i >> SHIFT; //计算位于数组中第？个元素 P = i / BIT_LENGTH;
        S =  i & MASK;   //计算余数  S = i % BIT_LENGTH;
        a[P] &= ~(1 << S);

        //更优写法
        //将int型变量j的第k个比特位设置为0，即j= j&~(1<<k)
        //a[i>>SHIFT] &= ~(1<<(i &MASK));
    }

    /**
     * 读取操作，返回1代表该bit位有值，返回0代表该bit位没值
     * @param i
     * @return
     */
    public int get(int i){
        //a[i>>SHIFT] & (1<<(i&MASK));
        P = i >> SHIFT;
        S = i &  MASK;
        return Integer.bitCount(a[P] & (1 << S));
    }

    //测试
    public static void main(String[] args) {
        int count = 25;
        List<Integer> randoms = getRandomsList(count);
        System.out.println("排序前：");

        BitVetory bitVetory = new BitVetory(count);
        for (Integer e : randoms) {
            System.out.print(e+",");
            bitVetory.set(e);
        }

        List<Integer> sortedArray = bitVetory.getSortedArray();
        System.out.println();
        System.out.println("排序后：");
        for (Integer e : sortedArray) {
            System.out.print(e+",");
        }

        /**
         输出结果:
         排序前：
         6,3,20,10,18,15,19,16,13,4,21,22,24,2,14,5,12,7,23,8,1,17,9,11,
         排序后：
         1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,
         */
    }

    private static List<Integer> getRandomsList(int count) {
        Random random = new Random();

        List<Integer> randomsList = new ArrayList<Integer>();
        while(randomsList.size() < (count - 1)){
            int element = random.nextInt(count - 1) + 1;//element ∈  [1,count)
            if (!randomsList.contains(element)) {
                randomsList.add(element);
            }
        }
        return randomsList;
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134
```

### EnumSet原理

有前面位向量的分析，对于了解EnumSet的实现原理就相对简单些了，EnumSet内部使用的位向量实现的，前面我们说过EnumSet是一个抽象类，事实上它存在两个子类，RegularEnumSet和JumboEnumSet。RegularEnumSet使用一个long类型的变量作为位向量，long类型的位长度是64，因此可以存储64个枚举实例的标志位，一般情况下是够用的了，而JumboEnumSet使用一个long类型的数组，当枚举个数超过64时，就会采用long数组的方式存储。先看看EnumSet内部的数据结构：

```java
public abstract class EnumSet<E extends Enum<E>> extends AbstractSet<E>
    implements Cloneable, java.io.Serializable
{
    //表示枚举类型
    final Class<E> elementType;
    //存储该类型信息所表示的所有可能的枚举实例
    final Enum<?>[] universe;
    //..........
}123456789
```

EnumSet中有两个变量，一个elementType用于表示枚举的类型信息，universe是数组类型，存储该类型信息所表示的所有可能的枚举实例，EnumSet是抽象类，因此具体的实现是由子类完成的，下面看看`noneOf(Class<E> elementType)`静态构建方法

```java
  public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        //根据EnumMap中的一样，获取所有可能的枚举实例
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            //枚举个数小于64，创建RegularEnumSet
            return new RegularEnumSet<>(elementType, universe);
        else
            //否则创建JumboEnumSet
            return new JumboEnumSet<>(elementType, universe);
    }12345678910111213
```

从源码可以看出如果枚举值个数小于等于64，则静态工厂方法中创建的就是RegularEnumSet，否则大于64的话就创建JumboEnumSet。无论是RegularEnumSet还是JumboEnumSet，其构造函数内部都间接调用了EnumSet的构造函数，因此最终的elementType和universe都传递给了父类EnumSet的内部变量。如下：

```java
//RegularEnumSet构造
RegularEnumSet(Class<E>elementType, Enum<?>[] universe) {
      super(elementType, universe);
  }

//JumboEnumSet构造
JumboEnumSet(Class<E>elementType, Enum<?>[] universe) {
      super(elementType, universe);
      elements = new long[(universe.length + 63) >>> 6];
  }12345678910
```

在RegularEnumSet类和JumboEnumSet类中都存在一个elements变量，用于记录位向量的操作，

```
//RegularEnumSet
class RegularEnumSet<E extends Enum<E>> extends EnumSet<E> {
    private static final long serialVersionUID = 3411599620347842686L;
    //通过long类型的elements记录位向量的操作
    private long elements = 0L;
    //.......
}

//对于JumboEnumSet则是：
class JumboEnumSet<E extends Enum<E>> extends EnumSet<E> {
    private static final long serialVersionUID = 334349849919042784L;
    //通过long数组类型的elements记录位向量
    private long elements[];
     //表示集合大小
    private int size = 0;

    //.............
    }123456789101112131415161718
```

在RegularEnumSet中elements是一个long类型的变量，共有64个bit位，因此可以记录64个枚举常量，当枚举常量的数量超过64个时，将使用JumboEnumSet，elements在该类中是一个long型的数组，每个数组元素都可以存储64个枚举常量，这个过程其实与前面位向量的分析是同样的道理，只不过前面使用的是32位的int类型，这里使用的是64位的long类型罢了。接着我们看看EnumSet是如何添加数据的，RegularEnumSet中的add实现如下

```
public boolean add(E e) {
    //检测是否为枚举类型
    typeCheck(e);
    //记录旧elements
    long oldElements = elements;
    //执行位向量操作，是不是很熟悉？
    //数组版：a[i >> SHIFT ] |= (1 << (i & MASK))
    elements |= (1L << ((Enum)e).ordinal());
    return elements != oldElements;
}12345678910
```

关于`elements |= (1L << ((Enum)e).ordinal());`这句跟我们前面分析位向量操作是相同的原理，只不过前面分析的是数组类型实现，这里用的long类型单一变量实现，`((Enum)e).ordinal()`通过该语句获取要添加的枚举实例的序号，然后通过1左移再与 long类型的elements进行或操作，就可以把对应位置上的bit设置为1了，也就代表该枚举实例存在。图示演示过程如下，注意universe数组在EnumSet创建时就初始化并填充了所有可能的枚举实例，而elements值的第n个bit位1时代表枚举存在，而获取的则是从universe数组中的第n个元素值。

![img](https://img-blog.csdn.net/20170513151544085?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这就是枚举实例的添加过程和获取原理。而对于JumboEnumSet的add实现则是如下：

```
public boolean add(E e) {
    typeCheck(e);
    //计算ordinal值
    int eOrdinal = e.ordinal();
    int eWordNum = eOrdinal >>> 6;

    long oldElements = elements[eWordNum];
    //与前面分析的位向量相同：a[i >> SHIFT ] |= (1 << (i & MASK))
    elements[eWordNum] |= (1L << eOrdinal);
    boolean result = (elements[eWordNum] != oldElements);
    if (result)
        size++;
    return result;
}1234567891011121314
```

关于JumboEnumSet的add实现与RegularEnumSet区别是一个是long数组类型，一个long变量，运算原理相同，数组的位向量运算与前面分析的是相同的，这里不再分析。接着看看如何删除元素

```
//RegularEnumSet类实现
public boolean remove(Object e) {
    if (e == null)
        return false;
    Class eClass = e.getClass();
    if (eClass != elementType && eClass.getSuperclass() != elementType)
        return false;

    long oldElements = elements;
    //将int型变量j的第k个比特位设置为0，即j= j&~(1<<k)
    //数组类型：a[i>>SHIFT] &= ~(1<<(i &MASK));

    elements &= ~(1L << ((Enum)e).ordinal());//long遍历类型操作
    return elements != oldElements;
}


//JumboEnumSet类的remove实现
public boolean remove(Object e) {
        if (e == null)
            return false;
        Class<?> eClass = e.getClass();
        if (eClass != elementType && eClass.getSuperclass() != elementType)
            return false;
        int eOrdinal = ((Enum<?>)e).ordinal();
        int eWordNum = eOrdinal >>> 6;

        long oldElements = elements[eWordNum];
        //与a[i>>SHIFT] &= ~(1<<(i &MASK));相同
        elements[eWordNum] &= ~(1L << eOrdinal);
        boolean result = (elements[eWordNum] != oldElements);
        if (result)
            size--;
        return result;
    }1234567891011121314151617181920212223242526272829303132333435
```

删除remove的实现，跟位向量的清空操作是同样的实现原理，如下：
![img](https://img-blog.csdn.net/20170513171112915?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

至于JumboEnumSet的实现原理也是类似的，这里不再重复。下面为了简洁起见，我们以RegularEnumSet类的实现作为源码分析，毕竟JumboEnumSet的内部实现原理可以说跟前面分析过的位向量几乎一样。o~，看看如何判断是否包含某个元素

```
public boolean contains(Object e) {
    if (e == null)
        return false;
    Class eClass = e.getClass();
    if (eClass != elementType && eClass.getSuperclass() != elementType)
        return false;
    //先左移再按&操作
    return (elements & (1L << ((Enum)e).ordinal())) != 0;
}

public boolean containsAll(Collection<?> c) {
    if (!(c instanceof RegularEnumSet))
        return super.containsAll(c);

    RegularEnumSet<?> es = (RegularEnumSet<?>)c;
    if (es.elementType != elementType)
        return es.isEmpty();
    //~elements取反相当于elements补集，再与es.elements进行&操作，如果为0，
    //就说明elements补集与es.elements没有交集，也就是es.elements是elements的子集
    return (es.elements & ~elements) == 0;
}
12345678910111213141516171819202122
```

对于contains(Object e) 方法，先左移再按位与操作，不为0，则表示包含该元素，跟位向量的get操作实现原理类似，这个比较简单。对于`containsAll(Collection<?> c)`则可能比较难懂，这里分析一下，elements变量(long类型)标记EnumSet集合中已存在元素的bit位，如果bit位为1则说明存在枚举实例，为0则不存在，现在执行`~elements` 操作后 则说明`~elements`是elements的补集，那么只要传递进来的es.elements与补集`~elements` 执行&操作为0，那么就可以证明es.elements与补集`~elements` 没有交集的可能，也就是说es.elements只能是elements的子集，这样也就可以判断出当前EnumSet集合中包含传递进来的集合c了，借着下图协助理解：

![img](https://img-blog.csdn.net/20170513163057795?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

图中，elements代表A，`es.elements`代表S，`~elements`就是求A的补集，`(es.elements & ~elements) == 0`就是在验证A’∩B是不是空集，即S是否为A的子集。接着看retainAll方法，求两个集合交集

```
public boolean retainAll(Collection<?> c) {
        if (!(c instanceof RegularEnumSet))
            return super.retainAll(c);

        RegularEnumSet<?> es = (RegularEnumSet<?>)c;
        if (es.elementType != elementType) {
            boolean changed = (elements != 0);
            elements = 0;
            return changed;
        }

        long oldElements = elements;
        //执行与操作，求交集，比较简单
        elements &= es.elements;
        return elements != oldElements;
    }12345678910111213141516
```

最后来看看迭代器是如何取值的

```
 public Iterator<E> iterator() {
        return new EnumSetIterator<>();
    }

    private class EnumSetIterator<E extends Enum<E>> implements Iterator<E> {
        //记录elements
        long unseen;

        //记录最后一个返回值
        long lastReturned = 0;

        EnumSetIterator() {
            unseen = elements;
        }

        public boolean hasNext() {
            return unseen != 0;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            if (unseen == 0)
                throw new NoSuchElementException();
            //取值过程，先与本身负执行&操作得出的就是二进制低位开始的第一个1的数值大小
            lastReturned = unseen & -unseen;
            //取值后减去已取得lastReturned
            unseen -= lastReturned;
            //返回在指定 long 值的二进制补码表示形式中最低位（最右边）的 1 位之后的零位的数量
            return (E) universe[Long.numberOfTrailingZeros(lastReturned)];
        }

        public void remove() {
            if (lastReturned == 0)
                throw new IllegalStateException();
            elements &= ~lastReturned;
            lastReturned = 0;
        }
    }1234567891011121314151617181920212223242526272829303132333435363738
```

比较晦涩的应该是

```
//取值过程，先与本身负执行&操作得出的就是二进制低位开始的第一个1的数值大小
lastReturned = unseen & -unseen; 
//取值后减去已取得lastReturned
unseen -= lastReturned;
return (E) universe[Long.numberOfTrailingZeros(lastReturned)];12345
```

我们通过原理图来协助理解，现在假设集合中已保存所有可能的枚举实例变量，我们需要把它们遍历展示出来，下面的第一个枚举元素的获取过程，显然通过`unseen & -unseen;`操作，我们可以获取到二进制低位开始的第一个1的数值，该计算的结果是要么全部都是0，要么就只有一个1，然后赋值给lastReturned，通过`Long.numberOfTrailingZeros(lastReturned)`获取到该bit为1在64位的long类型中的位置，即从低位算起的第几个bit，如图，该bit的位置恰好是低位的第1个bit位置，也就指明了universe数组的第一个元素就是要获取的枚举变量。执行`unseen -= lastReturned;`后继续进行第2个元素的遍历，依次类推遍历出所有值，这就是EnumSet的取值过程，真正存储枚举变量的是universe数组，而通过long类型变量的bit位的0或1表示存储该枚举变量在universe数组的那个位置，这样做的好处是任何操作都是执行long类型变量的bit位操作，这样执行效率将特别高，毕竟是二进制直接执行，只有最终获取值时才会操作到数组universe。

![img](https://img-blog.csdn.net/20170513174622751?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

ok~，到这关于EnumSet的实现原理主要部分我们就分析完了，其内部使用位向量，存储结构很简洁，节省空间，大部分操作都是按位运算，直接操作二进制数据，因此效率极高。当然通过前面的分析，我们也掌握位向量的运算原理。好~，关于java枚举，我们暂时聊到这。