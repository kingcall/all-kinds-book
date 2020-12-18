[TOC]

## 枚举初识

![枚举.png](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/15/17:04:08-1240.png)

`enum` 的全称为 enumeration，在Java中，被 `enum` 关键字修饰的类型就是枚举类型。例如`enum{RED,BLUE,WHITE,BLACK}`

枚举类型是Java 5中新增特性的一部分，它是一种特殊的数据类型，之所以特殊是因为它既是一种类(class)类型却又比类类型**多了些特殊的约束，但是这些约束的存在也造就了枚举类型的简洁性、安全性以及便捷性**。

**枚举的好处**：可以将常量组织起来，统一进行管理。枚举的典型应用场景 **错误码、状态机等**。



我们看一下下面的程序，这是在没有枚举类型时定义常量常见的方式，当然直到今天依然有人这么定义和使用，当然这也无可厚非，今天学习了之后就不要这么干了

```java
public class DateNoneEnum {
    public static final int MONDAY =1;
    public static final int TUESDAY=2;
    public static final int WEDNESDAY=3;
    public static final int THURSDAY=4;
    public static final int FRIDAY=5;
    public static final int SATURDAY=6;
    public static final int SUNDAY=7;
}
```

上述的常量定义常量的方式称为Int枚举模式，当然还有其他的模式，例如字符串枚举模式，这样的定义方式并没有什么错，但它存在许多不足，如在类型安全和使用方便性上并没有多少好处，如果存在定义int值相同的变量，混淆的几率还是很大的，编译器也不会提出任何警告，因此这种方式在枚举出现后并不提倡，现在我们利用枚举类型来重新定义上述的常量，同时也感受一把枚举定义的方式，如下定义周一到周日的常量

```java
public enum DateEnum {
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY
}
```

相当简洁，在定义枚举类型时我们使用的关键字是enum，与class关键字类似，只不过前者是定义枚举类型，后者是定义类类型。

枚举类型DateEnum中分别定义了从周一到周日的值，这里要注意的是**枚举值一般是大写的**，多个值之间以逗号分隔。

同时我们应该知道的是枚举类型可以像类(class)类型一样，定义为一个单独的文件，当然也可以定义在其他类内部，更重要的是枚举常量在类型安全性和便捷性都很有保证，如果出现类型问题编译器也会提示我们改进，**但务必记住枚举表示的类型其取值是必须有限的，也就是说每个值都是可以枚举出来的，比如上述描述的一周共有七天**(有很多东西也是无法枚举的例如整数值这样，无限的东西)

那么该如何使用呢？如下：

```java
public class EnumDemo {
    @Test
    public void useEnum() {
        DateEnum dateEnum = DateEnum.MONDAY;
    }
}
```

就像上述代码那样，直接引用枚举的值即可，这便是枚举类型的最简单模型。

### 枚举实现原理

#### 继承自Enum 抽象类的枚举

我们大概了解了枚举类型的定义与简单使用后，现在有必要来了解一下枚举类型的基本实现原理。

实际上在使用关键字enum创建枚举类型并编译后，编译器会为我们生成一个相关的类，这个类继承了Java API中的java.lang.Enum类，也就是说通过关键字enum创建枚举类型在编译后事实上也是一个类类型而且该类继承自java.lang.Enum类。

下面我们编译前面定义的DateEnum.java并查看生成的class文件来验证这个结论：

利用javac编译前面定义的DateEnum.java文件，然后我们查看生成的DateEnum.class，这也就验证前面所说的使用关键字enum定义枚举类型并编译后，编译器会自动帮助我们生成一个与枚举相关的类。

![image-20201214213850517](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/14/21:38:51-image-20201214213850517.png)

从反编译的代码可以看出编译器确实帮助我们生成了一个DateEnum类，**注意该类是final类型的，将无法被继承**，而且该类继承自`java.lang.Enum类`，该类是一个抽象类(稍后我们会分析该类中的主要方法)

除此之外，此外我们看到我们在DateEnum类中定义的七个变量，都是DateEnum类型的变量，并且都是public static final 修饰的,这个说明了我们我们在上面为什么可以这样`DateEnum.MONDAY`使用

这也充分说明了我们前面使用关键字enum定义的DateEnum类型中的每种日期枚举常量也是实实在在的DateEnum类的对象

注意编译器还为我们生成了两个静态方法，分别是values()和 valueOf()，稍后会分析它们的用法，到此我们也就明白了，使用关键字enum定义的枚举类型，在编译期后，也将转换成为一个实实在在的类，而在该类中，会存在每个在枚举类型中定义好变量的对应的是该类的对象，如上述的MONDAY枚举类型对应`public static final DateEnum MONDAY;`，同时编译器会为该类创建两个方法，分别是values()和valueOf()。

ok~，到此相信我们对枚举的实现原理也比较清晰，下面我们深入了解一下java.lang.Enum类以及values()和valueOf()的用途以及重点介绍一下Enum抽象类，既然知道了本质上还是一个类，那我们就看一下它的父类，也就是java.lang.Enum的说明书和类的继承关系



#### Enum 抽象类的说明书

在看是看说明书之前，我们先看一下Enum类的继承关系，有一个大致的了解

![image-20201215084418131](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/15/08:44:19-image-20201215084418131.png)



```java
/**
 * This is the common base class of all Java language enumeration types.
 * 这是java 枚举类型的基础公共类
 * More information about enums, including descriptions of the implicitly declared methods synthesized by the compiler, can be
 * found in section 8.9 of <cite>The Java&trade; Language Specification</cite>.
 * 有关枚举的更多信息，包括编译器合成的隐式声明方法的描述，可以看
 * <p> Note that when using an enumeration type as the type of a set
 * or as the type of the keys in a map, specialized and efficient
 * {@linkplain java.util.EnumSet set} and {@linkplain
 * java.util.EnumMap map} implementations are available.
 * 如果你想使用枚举类型的set 或者map 可以使用EnumSet和EnumMap 
 * @param <E> The enum type subclass
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see     Class#getEnumConstants()
 * @see     java.util.EnumSet
 * @see     java.util.EnumMap
 * @since   1.5
 */
public abstract class Enum<E extends Enum<E>> implements Comparable<E>, Serializable {
 		... ...       
 }       
```

我们可以看到Enum实现了Comparable和Serializable接口，关于Comparable接口我们前面已经讲过了[深度剖析—Comparable和Comparator](https://blog.csdn.net/king14bhhb/article/details/110941207) 和[一文掌握Comparator的数十种用法](https://blog.csdn.net/king14bhhb/article/details/110941401),关于Serializable接口，后面我们也会单独讲解，不同于常规的继承Serializable接口，Enum抽象类对其的实现有其特殊的地方，我们后面讲解



### Enum的常见方法

Enum是所有 Java 语言枚举类型的公共基本类（注意Enum是抽象类），以下是它的常见方法：

| 返回类型                      | 方法名称                                         | 方法说明                                                     |
| ----------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| `int`                         | `compareTo(E o)`                                 | 比较此枚举与指定对象的顺序                                   |
| `boolean`                     | `equals(Object other)`                           | 当指定对象等于此枚举常量时，返回 true。                      |
| `Class<?>`                    | `getDeclaringClass()`                            | 返回与此枚举常量的枚举类型相对应的 Class 对象                |
| `String`                      | `name()`                                         | 返回此枚举常量的名称，在其枚举声明中对其进行声明             |
| `int`                         | `ordinal()`                                      | 返回枚举常量的序数（它在枚举声明中的位置，其中初始常量序数为零） |
| `String`                      | `toString()`                                     | 返回枚举常量的名称，它包含在声明中                           |
| `static<T extends Enum<T>> T` | `static valueOf(Class<T> enumType, String name)` | `返回带指定名称的指定枚举类型的枚举常量。`                   |

#### ordinal方法

`ordinal()`方法，该方法获取的是枚举变量在枚举类中声明的顺序，下标从0开始，如日期中的MONDAY在第一个位置，那么MONDAY的ordinal值就是0，如果MONDAY的声明位置发生变化，那么ordinal方法获取到的值也随之变化

注意在大多数情况下我们都不应该首先使用该方法，毕竟它总是变幻莫测的。下面我们测试一下，是否是这样的，我们发现这个测试用例可以跑过，那就说明`DateEnum.MONDAY.ordinal()`的返回值确实是0

```java
@Test
public void useEnumOrdinal() {
    int ordinal=DateEnum.MONDAY.ordinal();
    Assert.assertEquals(0,ordinal);
}
```

#### compareTo 方法

`compareTo(E o)`方法则是比较枚举的大小，注意其内部实现是根据每个枚举的ordinal值大小进行比较的。

```java
@Test
public void useEnumCompareTo() {
    int compare=DateEnum.MONDAY.compareTo(DateEnum.FRIDAY);
    System.out.println(compare);
}
// 数据结果-4 
```

我们知道DateEnum.MONDAY的ordinal是0，DateEnum.FRIDAY的ordinal是4，那相减不就是-4吗？接下来我们还是看一下源码

```java
/**
 * Compares this enum with the specified object for order.  Returns a
 * negative integer, zero, or a positive integer as this object is less
 * than, equal to, or greater than the specified object.
 * 为了获得先后顺序，将当前枚举和指定对象进行比较，当当前对象小于、等于、大于、指定对象的时候，依次返回一个负数，0,正数 
 * Enum constants are only comparable to other enum constants of the same enum type.  
 * 枚举常量只能与其他相同类型的枚举常量比较
 * The natural order implemented by this method is the order in which the constants are declared.
 * 此方法实现的自然顺序是常量的声明顺序
 */
public final int compareTo(E o) {
    Enum<?> other = (Enum<?>)o;
    Enum<E> self = this;
    if (self.getClass() != other.getClass() && // optimization
        self.getDeclaringClass() != other.getDeclaringClass())
        throw new ClassCastException();
    return self.ordinal - other.ordinal;
}
```

总结：**1. 比较的是位置(变量申明的位置) 2. 只能进行同类型的比较**



#### name 方法和 toString 方法

`name()`方法与`toString()`几乎是等同的，都是输出变量的字符串形式。

```java
@Test
public void useEnumName() {
    String name = DateEnum.MONDAY.name();
    String toStr = DateEnum.MONDAY.toString();
    System.out.println(name);
    System.out.println(toStr);
}
// 输出结果
MONDAY
MONDAY
```

这两个方法的签名和实现我都帖在下边了，接下来我们看一下，有人要说这有啥好看的，那是因为有两个方法都能达到获取变量名称这个效果，你可不得看一下

```java
/**
 * Returns the name of this enum constant, exactly as declared in its enum declaration.
 * 返回枚举常量的名称，与枚举声明中声明的完全相同
 * <b>Most programmers should use the {@link #toString} method in preference to this one, as the toString method may return a more user-friendly name.</b> 
 * 程序中应该优先使用toString而不是这个方法，因为toString方法返回的值更加友好
 * This method is designed primarily for use in specialized situations where correctness depends on getting the
 * exact name, which will not vary from release to release.
 * 而这个方法主要是为了特殊用途设计的，就是那些程序的正确性依赖精确的变量名称的场景，也就是那些不会因为版本的而变化的场景
 * @return the name of this enum constant
 */
public final String name() {
    return name;
}
/**
 * Returns the name of this enum constant, as contained in the declaration.  
 * 返回声明中包含的此枚举常量的名称
 * This method may be overridden, though it typically
 * isn't necessary or desirable.  An enum type should override this
 * method when a more "programmer-friendly" string form exists.
 * 这个方法可能被覆盖尽管这不是必须的或者值得做的，如果程序需要一个更加友好的返回值，可以覆盖这个方法
 * @return the name of this enum constant
 */
public String toString() {
    return name;
}
```

这下，你知道了吧，这两个是有区别的，存在即合理，别人也不会无缘无故的设计！保持好奇心，**永远是你优秀而又高贵的品质**，**因为name 方法的设计目的的原因，所以name 方法是final的，也就是不能重写的，所以它保证了name 方法的返回值永远是不会变的**

接下来，我们重写一下toString 方法看看，上面的输出，也就是我们的测试用例`useEnumName`

```java
public enum DateEnum {
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY;

    @Override
    public String toString() {
        return this.getClass() + "." + name();
    }
}
```

```java
@Test
public void useEnumName() {
    String name = DateEnum.MONDAY.name();
    String toStr = DateEnum.MONDAY.toString();
    System.out.println(name);
    System.out.println(toStr);
}
// 输出结果
MONDAY
class datastructure.java.DateEnum.MONDAY
```

**可以看出toString我们可以重写它使得输出更加的友好**



#### 编译器生成的Values方法与ValueOf方法

values()方法和valueOf(String name)方法是编译器生成的static方法，因此从前面的分析中，在Enum类中并没出现values()方法，但valueOf()方法还是有出现的，只不过编译器生成的valueOf()方法只需传递一个name参数，而Enum自带的静态方法valueOf()则需要传递两个方法，从前面反编译后的代码可以看出，编译器生成的valueOf方法最终还是调用了Enum类的valueOf方法



下面通过代码来演示这两个方法的作用，首先我们看一下values() 方法,因为上面我重写了DateEnum的toString 方法，所以在演示之前我将重写的toString方法注释掉了，因为values()方法返回的是一个数组，其实我们大概也能猜个七七八八，但是为了验证我们的擦想，我这里将其输出

```java
@Test
public void useEnumValues() {
    System.out.println(Arrays.toString( DateEnum.MONDAY.values()));
    System.out.println(Arrays.toString( DateEnum.values()));
}

//输出结果
[MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY]
[MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY]
```

从结果可知道，values()方法的作用就是获取枚举类中的所有变量，并作为数组返回

这里我们还必须注意到，由于values()方法是由编译器插入到枚举类(生成的结果类)中的static方法，所以如果我们将枚举实例向上转型为Enum，那么values()方法将无法被调用，因为Enum类中并没有values()方法，valueOf()方法也是同样的道理(注意是一个参数的)。

这里我们先看一下，抽闲类中的源码

```java
/**
 * Returns the enum constant of the specified enum type with the
 * specified name.  The name must match exactly an identifier used
 * to declare an enum constant in this type.  (Extraneous whitespace characters are not permitted.)
 * 根据指定的名称和类型，返回指定类型，指定名称的枚举常量，名称必须和枚举类型声明的时候完全一致（多余的空格都不允许）
 * @since 1.5
 */
public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                            String name) {
    T result = enumType.enumConstantDirectory().get(name);
    if (result != null)
        return result;
    if (name == null)
        throw new NullPointerException("Name is null");
    throw new IllegalArgumentException(
        "No enum constant " + enumType.getCanonicalName() + "." + name);
}
```

从上面的注释中我们知道了，**它的作用就是根据我们传入的名称，返回该名称对应的枚举常量**，仅此而已，接下来我们看一下它的使用方式，这里你知道为什么有了toString方法还有name 方法了吧，你总不能那toString 的返回值当做valueOf 方法的参数吧，更何况我们还重写了

toString 方法

```java
@Test
public void useEnumvalueOf() {
    Assert.assertEquals(DateEnum.MONDAY,DateEnum.MONDAY.valueOf(DateEnum.MONDAY.name()));
    System.out.println( DateEnum.MONDAY.valueOf(DateEnum.MONDAY.name()));
    System.out.println( DateEnum.valueOf(DateEnum.MONDAY.name()));
}
// 输出结果 
class datastructure.DateEnum.MONDAY
class datastructure.DateEnum.MONDAY
```

从输出结果，我们可以看到测试用例跑过了，也就是说它返回了DateEnum.MONDAY枚举常量,接下来我们尝试调用一下抽象类中的valueOf方法，也就是两个参数的那个

```java
@Test
public void useEnumvalueOf2() {
    Assert.assertEquals(DateEnum.MONDAY,DateEnum.MONDAY.valueOf(DateEnum.class,DateEnum.MONDAY.name()));
    System.out.println( DateEnum.MONDAY.valueOf(DateEnum.class,DateEnum.MONDAY.name()));
    System.out.println( DateEnum.valueOf(DateEnum.class,DateEnum.MONDAY.name()));
}
// 输出结果 
class datastructure.DateEnum.MONDAY
class datastructure.DateEnum.MONDAY
```



从输出结果上来看，效果是一样的，也就是说编译器帮我们生成的抽象类的valueOf是简化了Enum 的valueOf的使用方式



#### getDeclaringClass

首先我们先看一下这个方法的源码

```java
/**
 * Returns the Class object corresponding to this enum constant's enum type.  
 * 返回这个枚举常量的对应的Class 对象(关于什么是class 对象后面我们会讲的)
 * Two enum constants e1 and  e2 are of the same enum type if and only if
 *   e1.getDeclaringClass() == e2.getDeclaringClass().
 * 判断两个枚举常量e1 和 e2 是不是相同的  e1.getDeclaringClass() == e2.getDeclaringClass().
 * (The value returned by this method may differ from the one returned
 * by the {@link Object#getClass} method for enum constants with
 * constant-specific class bodies.)
 * 这个方法的返回值可能和枚举对象调用getClass方法的返回值不一样
 * @return the Class object corresponding to this enum constant's
 *     enum type
 */
@SuppressWarnings("unchecked")
public final Class<E> getDeclaringClass() {
  	// 获取当前对象的类对象
    Class<?> clazz = getClass();
  	// 获取当前对象的类对象的父类对象
    Class<?> zuper = clazz.getSuperclass();
    return (zuper == Enum.class) ? (Class<E>)clazz : (Class<E>)zuper;
}
```

下面我们看一下一个demo,帮助我们理解一下

```java
@Test
public void useEnumClass() {
    Class klass = DateEnum.MONDAY.getDeclaringClass();
    System.out.println(klass);
    Class blass = DateEnum.MONDAY.getClass();
    System.out.println(blass);
    System.out.println(blass);
    System.out.println(klass==blass);
}
```

输出结果

```java
class datastructure.java.DateEnum
class datastructure.java.DateEnum
class java.lang.Enum
true
```

我们可以看到getDeclaringClass 方法和.getClass方法的输出是一样的



上述我们提到当枚举实例向上转型为Enum类型后，values()方法将会失效，也就无法一次性获取所有枚举实例变量

但是由于Class对象的存在，即使不使用values()方法，还是有可能一次获取到所有枚举实例变量的，在**Class**对象中存在如下方法：

| 返回类型  | 方法名称             | 方法说明                                                     |
| --------- | -------------------- | ------------------------------------------------------------ |
| `T[]`     | `getEnumConstants()` | 返回该枚举类型的所有元素，如果Class对象不是枚举类型，则返回null。 |
| `boolean` | `isEnum()`           | 当且仅当该类声明为源代码中的枚举时返回 true                  |

因此通过getEnumConstants()方法，同样可以轻而易举地获取所有枚举实例变量下面通过代码来演示这个功能：

```java
@Test
public void useClassGetEnumConstants() {
    Enum e = DateEnum.MONDAY;
    Class<?> klass = e.getDeclaringClass();
    if(klass.isEnum()) {
        DateEnum[] constants = (DateEnum[]) klass.getEnumConstants();
        System.out.println("constants1:"+Arrays.toString(constants));
    }
    DateEnum[] constants= DateEnum.MONDAY.values();
    System.out.println("constants2:"+Arrays.toString(constants));
}
```

输出结果

```
constants1:[MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY]
constants2:[MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY]
```



正如上述代码所展示，通过Enum的class对象的getEnumConstants方法，我们仍能一次性获取所有的枚举实例常量,就和枚举对象的values 方法一样

## 总结

1. **枚举的好处**：可以将常量组织起来，统一进行管理 枚举的典型应用场景 **错误码、状态机等**。
2. 枚举的使用方法很简单，只需要使用enum 关键字申明即可
3. 枚举的底层原理是继承了Enum 抽象类，编译器最终会将枚举编译成一个普通的Java类供Java 虚拟机使用
4. 枚举中申明的变量都是有类型的，那就是该枚举类型(其实也是一个普通的java类)



