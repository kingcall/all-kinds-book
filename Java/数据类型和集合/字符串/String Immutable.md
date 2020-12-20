[TOC]

## String 的不可变性

其实前面的文章里提到了不可变性，也说到了不可变的实现方式，可以看[Java数据类型—String基础](https://blog.csdn.net/king14bhhb/article/details/111314482) 其实看到这个名字大家就知道这个事情就没有完，因为我还会写String 进阶和String 实战和优化系列。

### 不可变初识

这里我们先看一下不可变性的定义是什么，**这意味着一旦创建了一个字符串对象，就不能修改该字符串的内容。如果修改该字符串，则会使用修改后的值创建一个新的字符串对象，保留原始字符串不变**，接下来我们看一下这个怎么理解

```java
@Test
public void test(){
    String str = "Test";
    str.concat(" String");
    System.out.println("Original String- " + str);
}
// 输出结果
Original String- Test
```

可以看到我们的字符串对象str并没有在调用concat 方法后发生任何变化，这就是String 的不可变性，这是因为`  str.concat(" String")` 会返回一个新的字符串对象我们可以看一下源代码

```java
public String concat(String str) {
  	// 判断传入的字符串是不是空串
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
  	// 添加到value 数组中去
    char buf[] = Arrays.copyOf(value, len + otherLen);
    str.getChars(buf, len);
  	// 返回新创建的字符串
    return new String(buf, true);
}
```

由于` str.concat(" String")`调用之后，也就是`Test String”`没有被引用，所以它会被垃圾回收器所回收

### 不可变性的实现方式

当用final修饰类的时候，此类不可被继承，即final类没有子类。这样可以用final保证用户调用时动作的一致性，可以防止子类覆盖情况的发生。

开始之前我们先看一段代码，这是我自己定义的字符串类型——BuerSting

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

从上面的运行结果来看，我定义的这个字符串类是可变的，虽然它也是final修饰的，所以可以看出仅仅依靠的final 修饰是不能保证不可变性的，接下来我们看一下Java 是怎么保证String 的不可变的呢

1. 存储字符的数组，该数组由final关键字修饰，不可变，但是这里你可易通过继承的方式修改String 的某些行为使其可变
2. String 类是final 修饰的，这个时候就避免了因为继承而改变某些方法的行为而引发的不安全问题

### 为什么要不可变

#### 为了使用常量池

前面我们说到，String 作为Java 中使用最广泛的数据类型，Java 为了提高String 的性能，做了大量的工作，其中之一就是起到缓存左右的字符串常量池

因为通过串池的这种设计方式，可能导致多个字符串变量，持有同一个字符串的引用，那么如果是可变的话，通过一个变量改变了字符串的内容的话，就会导致其他字符串的变量的引用的内容发生变化，所以这就是字符串为什么设计成不可变的。其实就是共享对象的不可变性，否则就会存在安全问题

#### 线程安全

在Java中使字符串不可变也使字符串对象是线程安全的。字符串对象可以在多个线程之间共享，即使任何线程修改了原始对象，由于其不变性，修改它的线程将得到一个新的对象，因此原始对象保持不变。

#### Hash的Key在

Java中使字符串不可变也使它们成为HashMap等基于散列的数据结构中的一个很好的key的候选对象。因为字符串一旦赋值就不能修改，这意味着为任何字符串计算的哈希代码也不会改变。这有助于缓存HashCode，而不是每次都计算HashCode，使哈希更高效。



### 不可变性的不足

不可变性最大的不足是任何修改都会产生新的对象，而新的对象就会引起创建对象的开销，例如下面这种就会导致创建大量对象，然后又抛弃不用

```java
String str = "Test";
str = str.concat(" String").concat("Another").concat("String");
```

关于这个的解决方案就是 StringBuffer 和 StringBuilder



## 总结

1. 不可变性的实现方式
2. 为什么需要不可变性或者说是不可变性的好处
3. 不可变性的不足