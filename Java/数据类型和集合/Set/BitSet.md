

[TOC]



## BitSet

BitSet类实现了一个按需增长的位向量,实际是由“二进制位”构成的一个Vector。每一位都是一个表示true或者false 的boolean 值。如果我们希望高效地存储这样只有两种类型的数据，就可以使用BitSet。

首先需要说明的是，BitSet并不属于集合框架，没有实现List或Map或者Set接口，BitSet更多的表示一种开关信息，对于海量不重复数据，利用索引表示数据的方式，将会大大节省空间使用。



### 位图

`vector of bits`也就是位图，由于可以用**非常紧凑的格式**来表示给定范围的连续数据而经常出现在各种算法设计中，这里**非常紧凑的格式**指的就是数组。

基本原理是，在一个特定的位置上(往往是数组下标的位置上)的值(开关)，0为没有出现过，1表示出现过，也就是说使用的时候可根据某一个位置是否为0表示此数(这个位置代表的数，往往是下标)是否出现过。

![image-20210119092558445](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210119092558445.png)

为了方便大家理解，我么可以对上图的数据解释一下，上图是我截取了一个位向量的一部分，也就是下标为1000-1009 的位置。这里开关为1(值为1)代表着此处的数据出现过，也就是说值为数组下标的数据，也就是1000出现过，同理1003 也出现过，然后就是1008 也出现过，其他的都没有出现过。

其实到这里我们就可以对位图有了一个基本的认识，说白了它就是用数组表示特定位置上的数据出现过没有，因为出现还是没有出现只会有两种结果，也就是true 和 false ，所以我们可以使用0 1 来表示，这里的0 1 指的是二进制中的0 1 ，这样我们用bit 来表示，而不是使用其他数据类型，例如Int 类型或者是String ,因为这种数据类型是比较耗空间的，这就是为什么我们使用这种数据结构的原因



#### 位图为什么省空间

#### 位图的使用场景

#### 位图的不足

#### 位图的面试题







**面试题**中也常出现，比如：统计40亿个数据中没有出现的数据，将40亿个不同数据进行排序等。

又如：现在有1千万个随机数，随机数的范围在1到1亿之间。现在要求写出一种算法，将1到1亿之间没有在随机数中的数求出来(百度)。

programming pearls上也有一个关于使用bitset来查找电话号码的题目。

Bitmap的常见扩展，是用2位或者更多为来表示此数字的更多信息，比如出现了多少次等。





### BitSet 初识

Bitset这种结构虽然简单，实现的时候也有一些细节需要主要。其中的关键是一些位操作，比如如何将指定位进行反转、设置、查询指定位的状态（0或者1）等。

#### 一. BitSet 说明书

```java
/**
 * This class implements a vector of bits that grows as needed. Each
 * component of the bit set has a {@code boolean} value. The
 * bits of a {@code BitSet} are indexed by nonnegative integers.
 * Individual indexed bits can be examined, set, or cleared. One
 * {@code BitSet} may be used to modify the contents of another
 * {@code BitSet} through logical AND, logical inclusive OR, and
 * logical exclusive OR operations.
 *
 * <p>By default, all bits in the set initially have the value
 * {@code false}.
 *
 * <p>Every bit set has a current size, which is the number of bits
 * of space currently in use by the bit set. Note that the size is
 * related to the implementation of a bit set, so it may change with
 * implementation. The length of a bit set relates to logical length
 * of a bit set and is defined independently of implementation.
 *
 * <p>Unless otherwise noted, passing a null parameter to any of the
 * methods in a {@code BitSet} will result in a
 * {@code NullPointerException}.
 *
 * <p>A {@code BitSet} is not safe for multithreaded use without
 * external synchronization.
 *
 * @author  Arthur van Hoff
 * @author  Michael McCloskey
 * @author  Martin Buchholz
 * @since   JDK1.0
 */
 public class BitSet implements Cloneable, java.io.Serializable {
    /*
     * BitSets are packed into arrays of "words."  Currently a word is
     * a long, which consists of 64 bits, requiring 6 address bits.
     * The choice of word size is determined purely by performance concerns.
     */
    private final static int ADDRESS_BITS_PER_WORD = 6;
    private final static int BITS_PER_WORD = 1 << ADDRESS_BITS_PER_WORD;
    private final static int BIT_INDEX_MASK = BITS_PER_WORD - 1;

    /* Used to shift left or right for a partial word mask */
    private static final long WORD_MASK = 0xffffffffffffffffL;

    /**
     * @serialField bits long[]
     *
     * The bits in this BitSet.  The ith bit is stored in bits[i/64] at
     * bit position i % 64 (where bit position 0 refers to the least
     * significant bit and 63 refers to the most significant bit).
     */
    private static final ObjectStreamField[] serialPersistentFields = {
        new ObjectStreamField("bits", long[].class),
    };

    /**
     * The internal field corresponding to the serialField "bits".
     * BitSet的底层实现是使用long数组作为内部存储结构的，所以BitSet的大小为long类型大小(64位)的整数倍。
     */
    private long[] words;

    /**
     * The number of words in the logical size of this BitSet.
     */
    private transient int wordsInUse = 0;

    /**
     * Whether the size of "words" is user-specified.  If so, we assume
     * the user knows what he's doing and try harder to preserve it.
     */
    private transient boolean sizeIsSticky = false;

    /* use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = 7997698588986878753L;

 }
```

#### 二. 构造方法

第一个构造方法创建一个默认的对象：

```java
/**
 * Creates a new bit set. All bits are initially {@code false}.
 */
public BitSet() {
  	// BITS_PER_WORD=64
    initWords(BITS_PER_WORD);
    sizeIsSticky = false;
}
```

第二个方法允许用户指定初始大小。所有位初始化为0。

```java
/**
 * Creates a bit set whose initial size is large enough to explicitly
 * represent bits with indices in the range {@code 0} through
 * {@code nbits-1}. All bits are initially {@code false}.
 *
 * @param  nbits the initial size of the bit set
 * @throws NegativeArraySizeException if the specified initial size
 *         is negative
 */
public BitSet(int nbits) {
    // nbits can't be negative; size 0 is OK
    if (nbits < 0)
        throw new NegativeArraySizeException("nbits < 0: " + nbits);

    initWords(nbits);
    sizeIsSticky = true;
}
```

上面两个构造方法，实际的空间是由initWords方法控制的，在这个方法里面，我们实例化了一个long型数组

```java
private void initWords(int nbits) {
    words = new long[wordIndex(nbits-1) + 1];
}
/**
 * Given a bit index, return word index containing it.
 */
private static int wordIndex(int bitIndex) {
    return bitIndex >> ADDRESS_BITS_PER_WORD;
}
```

这里引入了另外一个方法wordIndex，那么wordIndex又是干嘛的呢，这里涉及到一个常量ADDRESS_BITS_PER_WORD，先解释一下，源码中的定义如下：

```java
private final static int ADDRESS_BITS_PER_WORD = 6;  
```


很明显2^6=64（8个byte，对应long型）。所以，当我们传进129作为参数的时候，我们会申请一个long[(129-1)>>6+1]也就是long[3]的数组。BitSet是通过一个整数来表示一定的bit位。

#### 三. set 方法

```java
/**
 * Sets the bit at the specified index to {@code true}.
 *
 * @param  bitIndex a bit index
 * @throws IndexOutOfBoundsException if the specified index is negative
 * @since  JDK1.0
 */
public void set(int bitIndex) {
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);

    int wordIndex = wordIndex(bitIndex);
    expandTo(wordIndex);

    words[wordIndex] |= (1L << bitIndex); // Restores invariants

    checkInvariants();
}
```

这个方法将bitIndex位上的值由false设置为true。首先确定修改的是long型数组中的哪一个元素，其次决定是否需要扩容