[TOC]

## 一. LinkedHashSet 初识

前面我们讲了HashMap说到了，HashMap 是一个利用数组存储key-value键值对的一个数据结构，然后又讲了HashMap中的数据是无序的，后面我们有了有序的要求，然后我们引入了LinkedHashMap来满足我们对顺序的要求，再到后面我们学习了HashSet这种数据结构，利用的是HashMap的Key 的唯一性来实现HashSet 的去重的目的，也介绍了HashSet是如何使用HashMap进行数据存储的(给了一个固定的value,其实存进去真正有意义的实key，也就是HashSet的元素)

其实学了前面几节[深度剖析HashMap](https://blog.csdn.net/king14bhhb/article/details/110294590)、[深度剖析LinkedHashMap](https://blog.csdn.net/king14bhhb/article/details/110294651)、[深度剖析HashSet](https://blog.csdn.net/king14bhhb/article/details/110679661) 到这里我们大概否能猜到这个数据结构是什么样了，LinkedHashMap 对HashMap 增加了双向链表维持了顺序，HashSet使用了HashMap弱化了Key-Value 中的Value 实现了去重且只有value集合的功能



那么我们大胆猜测LinkedHashSet 也HashSet 一样也在内部使用了HashMap，因为LinkedHashSet要维持元素之间的顺序，所以它使用的实HashMap的有序版本，也就是[LinkedHashMap](https://blog.csdn.net/king14bhhb/article/details/110294651) ,**如果是的话，那它和HashSet 一样也是个 No Face 的数据结构，没有自己的核心技术**

悄悄剧透一下，当你看到LinkedHashSet的代码的时候你就对它更无语了，总共五个方法，其中四个都是构造方法，这年头都是这么任性的吗，**没有核心技术还这么嚣张，不怕美国打压吗**



我们还是先来看一个例子，有一个最直观的认识，后面我们再去分析其原理和使用,假设我们有一个需求，首先就是集合中的元素不能重复，然后是按照添加的顺序输出集合中的元素，例如我们想输出网站今天的访问用户，然后按照他们的访问时间的顺序进行输出

因为是去重的，所以你最先想到的肯定是set，例如HashSet

```java
@Test
public void test() {
    HashSet<String> setString = new HashSet<> ();
    setString.add("星期一");
    setString.add("星期二");
    setString.add("星期三");
    setString.add("星期四");
    setString.add("星期五");
    setString.forEach(ele-> System.out.println(ele));
}
// 输出结果
星期二
星期三
星期四
星期五
星期一
```

上面的结果很明显不对，这个时候你深思熟虑之后，你想到了我们曾经学过的[LinkedHashMap](https://blog.csdn.net/king14bhhb/article/details/110294651) ，然后输出的时候不要value 即可，今天我们介绍一种新的数据结构LinkedHashSet

```java
@Test
public void iterator() {
    LinkedHashSet<String> setString = new LinkedHashSet<> ();
    setString.add("星期一");
    setString.add("星期二");
    setString.add("星期三");
    setString.add("星期四");
    setString.add("星期五");
    setString.forEach(ele-> System.out.println(ele));
}
// 输出结果
星期一
星期二
星期三
星期四
星期五
```

这下是不是就对了，按照国际惯例，有了直观的认识之后我们先看一下**LinkedHashSet 的操作手册和说明书，也就是类注释，包括方法注释**

### 1. LinkedHashSet 的说明书

在此之前我们先看一下LinkedHashSet 继承关系，先对它有一个直观的认识



![image-20201205150154231](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/05/15:01:55-image-20201205150154231.png)



```java
/**
 * <p>Hash table and linked list implementation of the <tt>Set</tt> interface, with predictable iteration order.  
 * Set 接口的具有可预测顺序的Hash表和链表特点实现(同时具有hash表功能和链表功能的set 接口实现类)，
 *  This implementation differs from <tt>HashSet</tt> in that it maintains a doubly-linked list running through all of its entries.  
 * 这个实现和HashSet的区别是它在它的全部元素之间维护了一个双向链表
 * This linked list defines the iteration ordering, which is the order in which elements were inserted into the set(<i>insertion-order</i>).
 * 这个链表维护了一个迭代顺序，这个顺序就是元素的插入顺序(nsertion-order)
 * Note that insertion order is <i>not</i> affected if an element is <i>re-inserted</i> into the set.
 * 需要注意的是这个插入顺序在元素重复插入的情况下是不生效的
 * <p>This implementation spares its clients from the unspecified, generally haotic ordering provided by {@link HashSet}, without incurring the
 * increased cost associated with {@link TreeSet}.  
 * 此实现通常使的调用者免于HashSet的混乱顺序，但是也不会引如TreeSet相关的成本增加。 
 * It can be used to produce a copy of a set that has the same order as the original, regardless of the original set's implementation:
  *它可以用来实现与原始顺序相同的一组副本 ，但是不用理会原始集的实现
 * This technique is particularly useful if a module takes a set on input, copies it, and later returns results whose order is determined by that of
 * the copy.  (Clients generally appreciate having things returned in the same order they were presented.)
 * 如果一个模块在输入时获取一个集合，复制它，然后返回结果，但是结果的顺序由复制的顺序决定，这种技术特别有用(客户端会非常喜欢返回值是按照它原来的顺序返回的)
 * <p>This class provides all of the optional <tt>Set</tt> operations, and permits null elements.  Like <tt>HashSet</tt>, it provides constant-time
 * performance for the basic operations (<tt>add</tt>, <tt>contains</tt> and <tt>remove</tt>), 
 * assuming the hash function disperses elements properly among the buckets.  
 * 这个类提供了Se的所有操作，并且允许null 值，和HashSet一样，如果hash函数合理的将数据分到各个桶中的话，它的基本操作例如add、contains、remove 都是常数级时间性能的
 * Performance is likely to be just slightly below that of <tt>HashSet</tt>, due to the added expense of maintaining the linked list，
 * 性能可能略低于HashSet，因为add的时候要维护链表
 * with one exception: Iteration over a <tt>LinkedHashSet</tt> requires time proportional to the <i>size</i> of the set, regardless of its capacity. 
 *Iteration over a <tt>HashSet</tt> is likely to be more expensive, requiring time proportional to its <i>capacity</i>.
 * 但是有一点是例外的，那就是遍历的时长只考虑szie 的大小，而不用管容量大小(其实就是buckets数组的大小)，迭代HashSet貌似代价更高，因为要考虑它的容量
 * <p>A linked hash set has two parameters that affect its performance: <i>initial capacity</i> and <i>load factor</i>.  They are defined precisely as for <tt>HashSet</tt>.  
 * LinkedHashSet 有两个影响它性能的参数，那就是initial capacity 和 load factor，但是这个两个参数更准确的说是为HashSet 定义的
 * Note, however, that the penalty for choosing an excessively high value for initial capacity is less severe for this class than for <tt>HashSet</tt>,
 * as iteration times for this class are unaffected by capacity.
 * 需要注意的实，然后nitial capacity 选择很高的代价对LinkedHashSet 的影响比HashSet要大，因为容量对LinkedHashSet的迭代来说是没有影响的
 *<p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access a linked hash set concurrently, and at least
 * one of the threads modifies the set, it <em>must</em> be synchronized
 * externally.  This is typically accomplished by synchronizing on some
 * object that naturally encapsulates the set.
 * 这一段，前面也解释过很多次了，就是线程安全，需要同步访问
 * If no such object exists, the set should be "wrapped" using the
 * {@link Collections#synchronizedSet Collections.synchronizedSet}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the set: <pre> Set s = Collections.synchronizedSet(new LinkedHashSet(...));</pre>
 * 然后可以尝试使用该类的包装类，创建包装类的方法是Set s = Collections.synchronizedSet(new LinkedHashSet(...))
 * <p>The iterators returned by this class's <tt>iterator</tt> method are
 * <em>fail-fast</em>: if the set is modified at any time after the iterator
 * is created, in any way except through the iterator's own <tt>remove</tt>
 * method, the iterator will throw a {@link ConcurrentModificationException}.
 * Thus, in the face of concurrent modification, the iterator fails quickly
 * and cleanly, rather than risking arbitrary, non-deterministic behavior at
 * an undetermined time in the future.
 *这个前面也解释过很多次了，就是关于快速失败的场景以及如何避免
 * @author  Josh Bloch
 * @see     Object#hashCode()
 * @since   1.4
 */

public class LinkedHashSet<E> extends HashSet<E> implements Set<E>, Cloneable, java.io.Serializable {
	... ...
}
```



### 2. LinkedHashSet 的构造方法

```java
// 指定初始容量和加载因子
public LinkedHashSet(int initialCapacity, float loadFactor) {
  	// 调用HashSet的构造方法，并且dummy=true
    super(initialCapacity, loadFactor, true);
}

/**
 * Constructs a new, empty linked hash set with the specified initial capacity and the default load factor (0.75).
 * 创建一个指定初始容量的LinkedHashSet，使用默认的0.75 作为加载因子
 */
public LinkedHashSet(int initialCapacity) {
    super(initialCapacity, .75f, true);
}

/**
 * Constructs a new, empty linked hash set with the default initial capacity (16) and load factor (0.75).
 * 无参构造，全部使用默认参数
 */
public LinkedHashSet() {
    super(16, .75f, true);
}

/**
 * Constructs a new linked hash set with the same elements as the specified collection.  The linked hash set is created with an initial capacity sufficient to hold the elements in the specified collection
 * and the default load factor (0.75).
 * 创建一个包含指定集合c中全部元素的LinkedHashSet，这个时候初始容量是Math.max(2*c.size(), 11)
 * @param c  the collection whose elements are to be placed into this set
 * @throws NullPointerException if the specified collection is null
 */
public LinkedHashSet(Collection<? extends E> c) {
  	//首先父类构造方法创建LinkedHashMap ，然后添加全部元素
    super(Math.max(2*c.size(), 11), .75f, true);
    addAll(c);
}
```

其实看完代码的注释之后，你忽然发现这个结构和HashSet 有点不一样，本应该是利用LinkedHashMap 来说在内存实现的有序的效果的，至少类中得有一个LinkedHashMap的引用吧，但是这里并没有，难道是我们的想法出错了，还记得我们在上一节介绍HashSet 构造方法的时候，也就是最后一个构造方法的时候说了一句话吗`关于这个我们后面在LinkedHashSet中详细讲解，其实就是在LinkedHashSet中会给这个参数一个True,那么这个时候map 就是LinkedHashMap的引用了，而不是HashMap` ,具体可以看[深度剖析HashSet](https://blog.csdn.net/king14bhhb/article/details/110679661) 一文

明白了吗，LinkedHashSet 继承了HashSet中的map 引用，并且给dummy 这个boolean 值了一个true,这个时候HashSet 中的map 就是LinkedHashMap，也就是说我们调用add 方法的时候，元素就被添加到LinkedHashMap 中去了，而不是HashMap，然后这下就可以维持顺序了

## 二. 总结

其实这一节我们没有讲什么，因为LinkedHashSet就是使用了LinkedHashMap这数据结构，本身没有太复杂的实现，所以学习的时候还是以[深度剖析HashMap](https://blog.csdn.net/king14bhhb/article/details/110294590)、[深度剖析LinkedHashMap](https://blog.csdn.net/king14bhhb/article/details/110294651)、[深度剖析HashSet](https://blog.csdn.net/king14bhhb/article/details/110679661) 这三篇文章为主

### LinkedHashSet 的顺序

1. 首先它只维护了插入顺序，因为HashSet 中创建的LinkedHashMap就是维护插入顺序的实现，而不是维护访顺序的(其实你仔细思考一下就知道为甚它只维护插入顺序了，而不维护访问顺序了，因为set 集合我们访问的方式往往是插入和遍历，很少去使用contains),那这个时候唯一的顺序就是插入顺序，而对于Map中我们的访问方式更多的是根据key 去访问value 

2. **插入顺序指的是首次插入顺序，重复插入的元素虽然会被重新插入，但是顺序不会维护**，这里为什么这么说呢，下面我也简单解释一下，当然最好你去看一下以前的文章

   ```java
   final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                  boolean evict) {
       Node<K,V>[] tab; Node<K,V> p; int n, i;
       if ((tab = table) == null || (n = tab.length) == 0)
           n = (tab = resize()).length;
       if ((p = tab[i = (n - 1) & hash]) == null)
           tab[i] = newNode(hash, key, value, null);
       else {
         	// 如果插入的元素不存在，则e=null，否则e!=null (具体为什么可以看前面的文章 )
           if (e != null) { // existing mapping for key
               V oldValue = e.value;
               if (!onlyIfAbsent || oldValue == null)
                   e.value = value;
             	// 这个维护的实访顺序，也就是元素存在的时候，被访问节点放大链表的头部，那这个时候不就是和我们上面说的不一样了吗，下面我们再看一下这个方法的实现
               afterNodeAccess(e);
               return oldValue;
           }
       }
       ++modCount;
       if (++size > threshold)
           resize();
     	// 这个维护的就是新插入的顺序，也就是当插入元素不存在的时候,这个时候新插入的节点，会被放到链表的尾部
       afterNodeInsertion(evict);
       return null;
   }
   ```

   ```java
   void afterNodeAccess(Node<K,V> e) { // move node to last
       LinkedHashMap.Entry<K,V> last;
     	// accessOrder 为true 的时候才维护这个顺序，也就是访问的元素放到链表的头部
       if (accessOrder && (last = tail) != e) {
           LinkedHashMap.Entry<K,V> p =
               (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
           p.after = null;
           if (b == null)
               head = a;
           else
               b.after = a;
           if (a != null)
               a.before = b;
           else
               last = b;
           if (last == null)
               head = p;
           else {
               p.before = last;
               last.after = p;
           }
           tail = p;
           ++modCount;
       }
   }
   
   ```

   ​	看到这里就明白了吧，因为accessOrder=false,所以二次插入的时候不会维护顺序，只有首次插入才会

### LinkedHashSet 的性能

LinkedHashSet 的插入性能可能略低于HashSet，因为它需要维护链表的顺序，

LinkedHashSet 的迭代性能应该是略高于HashSet的，因为它只需要按照链表的顺序进行迭代即可，也就是只考虑元素的多少，而不用考虑容量的大小

