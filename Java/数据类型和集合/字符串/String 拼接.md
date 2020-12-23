[TOC]

## StringBuilder与StringBuffer



StringBuilder与StringBuffer作用就是用来处理字符串，但String类本身也具备很多方法可以用来处理字符串，那么为什么还要引入这两个类呢？前面我们讲解到String 对象的不可变性，以及它的不足那就是创建新的对象，具体你可以查看文章[String进阶之不可变性](https://blog.csdn.net/king14bhhb/article/details/111415444)，因为它是不可变的，所以你对它的操作逻辑就体现在另外一个对象里，那就是你的操作新创建的对象。

这种操作最常见的就是字符串的拼接，所以我们几天学习的这两个类都是为了解决这个问题的，那既然都是为了解决这个问题的，为什么会有两个类的，我们后面慢慢分析

### 初识

首先看下面的例子

```java
    @Test
    public void testPerformance(){
        String str0 = "hello,world";

        long start = System.currentTimeMillis();
        for (int i = 0; i < 100000; i++) {
            str0 += i;
        }
        System.out.println(System.currentTimeMillis() - start);

        StringBuilder sb = new StringBuilder("hello,world");
        long start1 = System.currentTimeMillis();
        for (int i = 0; i < 100000; i++) {
            sb.append(i);
        }
        System.out.println(System.currentTimeMillis() - start1);

        StringBuffer sbf = new StringBuffer("hello,world");
        long start2 = System.currentTimeMillis();
        for (int i = 0; i < 100000; i++) {
            sbf.append(i);
        }
        System.out.println(System.currentTimeMillis() - start2);
    }
```

上述代码中3处循环完成了同样的功能，字符串拼接，执行的结果如下：

```
38833
4
4
```

可以看出执行时间差别很大，为了解决String不擅长的大量字符串拼接这种业务场景，所以我们引入了StringBuffer和StringBuilder.

首先我们分析一下为什么String在大量字符串拼接这种场景下这么慢？其实前面我们已经说到了，这我我们再解释一下

因为String本身不可变，我们对String的任何操作都会返回一个新的对象，然后当前String变量指向新的对象，而原来的String对象就会被GC回收，那么在循环中就会**大量快速的创建新的对象**，大量原来的对象会不断的被GC回收，消耗的时间是非常恐怖的，而且内存占用非常大。

但是我们从输出结果看到另外一个问题，那就是StringBuffer与StringBuilder的允许时间基本一致，那为什么需要定义两个功能相似的类呢？

接下来我们再看一段代码

```java
    @Test
    public void testSafe() throws InterruptedException {
        String str0 = "hello,world";

        StringBuilder sb = new StringBuilder(str0);
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                try {
                    TimeUnit.MILLISECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                sb.append("a");
            }).start();
        }


        StringBuffer sbf = new StringBuffer(str0);
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                try {
                    TimeUnit.MILLISECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                sbf.append("a");
            }).start();
        }
        // 等待工作线程运行结束
        while (Thread.activeCount()>2){

        }
        System.out.println("StringBuilder:"+sb.toString().length());
        System.out.println("StringBuffer:"+sbf.toString().length());
    }
```

输出结果

```
StringBuilder:109
StringBuffer:111
```

看出什么了吗，本来拼接完程度应该是111的，但是现在呢，StringBuilder是109，说明了StringBuilder不是安全的，也就是说再多线程的环境下，我们应该使用StringBuilder

下面我们对比了String、StringBuffer与StringBuilder的区别

| String                   | StringBuffer                               | StringBuilder          |
| ------------------------ | ------------------------------------------ | ---------------------- |
| final修饰，不可继承      | final修饰，不可继承                        | final修饰，不可继承    |
| 字符串常量，创建后不可变 | 字符串变量，可动态修改                     | 字符串变量，可动态修改 |
| 不存在线程安全问题       | 线程安全，所有public方法由synchronized修改 | 线程不安全             |
| 大量字符串拼接效率最低   | 大量字符串拼接效率非常高                   | 大量字符串拼接效率最高 |

StringBuffer与StringBuilder实现非常类似，下面以StringBuilder简单说明一下append()方法基本原理

## 源码剖析

这里我们以StringBuilder为例，

### 1. StringBuilder的构造方法

```
StringBuilder sb1 = new StringBuilder();
StringBuilder sb2 = new StringBuilder(100);
```

StringBuilder对字符串的操作是通过char[]来实现的，通过默认构造器创建的StringBuilder，其内部创建的char[]的默认长度为16，当然可以调用重载的构造器传递初始长度（推荐这样，因为这样可以减少数组扩容次数，提高效率）。

```java
     */
    public StringBuilder() {
        super(16);
    }

    /**
     * Constructs a string builder with no characters in it and an
     * initial capacity specified by the {@code capacity} argument.
     *
     * @param      capacity  the initial capacity.
     * @throws     NegativeArraySizeException  if the {@code capacity}
     *               argument is less than {@code 0}.
     */
    public StringBuilder(int capacity) {
        super(capacity);
    }

    /**
     * Constructs a string builder initialized to the contents of the
     * specified string. The initial capacity of the string builder is
     * {@code 16} plus the length of the string argument.
     *
     * @param   str   the initial contents of the buffer.
     */
    public StringBuilder(String str) {
        super(str.length() + 16);
        append(str);
    }

    /**
     * Constructs a string builder that contains the same characters
     * as the specified {@code CharSequence}. The initial capacity of
     * the string builder is {@code 16} plus the length of the
     * {@code CharSequence} argument.
     *
     * @param      seq   the sequence to copy.
     */
    public StringBuilder(CharSequence seq) {
        this(seq.length() + 16);
        append(seq);
    }
```

### 2. StringBuilder的append()方法

每次调用append(str)方法时，会首先判断数组长度是否足以添加传递来的字符串

```java
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

```java
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

## 总结

1. StringBuilder与StringBuffer 都是为了解决大量字符串拼接时的性能问题，其实就是为了解决String类在拼接过程中产生的大量对象的问题，因为这会导致大量内内存分配和GC 问题
2. StringBuilder与StringBuffer 二者之间的主要区别是线程是否安全，StringBuffer 通过对方法添加synchronized关键字保证了线程安全。 
3. StringBuilder与StringBuffer底层都是依赖字符数组实现的，不同于String 的是String 底层的字符数组是不可变的，也就是final 修饰的，其实这就是StringBuilder与StringBuffer可变的原因，动态扩展字符数组来实现的。
4. 为了提高StringBuilder与StringBuffer的性能我们可以通过设置合适的容量来避免数组库容。

