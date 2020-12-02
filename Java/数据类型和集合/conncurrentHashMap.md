[TOC]

## 概论

### 为什么还需要ConcurrentHashMap

ConcurrentHashMap融合了hashtable和hashmap二者的优势，或者说它是HashMap的一个线程安全的、支持高效并发的版本

Hashtable是做了同步的，即线程安全，hashmap未考虑同步。所以hashmap在单线程情况下效率较高。hashtable在的多线程情况下，同步操作能保证程序执行的正确性。但是hashtable是阻塞的，每次同步执行的时候都要锁住整个结构导致性能底下，ConcurrentHashMap正是为了解决这个问题而诞生的

ConcurrentHashMap除了加锁，原理上与HashMap无太大区别。另外，HashMap 的键值对允许有null，但是ConCurrentHashMap 都不允许。

HashTable 使用一把锁（锁住整个链表结构）处理并发问题，多个线程竞争一把锁，容易阻塞，**ConcurrentHashMap** 不论是1.7 还是1.8 都降低了锁的粒度

Problem with Hashtable or synchronized Map (**Collections.synchronizedMap()**)is that all its methods are synchronized on a common lock thus only a single thread can access it at any given time, even for read operations. ConcurrentHashMap in Java tries to address these issues.

### ConcurrentHashMap 的性能优势

第一点：加锁的范围，HashTable 是整个table 加锁，而ConcurrentHashMap只加锁某一个桶或者节点，所以二者在并发性上存在很大差异

第二点：设计结构上，ConcurrentHashMap在链表长度过长的时候会转化成红黑树，而HashTable不会，这就导致了做获取数据上的性能差异

第三点：读取数据时不加锁的，而且是并发读取不加锁

第四点：其实就是在上述两点之外的一些细节上的东西了，例如hash 值的计算方式什么的，代码的优化上面的一些问题了，因为HashTable的代码现在不被更新了。

### ConcurrentHashMap 的继承关系





![image-20201130211505990](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/30/21:15:06-image-20201130211505990.png)

其实这里我们注意到ConcurrentHashMap比HashMap多继承了一个接口，那就是ConcurrentMap,ConcurrentHashMap在JUC 包下的而HashMap是在JU包下，ConcurrentHashMap是在Java1.5 之后才有的，它的作者就是大名鼎鼎的Doug Lea，当然整个JUC 包几乎都是他写的，哈哈 大佬就是大佬



ConcurrentMap 只是一个接口，当然实现不了线程安全的操作方式，但是我也将它贴了出来，大家可以看一下它的注释，了解一下

```java
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

####  ConcurrentHashMap 的构造方法



- **ConcurrentHashMap()**- 创建一个新的，空的HashMap,初始容量大小是16
- **ConcurrentHashMap(int initialCapacity)** 创建一个指定容量大小的HashMap
- **ConcurrentHashMap(int initialCapacity, float loadFactor)**- Creates a new, empty map with an initial table size based on the given number of elements (initialCapacity) and initial table density (loadFactor).
- **ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel)**- Creates a new, empty map with an initial table size based on the given number of elements (initialCapacity), table density (loadFactor), and number of concurrently updating threads (concurrencyLevel).
- **ConcurrentHashMap(Map<? extends K,? extends V> m)**- Creates a new map with the same mappings as the given map.


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

```java
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
读取操作例如get操作不加锁，因此可能导致和更新操作有所重叠(冲突)
Retrievals reflect the results of the most recently completed update operations holding upon their onset. 
(More formally, an update operation for a given key bears a happens-before relation with any (non-null) retrieval for that key reporting the updated value) 
读取操作通常获取的是最近完成的更新操作的结果，这可能意味着读取操作可能无法获取当前/正在进行的值（这是一个缺点）
For aggregate operations such as {@code putAll} and {@code clear}, concurrent retrievals may
reflect insertion or removal of only some entries.  Similarly,
同样对于聚合操作（例如putAll和clear）可以在整个地图上运行的情况，并发检索可能只反映了某些条目的插入或删除（单独锁定的另一个缺点）。 因为读取操作没有被阻塞，但是某些写入（位于同一存储桶中）可能仍然被阻塞。
```



```
The table is dynamically expanded when there are too many collisions 
(i.e., keys that have distinct hash codes but fall into the same slot modulo the table size), with the expected average effect of maintaining roughly two bins per mapping (corresponding to a 0.75 load factor threshold for resizing). There may be much variance around this average as mappings are added and removed, but overall, this maintains a commonly accepted time/space tradeoff for hash tables. However, resizing this or any other kind of hash table may be a relatively slow operation. When possible, it is a good idea to provide a size estimate as an optional initialCapacity constructor argument. An additional optional loadFactor constructor argument provides a further means of customizing initial table capacity by specifying the table density to be used in calculating the amount of space to allocate for the given number of elements. Also, for compatibility with previous versions of this class, constructors may optionally specify an expected concurrencyLevel as an additional hint for internal sizing. Note that using many keys with exactly the same hashCode() is a sure way to slow down performance of any hash table. To ameliorate impact, when keys are Comparable, this class may use comparison order among keys to help break ties.
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

```java
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



#### ForwardingNode



```java
/**
 * A node inserted at head of bins during transfer operations.
 */
static final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab) {
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }

    Node<K,V> find(int h, Object k) {
        // loop to avoid arbitrarily deep recursion on forwarding nodes
        outer: for (Node<K,V>[] tab = nextTable;;) {
            Node<K,V> e; int n;
            if (k == null || tab == null || (n = tab.length) == 0 ||
                (e = tabAt(tab, (n - 1) & h)) == null)
                return null;
            for (;;) {
                int eh; K ek;
                if ((eh = e.hash) == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
                if (eh < 0) {
                    if (e instanceof ForwardingNode) {
                        tab = ((ForwardingNode<K,V>)e).nextTable;
                        continue outer;
                    }
                    else
                        return e.find(h, k);
                }
                if ((e = e.next) == null)
                    return null;
            }
        }
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

```java
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

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

### 核心方法  putVal

```java
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
        		 //  如果是空桶，利用 CAS 尝试写入，失败则自旋保证成功，最后跳出循环
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                break;                  
        }
        // 如果不是空桶，则判断当前位置元素的 hashcode == MOVED == -1,如果是则需要进行扩容。
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        // 下面才是正常情况下的放入流程 ,利用 synchronized锁住，然后写入数据。   
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
          // 如果数量大于 TREEIFY_THRESHOLD 则要转换为红黑树
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



```java
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```



```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
```



### 计算Hash值的方法spread

前面我们没有解释这个计算Hash的方法，这里我们看一下它的注释翻译一下，便于大家理解，这种计算hash值的方法，其实在HashMap 那一节我们说了为什么这么设计

```java
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

```java
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



### 

```java
/**
 * Helps transfer if a resize is in progress.
 */
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    Node<K,V>[] nextTab; int sc;
    if (tab != null && (f instanceof ForwardingNode) && (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
        int rs = resizeStamp(tab.length);
        while (nextTab == nextTable && table == tab &&
               (sc = sizeCtl) < 0) {
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                transfer(tab, nextTab);
                break;
            }
        }
        return nextTab;
    }
    return table;
}
```

## 正确使用ConcurrentHashMap

首先看一下这段代码，看上去好像没什么问题，但是当你多运行几次你就会发现每次结果都可能不一样，因为直觉上是多线程，然后你选择了线程安全的ConcurrentHashMap，那这样就万事无忧了吗

```java
@Test
public  void correctUse() throws InterruptedException {
    ConcurrentHashMap<String, Integer> wordMap = new ConcurrentHashMap<>();
    String word = "a";

    for (int i =0 ; i < 1000; i++) {
        new Thread(()->{
            Integer prevValue = wordMap.get(word);
            Integer newValue = (prevValue == null ? 1 : prevValue + 1);
            wordMap.put(word, newValue);
        }).start();
    }
    TimeUnit.SECONDS.sleep(3);
    System.out.println( wordMap.get(word));
}
```

虽然神器在手，但是要想说走就走，还是得注意场合，这里的问题就是在于那个加一的地方不是线程安全的，所以正确的做法是下面这样的

```java
public  void correctUse() throws InterruptedException {
    ConcurrentHashMap<String, Integer> wordMap = new ConcurrentHashMap<>();
    String word = "a";

    for (int i =0 ; i < 1000; i++) {
        new Thread(()->{
            synchronized (word){
                Integer prevValue = wordMap.get(word);
                Integer newValue = (prevValue == null ? 1 : prevValue + 1);
                wordMap.put(word, newValue);
            }
        }).start();
    }
    TimeUnit.SECONDS.sleep(3);
    System.out.println( wordMap.get(word));
}
```

看到这里，有没有人出来怼我，哈哈，这样的话我还有ConcurrentHashMap干嘛，我直接HashMap不香吗，你说东方不败练的葵宝典底和岳不群练的辟邪剑谱能一样吗

所以我想表明的是ConcurrentHashMap虽好，还是得注意使用场合，这个场合HashMap也挺香的。所以说，ConcurrentHashMap 只适合在put get 这种原子性操作下的多线程环境使用

所以这里我总结一下，ConcurrentHashMap适用于多线程的原子操作，所以它只能保证它的get put 操作是线程安全的，其次适用于读多于写的场景

### ConcurrentHashMap 提供的原子性操作

我知道上面的操作，不适合你们这群追求完美的人，毕竟你们练习的都是葵花宝典这种没有缺陷的武功啊，下面看看这种操作,就很符合你们的风格了

```java
 @Test
 public  void correctUse() throws InterruptedException {
     ConcurrentHashMap<String, Integer> wordMap = new ConcurrentHashMap<>();
     String word = "a";

     for (int i =0 ; i < 1000; i++) {
         new Thread(()->{
             wordMap.compute(word, (k,v)-> v == null ? 1 : v + 1);
         }).start();
     }
     TimeUnit.SECONDS.sleep(3);
     System.out.println( wordMap.get(word));
 }
```

但是需要注意的是，这个方法其实是锁了整个Table 的，所以其实本质上和上面的那种synchronized锁效果是一样的，当然这样的操作，还有 **computeIfAbsent**, **computeIfPresent**, **merge**, **putIfAbsent**



### 安全失败的遍历操作

其实以前我们看到过的是快速失败的设计，意思是说在你遍历的过程中，如果原始结构改变，遍历就会提前失败

```java
@Test
public void safeItrator(){
    // Creating ConcurrentHashMap
    Map<String, String> cityTemperatureMap = new HashMap<String, String>();

    // Storing elements
    cityTemperatureMap.put("Delhi", "24");
    cityTemperatureMap.put("Mumbai", "32");
    cityTemperatureMap.put("Chennai", "35");
    cityTemperatureMap.put("Bangalore", "22" );

    Iterator<String> iterator = cityTemperatureMap.keySet().iterator();
    while (iterator.hasNext()){
        System.out.println(cityTemperatureMap.get(iterator.next()));
        // adding new value, it won't throw error
        cityTemperatureMap.put("Kolkata", "34");
    }
}
//运行结果如下
24

java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextNode(HashMap.java:1445)

```

可以看出，在运行在第一次输出之后，改变了HashMap，然后第二次输出之前就失败了

接下来，我们看一下安全失败，其实就是说只有在失败操作是安全的情况下才会失败，否则不会抛出异常，因为ConcurrentHashMap是在多线程环境下运行的，所以一个线程抛出异常，会对其他线程有影响，所以这里不会抛出异常，也就是不会失败

```java
@Test
public void safeItrator(){
    // Creating ConcurrentHashMap
    Map<String, String> cityTemperatureMap = new ConcurrentHashMap<String, String>();
    // Storing elements
    cityTemperatureMap.put("Delhi", "24");
    cityTemperatureMap.put("Mumbai", "32");
    cityTemperatureMap.put("Chennai", "35");
    cityTemperatureMap.put("Bangalore", "22" );

    Iterator<String> iterator = cityTemperatureMap.keySet().iterator();
    while (iterator.hasNext()){
        System.out.println(cityTemperatureMap.get(iterator.next()));
        // adding new value, it won't throw error
        cityTemperatureMap.put("Kolkata", "34");
    }
}
//运行结果如下
24
35
34
32
22
```

更多关于快速失败和安全失败可以看[Fail-Fast Vs Fail-Safe Iterator in Java]()



## 总结

到现在，要是再说HashTable 是因为synchronized 这个比较重的锁导致性能不好的话，就说不过去了，因为ConcurrentHashMap 也使用的是synchronized，所以现在二者最主要的区别在两点

ConcurrentHashMap和HashMap 一样也是基于Hash 的键值对类型数据结构

ConcurrentHashMap的高性能主要是基于桶的加锁方式和无锁的获取方式

ConcurrentHashMap 不允许null 作为key 和value









> separate locks for separate buckets. So the default concurrency level is 16
>
> the first node in the bucket is locked by using synchronized keyword



