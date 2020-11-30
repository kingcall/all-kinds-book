StringBuilder与StringBuffer作用就是用来处理字符串，但String类本身也具备很多方法可以用来处理字符串，那么为什么还要引入这两个类呢？

关于String的讲解请看[Java基础(三) String深度解析]()

首先看下面的例子

```
public static void main(String[] args) {
    String str0 = "hel,lo,wor,l,d";

    long start = System.currentTimeMillis();
    for (int i = 0; i < 100000; i++){
        str0 += i;
    }
    System.out.println(System.currentTimeMillis() - start);

    StringBuilder sb = new StringBuilder("hel,lo,wor,l,d");
    long start1 = System.currentTimeMillis();
    for (int i = 0; i < 100000; i++){
        sb.append(i);
    }
    System.out.println(System.currentTimeMillis() - start1);

    StringBuffer sbf = new StringBuffer("hel,lo,wor,l,d");
    long start2 = System.currentTimeMillis();
    for (int i = 0; i < 100000; i++){
        sbf.append(i);
    }
    System.out.println(System.currentTimeMillis() - start2);
}
```

上述代码中3处循环完成了同样的功能，字符串拼接，执行的结果如下：

```
36823
3
4
```

可以看出执行时间差别太大，为了解决String不擅长的大量字符串拼接这种业务场景，引入了StringBuffer和StringBuilder.

首先我们分析一下为什么String在大量字符串拼接这种场景下这么慢？

因为String本身不可变，我们对String的任何操作都会返回一个新的对象，然后当前String变量指向新的对象，而原来的String对象就会被GC回收，那么在循环中就会大量快速的创建新的对象，大量原来的对象会不断的被GC回收，消耗的时间是非常恐怖的，而且内存占用非常大。

下面我们对比了String、StringBuffer与StringBuilder的区别

| String                   | StringBuffer                               | StringBuilder          |
| ------------------------ | ------------------------------------------ | ---------------------- |
| final修饰，不可继承      | final修饰，不可继承                        | final修饰，不可继承    |
| 字符串常量，创建后不可变 | 字符串变量，可动态修改                     | 字符串变量，可动态修改 |
| 不存在线程安全问题       | 线程安全，所有public方法由synchronized修改 | 线程不安全             |
| 大量字符串拼接效率最低   | 大量字符串拼接效率非常高                   | 大量字符串拼接效率最高 |

StringBuffer与StringBuilder实现非常类似，下面以StringBuilder简单说明一下append()方法基本原理

#### 1. 首先创建一个StringBuilder

```
StringBuilder sb1 = new StringBuilder();
StringBuilder sb2 = new StringBuilder(100);
```

StringBuilder对字符串的操作是通过char[]来实现的，通过默认构造器创建的StringBuilder，其内部创建的char[]的默认长度为16，当然可以调用重载的构造器传递初始长度（推荐这样，因为这样可以减少数组扩容次数，提高效率）。

```
/**
 * Constructs a string builder with no characters in it and an
 * initial capacity of 16 characters.
 */
public StringBuilder() {
    super(16);
}
```

#### 2. StringBuilder的append()方法

每次调用append(str)方法时，会首先判断数组长度是否足以添加传递来的字符串

```
/**
 * Appends the specified string to this character sequence.
 * <p>
 * The characters of the {@code String} argument are appended, in
 * order, increasing the length of this sequence by the length of the
 * argument. If {@code str} is {@code null}, then the four
 * characters {@code "null"} are appended.
 *
 * @param   str   a string.
 * @return  a reference to this object.
 */
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
/**
 * For positive values of {@code minimumCapacity}, this method
 * behaves like {@code ensureCapacity}, however it is never
 * synchronized.
 * If {@code minimumCapacity} is non positive due to numeric
 * overflow, this method throws {@code OutOfMemoryError}.
 */
private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value,
                newCapacity(minimumCapacity));
    }
}
```

如果传递的字符串长度 + 数组已存放的字符的长度 > 数组的长度，这时就需要进行数据扩容了

```
/**
 * Returns a capacity at least as large as the given minimum capacity.
 * Returns the current capacity increased by the same amount + 2 if
 * that suffices.
 * Will not return a capacity greater than {@code MAX_ARRAY_SIZE}
 * unless the given minimum capacity is greater than that.
 *
 * @param  minCapacity the desired minimum capacity
 * @throws OutOfMemoryError if minCapacity is less than zero or
 *         greater than Integer.MAX_VALUE
 */
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int newCapacity = (value.length << 1) + 2;
    if (newCapacity - minCapacity < 0) {
        newCapacity = minCapacity;
    }
    return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
        ? hugeCapacity(minCapacity)
        : newCapacity;
}
```

扩容规则如下：默认将数组长度设置为“ **(当前数组长度 \* 2) + 2**”，但如果按此规则扩容后的数组也不足以添加新的字符串，就需要将数组长度设置为“**数组内字符长度 + 传递的字符串长度**”。

因此假如我们知道拼接的字符串大概长度有100多字符，我们就可以设置初始长度150或200，这样就可以避免或减少数组扩容的次数，从而提高效率。

**总结：**

本文StringBuffer与StringBuilder的创建，append方法的原理讲解，对比了String、StringBuffer与StringBuilder异同，若有不对之处，请批评指正，谢谢！

