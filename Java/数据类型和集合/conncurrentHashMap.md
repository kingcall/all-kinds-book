[TOC]

## 概论

ConcurrentHashMap融合了hashtable和hashmap二者的优势。hashtable是做了同步的，即线程安全，hashmap未考虑同步。所以hashmap在单线程情况下效率较高。hashtable在的多线程情况下，同步操作能保证程序执行的正确性。但是hashtable是阻塞的，每次同步执行的时候都要锁住整个结构导致性能底下，ConcurrentHashMap正是为了解决这个问题而诞生的

ConcurrentHashMap除了加锁，原理上与HashMap无太大区别。另外，HashMap 的键值对允许有null，但是ConCurrentHashMap 都不允许。

HashTable 使用一把锁（锁住整个链表结构）处理并发问题，多个线程竞争一把锁，容易阻塞，**ConcurrentHashMap** 不论是1.7 还是1.8 都降低了锁的粒度

### ConcurrentHashMap 的继承关系



![image-20201130211505990](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/30/21:15:06-image-20201130211505990.png)

其实这里我们注意到ConcurrentHashMap比HashMap多继承了一个接口，那就是ConcurrentMap,ConcurrentHashMap在JUC 包下的而HashMap是在JU包下，ConcurrentHashMap是在Java1.5 之后才有的，它的作者就是大名鼎鼎的Doug Lea，当然整个JUC 包几乎都是他写的，哈哈 大佬就是大佬



ConcurrentMap 只是一个接口，当然实现不了线程安全的操作方式，但是我也将它贴了出来，大家可以看一下它的注释，了解一下

```
/**
 * A {@link java.util.Map} providing thread safety and atomicity
 * guarantees.
 *
 * <p>Memory consistency effects: As with other concurrent
 * collections, actions in a thread prior to placing an object into a
 * {@code ConcurrentMap} as a key or value
 * <a href="package-summary.html#MemoryVisibility"><i>happen-before</i></a>
 * actions subsequent to the access or removal of that object from
 * the {@code ConcurrentMap} in another thread.
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @since 1.5
 * @author Doug Lea
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface ConcurrentMap<K, V> extends Map<K, V> {
}
```




### 分段锁技术
- jdk 1.7 采用分段锁技术(ReentrantLock + Segment )，整个 Hash 表被分成多个段，每个段中会对应一个 Segment 段锁，段与段之间可以并发访问，但是多线程想要操作同一个段是需要获取锁的。所有的 put，get，remove 等方法都是根据键的 hash 值对应到相应的段中，然后尝试获取锁进行访问。

![image-20201126174152342](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:41:52-image-20201126174152342.png)

这样锁粒度的基于 Segment，包含多个 HashEntry，所以整个ConcurrentHashMap的**并发度就是分段数**

**put流程**

其实发现整个流程和HashMap非常类似，只不过是先定位到具体的Segment，然后通过ReentrantLock去操作而已，后面的流程我就简化了，因为和HashMap基本上是一样的。

1. 计算hash，定位到segment，segment如果是空就先初始化
2. 使用ReentrantLock加锁，如果获取锁失败则尝试自旋，自旋超过次数就阻塞获取，保证一定获取锁成功
3. 遍历HashEntry，就是和HashMap一样，数组中key和hash一样就直接替换，不存在就再插入链表，链表同样

![image-20201201082354444](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/01/08:23:54-image-20201201082354444.png)

**get流程**

get也很简单，key通过hash定位到segment，再遍历链表定位到具体的元素上，需要注意的是value是volatile的，所以get是不需要加锁的。

### cas

jdk 1.8 取消了基于 Segment 的分段锁思想，改用(CAS + synchronized +Node)控制并发操作，在某些方面提升了性能,锁的粒度降低了，并发性能就上来了(Segment->Node 的一个转变)

并且追随 1.8 版本的 HashMap 底层实现，使用数组+链表+红黑树进行数据存储。本篇主要介绍 1.8 版本的 ConcurrentHashMap 的具体实现。

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/30/21:36:03-640.png)

**ConcurrentHashMap 在 JDK 1.8 中，为什么要使用内置锁 synchronized 来代替重入锁 ReentrantLock**

①、粒度降低了；
②、JVM 开发团队没有放弃 synchronized，而且基于 JVM 的 synchronized 优化空间更大，更加自然。
③、在大量的数据操作下，对于 JVM 的内存压力，基于 API 的 ReentrantLock 会开销更多的内存

> 干儿子再好，总不如亲儿子好

程序运行时能够同时更新 ConccurentHashMap 且不产生锁竞争的最大线程数。默认为 16，且可以在构造函数中设置。当用户设置并发度时，ConcurrentHashMap 会使用大于等于该值的最小2幂指数作为实际并发度（假如用户设置并发度为17，实际并发度则为32）**其实并发度就是数组大小**



## ConcurrentHashMap 的内部关键元素





### 注释

```
A hash table supporting full concurrency of retrievals and high expected concurrency for updates. 
一个支持高并发更新和全量并发获取数据的hash table 
This class obeys the same functional specification as {@link java.util.Hashtable}, and
includes versions of methods corresponding to each method of {@code Hashtable}.
这个类和Hashtable中方法的使用一样，并且包含Hashtable中所有方法的变体
However, even though all operations are thread-safe, retrieval operations do <em>not</em> entail locking,and there is <em>not</em> any support for locking the entire table
in a way that prevents all access.
即使在支持全部操作线程安全的前提下，get 操作也不需要加锁，也不支持哪种锁住整个table的操作，因为哪种操作会阻止其他所有操作
This class is fully interoperable with {@code Hashtable} in programs that rely on its
thread safety but not on its synchronization details.
在哪些依赖线程安全而又不想关注同步细节的程序中，这个类可与Hashtable配合使用(在哪些需要线程安全的程序中，这个类可以替代Hashtable)
```



```
Retrieval operations (including {@code get}) generally do not block, so may overlap with update operations (including {@code put} and {@code remove}).

Retrievals reflect the results of the most recently completed update operations holding upon their onset. (More formally, an update operation for a given key bears a happens-before relation with any (non-null) retrieval for that key reporting the updated value) 

For aggregate operations such as {@code putAll} and {@code clear}, concurrent retrievals may
reflect insertion or removal of only some entries.  Similarly,
Iterators, Spliterators and Enumerations return elements reflecting the
state of the hash table at some point at or since the creation of the
iterator/enumeration.  They do <em>not</em> throw {@link
java.util.ConcurrentModificationException ConcurrentModificationException}.
However, iterators are designed to be used by only one thread at a time.
Bear in mind that the results of aggregate status methods including
{@code size}, {@code isEmpty}, and {@code containsValue} are typically
useful only when a map is not undergoing concurrent updates in other threads.
Otherwise the results of these methods reflect transient states
that may be adequate for monitoring or estimation purposes, but not
for program control.
```



①、重要的常量：
private transient volatile int sizeCtl;
当为负数时，-1 表示正在初始化，-N 表示 N - 1 个线程正在进行扩容；
当为 0 时，表示 table 还没有初始化；
当为其他正数时，表示初始化或者下一次进行扩容的大小。



②、数据结构：
Node 是存储结构的基本单元，继承 HashMap 中的 Entry，用于存储数据；
TreeNode 继承 Node，但是数据结构换成了二叉树结构，是红黑树的存储结构，用于红黑树中存储数据；
TreeBin 是封装 TreeNode 的容器，提供转换红黑树的一些条件和锁的控制。



③、存储对象时（put() 方法）：
1.如果没有初始化，就调用 initTable() 方法来进行初始化；
2.如果没有 hash 冲突就直接 CAS 无锁插入；
3.如果需要扩容，就先进行扩容；
4.如果存在 hash 冲突，就加锁来保证线程安全，两种情况：一种是链表形式就直接遍历到尾端插入，一种是红黑树就按照红黑树结构插入；
5.如果该链表的数量大于阀值 8，就要先转换成红黑树的结构，break 再一次进入循环
6.如果添加成功就调用 addCount() 方法统计 size，并且检查是否需要扩容。



④、扩容方法 transfer()：默认容量为 16，扩容时，容量变为原来的两倍。
helpTransfer()：调用多个工作线程一起帮助进行扩容，这样的效率就会更高。



⑤、获取对象时（get()方法）：
1.计算 hash 值，定位到该 table 索引位置，如果是首结点符合就返回；
2.如果遇到扩容时，会调用标记正在扩容结点 ForwardingNode.find()方法，查找该结点，匹配就返回；
3.以上都不符合的话，就往下遍历结点，匹配就返回，否则最后就返回 null。



### DEFAULT_CONCURRENCY_LEVEL

(没有用了，为了兼容老版本	)

The default concurrency level for this table. 

Unused but defined for compatibility with previous versions of this class.



### 和HashMap 中一样的一些变量

```
private static final int MAXIMUM_CAPACITY = 1 << 30;
// The default initial table capacity.  Must be a power of 2 最少是1 
private static final int DEFAULT_CAPACITY = 16;
```



### Node

这个内部类在HashMap中也有，但是这里我们依然将它贴了出来，因为它和HashMap 中的有不一样之处，关于这一点你从这个类的注都可以看出

```
 /**
  * Key-value entry.  This class is never exported out as a
  * user-mutable Map.Entry (i.e., one supporting setValue; see
  * MapEntry below), but can be used for read-only traversals used
  * in bulk tasks.  Subclasses of Node with a negative hash field
  * are special, and contain null keys and values (but are never
  * exported).  Otherwise, keys and vals are never null.
  */
static class Node<K,V> implements Map.Entry<K,V> {
     final int hash;
     final K key;
     volatile V val;
     volatile Node<K,V> next;

     Node(int hash, K key, V val, Node<K,V> next) {
         this.hash = hash;
         this.key = key;
         this.val = val;
         this.next = next;
     }

     public final K getKey()       { return key; }
     public final V getValue()     { return val; }
     public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
     public final String toString(){ return key + "=" + val; }
     public final V setValue(V value) {
         throw new UnsupportedOperationException();
     }

     public final boolean equals(Object o) {
         Object k, v, u; Map.Entry<?,?> e;
         return ((o instanceof Map.Entry) &&
                 (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                 (v = e.getValue()) != null &&
                 (k == key || k.equals(key)) &&
                 (v == (u = val) || v.equals(u)));
     }

     /**
     
      * Virtualized support for map.get(); overridden in subclasses.
      */
     Node<K,V> find(int h, Object k) {
         Node<K,V> e = this;
         if (k != null) {
             do {
                 K ek;
                 if (e.hash == h &&
                     ((ek = e.key) == k || (ek != null && k.equals(ek))))
                     return e;
             } while ((e = e.next) != null);
         }
         return null;
     }
}
```



## debug put

1. 首先计算hash，遍历node数组，如果node是空的话，就通过CAS+自旋的方式初始化
2. 如果当前数组位置是空则直接通过CAS自旋写入数据
3. 如果hash==MOVED，说明需要扩容，执行扩容
4. 如果都不满足，就使用synchronized写入数据，写入数据同样判断链表、红黑树，链表写入和HashMap的方式一样，key hash一样就覆盖，反之就尾插法，链表长度超过8就转换成红黑树

![image-20201201082620228](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/01/08:26:20-image-20201201082620228.png)

### 用户调用入口put 方法

从注释我们知道，key-value 都不能是null 和HashTable 一样

```
/**
* Maps the specified key to the specified value in this table.
* Neither the key nor the value can be null.
* @return the previous value associated with {@code key}, or
*         {@code null} if there was no mapping for {@code key}
* @throws NullPointerException if the specified key or value is null
*/
public V put(K key, V value) {
    return putVal(key, value, false);
}
```

这里注意一下，HashMap中，在调用putVal方法的时候已经计算了Hash 值,下面是HashMap 的put 方法

```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

### 核心方法  putVal

```
final V putVal(K key, V value, boolean onlyIfAbsent) {
		// 这里来了一个控制检测，这是HashMap 没有的，但是这里和HashTable 也不一样
		// 但是也和HashTable不一样， HashTable 只检测了Value ,然后key 的NullPointerException是在调用key.hashCode()的时候抛出来的
    if (key == null || value == null) throw new NullPointerException();
    //计算hash 值， (h ^ (h >>> 16)) & HASH_BITS  这里还是一样的(h ^ (h >>> 16))，高16位异或低16位
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        // 判断当前位置(桶)是不是空桶,tabAt 方法获取特定位置((n - 1) & hash)) 的元素
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
        		 // no lock when adding to empty bin  直接返给当前元素到桶，放入成功则跳出训话
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                break;                  
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        // 下面才是正常情况下的放入流程    
        else {
            V oldVal = null;
            // 考点 synchronized 加锁，加的是一个个的节点，也就是Node 
            synchronized (f) {
            		// 双重检测，因为测试的f 可能已经被其他线程修改了
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```



```
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```



```
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
```



### 计算Hash值的方法spread

前面我们没有解释这个计算Hash的方法，这里我们看一下它的注释翻译一下，便于大家理解，这种计算hash值的方法，其实在HashMap 那一节我们说了为什么这么设计

```
/**
 * Spreads (XORs) higher bits of hash to lower and also forces top bit to 0
 hash值的高位异或hash值的地位，并且强制要求hash 值的首位是0（hashmap 中是没有的，hashtable 中有），所以这就是为什么要与0x7fffffff
 Because the table uses power-of-two masking, sets of hashes that vary only in bits above the current mask will always collide.
 因为table 使用2的指数作为掩码，其实就是数组大小，如果很多hash值仅当前掩码上的部分位上发生变化将经常导致hash 冲突
 (Among known examples are sets of Float keys holding consecutive whole numbers in small tables.) 
 So we apply a transform that spreads the impact of higher bits downward. There is a tradeoff between speed, utility, and
 * quality of bit-spreading. Because many common sets of hashes
 * are already reasonably distributed (so don't benefit from
 * spreading), and because we use trees to handle large sets of
 * collisions in bins, we just XOR some shifted bits in the
 * cheapest possible way to reduce systematic lossage, as well as
 * to incorporate impact of the highest bits that would otherwise
 * never be used in index calculations because of table bounds.
 */
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS;
}
```

### 初始化table initTable

```
/**
 * Initializes table, using the size recorded in sizeCtl.
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

