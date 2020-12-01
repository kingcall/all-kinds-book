[TOC]

## 概论

HashTable是遗留类，很多映射的常用功能与HashMap类似，不同的是它承自Dictionary类，并且是线程安全的，任一时间只有一个线程能写Hashtable，并发性不如ConcurrentHashMap，因为ConcurrentHashMap引入了分段锁。

Hashtable**不建议在新代码中使用**，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。 



### 对比HashMap 的初始容量

#### 默认11 的初始容量

需要注意的是Hashtable的默认初始容量大小是11，而HashMap 是16,但是他们的加载因子都是0.75f

```
    /**
     * Constructs a new, empty hashtable with a default initial capacity (11)
     * and load factor (0.75).
     */
    public Hashtable() {
        this(11, 0.75f);
    }
```

```
/**
 * Constructs an empty <tt>HashMap</tt> with the default initial capacity
 * (16) and the default load factor (0.75).
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

#### 任意指定非负的容量

还有一点就是Hashtable的initialCapacity 也就是初始容量是是可以是你指定的任何**非负整数**，也就是你给它设置个0 也可以的

```
public Hashtable(int initialCapacity) {
    this(initialCapacity, 0.75f);
}

public Hashtable(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal Load: "+loadFactor);
        
    if (initialCapacity==0)
        initialCapacity = 1;
    this.loadFactor = loadFactor;
    table = new Entry<?,?>[initialCapacity];
    threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
}
```

但是你看一下HashMap 的初始容量就不那么听话了，默认情况下，当我们设置HashMap的初始化容量时，实际上HashMap会采用第一个大于该数值的2的幂作为初始化容量(0 1 除外)

```
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```



### 对比HashMap 的 对null 值的支持

#### HashTable key value 都不支持null 

首先HashMap 是支持null 值做key和value 的，但是HashTable 是不支持的，key 也不支持 value 也不支持

```
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }
    addEntry(hash, key, value, index);
    return null;
}
```

聪明的你们发现了吗，上面值检测了value ==null 则抛出NPE 但是没有说key 啊，因为如果key 是null 的话，key.hashCode()则会抛出异常，根本不需要判断，但是value 就不会抛出来

但是需要注意的实HashMap 对null 值虽然支持，但是可以从hash值的计算方法中看出，<null,value>的键值对，value 会覆盖的。

```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```



#### 升级HashTable 使其支持null 做value

大部分代码都是直接copy 的HashTable，只去掉了value 的空值检测

```
public class BuerHashTable<K, V> extends Hashtable<K, V> {
		// ..... 省略了部分代码，直接copy HashTable 的即可，主要是BuerHashTable.Entry 的定义和构造方法
    public synchronized V put(K key, V value) {

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
    private void addEntry(int hash, K key, V value, int index) {
        modCount++;

        BuerHashTable.Entry<?,?> tab[] = table;
        if (count >= threshold) {
            // Rehash the table if the threshold is exceeded
            rehash();

            tab = table;
            hash = key.hashCode();
            index = (hash & 0x7FFFFFFF) % tab.length;
        }

        // Creates the new entry.
        @SuppressWarnings("unchecked")
        BuerHashTable.Entry<K,V> e = (BuerHashTable.Entry<K,V>) tab[index];
        tab[index] = new BuerHashTable.Entry<>(hash, key, value, e);
        count++;
    }
}
```

接下来，就可以将null 值作为value 存入BuerHashTable 了

```
BuerHashTable<String, String> buerHashTable = new BuerHashTable<>();
buerHashTable.put("a", null);
```



###  对比 HashTable 的继承关系



![image-20201129184257104](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/29/18:42:58-image-20201129184257104.png)



#### Dictionary

这个类是HashTable特有继承的，HashMap 是没有继承的，但是这个抽象类其实是没有多大意义的,因为它的方法都在Map接口中有，其实这个就是个历史问题了，因为Map接口是在Java1.2 中才加进去的，而Dictionary抽象类在Java1.0中就存在了

```
public abstract
class Dictionary<K,V> {
    public Dictionary() {
    }
    abstract public int size();
    abstract public boolean isEmpty();
    abstract public Enumeration<K> keys();
    abstract public Enumeration<V> elements();
    abstract public V get(Object key);
    /**
     * @exception  NullPointerException  if the <code>key</code> or
     */
    abstract public V put(K key, V value);
    abstract public V remove(Object key);
}
```

这个地方的NullPointerException 对应的就是HashTable 中put 方法中的null 值检测

最后一点就是Dictionary 抽象类上的注释，新的实现应该实现Map 接口而不是该抽象类

```
NOTE: This class is obsolete.  New implementations should implement the Map interface, rather than extending this class
```

其实HashMap更准确地说是继承自AbstractMap类，而不是直接实现了Map 接口，所以要是Dictionary这个抽象类要是实现的实Map 接口，那HashMap和Hashtable 就在继承关系上保持一致了

## Hashtable

### 线程安全

其实HashTable 没有那么多要说的，比较重要的一点就是线程安全，但是这个线程安全的实现方式就是所有的操作都加了synchronized关键字，哈哈！ 关于synchronized 我们后面会说

```
public synchronized int size() {}
public synchronized boolean isEmpty() {}
public synchronized boolean contains(Object value) {}
public synchronized boolean containsKey(Object key) {}
public synchronized V get(Object key) {}
public synchronized V put(K key, V value) {}
public synchronized V remove(Object key) {}
```

而HashMap 是线程不安全的

### contains方法

HashMap中没有Hashtable中的contains方法，只有containsValue和containsKey，因为contains方法容易让人引起误解。

Hashtable则保留了contains，containsValue和containsKey三个方法，其中contains和containsValue功能相同。

### debug 源码 put 方法

```
public synchronized V put(K key, V value) {
    // Make sure the value is not null 确保value 不是null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    // 这里的英文注释很有意思啊，就是告诉你确保key 不存在，存在咋地，覆盖又咋地
    Entry<?,?> tab[] = table;
    // 哈希值的计算不同，HashTable直接使用对象的hashCode。而HashMap重新计算hash值(高16位异或低16位)
    int hash = key.hashCode();
    // 计算下标 HashMap 是计算key的hash再与tab.length-1进行与运算；
    // HashTable则是key的hash值与0x7FFFFFFF进行与运算，然后再对tab.length取模
    // 先hash&0x7FFFFFFF后，再对length取模，与0x7FFFFFFF的目的是为了将负的hash值转化为正值，因为hash值有可能为负数，而&0x7FFFFFFF后，只有符号外改变，而后面的位都不变
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    // 确定 index 位置上的链表头，这里主要是遍历链表找到key 值相等的节点，然后返回old value，这样的话就不用添加新值
    // 也就是不用调用addEntry 方法
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    // 存在key 
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }
    // 链表中不存在，则添加新值
    addEntry(hash, key, value, index);
    // 返回null 
    return null;
}

```

```
private void addEntry(int hash, K key, V value, int index) {
    modCount++;
    Entry<?,?> tab[] = table;
    // 判断是否要扩容
    if (count >= threshold) {
        // Rehash the table if the threshold is exceeded
        rehash();
        tab = table;
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;
    }
    // Creates the new entry.
    @SuppressWarnings("unchecked")
    Entry<K,V> e = (Entry<K,V>) tab[index];
    // e 也就是  tab[index] 是这个链表的头结点， tab[index] = new Entry<>(hash, key, value, e); 也就是将元素添加到链表的头部，e 做为new Entry<>(hash, key, value, e)的next 节点
    tab[index] = new Entry<>(hash, key, value, e);
    count++;
}
```

这里我们对比一下HashMap 的添加方法,很明显别人都是添加的链表尾部的，因为HashTable 是线程安全的，在这个前提下，使用头查法性能更好，否则还有遍历到链表的尾部插入

```
for (int binCount = 0; ; ++binCount) {
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            treeifyBin(tab, hash);
        break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        break;
    p = e;
}
```

最后我们再看一下扩容的方法

```
@SuppressWarnings("unchecked")
protected void rehash() {
    int oldCapacity = table.length;
    Entry<?,?>[] oldMap = table;

    // overflow-conscious code 
    // 扩容成2倍+1
    int newCapacity = (oldCapacity << 1) + 1;
    // 这里判断是否超出了容量限制
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        if (oldCapacity == MAX_ARRAY_SIZE)
            // Keep running with MAX_ARRAY_SIZE buckets
            return;
        // 最大容量 MAX_ARRAY_SIZE    
        newCapacity = MAX_ARRAY_SIZE;
    }
    // 创建新的数组
    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];
    modCount++;
    // 更新 threshold
    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    table = newMap;
    // 数据迁移，遍历数组
    for (int i = oldCapacity ; i-- > 0 ;) {
    		// for 循环的方式遍历链表
        for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            Entry<K,V> e = old;
            old = old.next;
            int index = (e.hash & 0x7FFFFFFF) % newCapacity;
            e.next = (Entry<K,V>)newMap[index];
            newMap[index] = e;
        }
    }
}
```



## 总结

1. 需要注意的是Hashtable的默认初始容量大小是11，而HashMap 是16,但是他们的加载因子都是0.75f
2. HashTable的初始容量可以使任何非负整数，但是HashMap会采用第一个大于该数值的2的幂作为初始化容量(0 1 除外，都是 1)
3. HashTable的线程安全是完全借助synchronized 的加持
4. HashTable 的元素是头插法，也就是插入到链表的头部，因为HashTable 是线程安全的，在这个前提下，使用头查法性能更好，否则还有遍历到链表的尾部插入
5. HashTable 是没有红黑树支持的，就是不论链表的长度有多长，都不会转化成红黑树
6.  哈希值的计算不同，HashTable直接使用对象的hashCode。而HashMap重新计算hash值(高16位异或低16位)，并且HashMap 支持key 为null 就是在这里的
7.  Hashtable扩容时，将容量变为原来的2倍加1，而HashMap扩容时，将容量变为原来的2倍。

###  你觉得LinkedHashMap 还有什么可以改进的地方吗，欢迎讨论

和上一节一样这里我依然给出这个思考题，虽然我们的说法可能不对，可能我们永远也站不到源代码作者当年的高度，但是我们依然积极思考，大胆讨论

虽然java 源代码的山很高，如果你想跨越，至少你得有登山的勇气，这里我给出自己的一点点愚见，希望各位不吝指教

```
int hash = key.hashCode();
addEntry(hash, key, value, index);
private void addEntry(int hash, K key, V value, int index) {
		// 记录修改，快速失败
    modCount++;
    Entry<?,?> tab[] = table;
    // count 实际存储的key-value 数目，在HashMap 中用size 表示
    if (count >= threshold) {
        // Rehash the table if the threshold is exceeded
        rehash();
        tab = table;
        // 咋地，数组扩容之后key 的hash值会变吗，你还有重新计算一下
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;
    }
    // Creates the new entry.
    @SuppressWarnings("unchecked")
    Entry<K,V> e = (Entry<K,V>) tab[index];
    tab[index] = new Entry<>(hash, key, value, e);
    count++;
}
```

当然这只是小问题，它最大的问题在设计上，例如hash值的计算方式就没有HashMap 设计的好，还有就是没有红黑树的支持，还有就是线程安全的实现方式也不高效，所以我们说它好像是遗留类,HashTable 在Java1.0 时代就存在了，而HashMap才是Java1.2才有的

![image-20201125223544817](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/29/22:43:11-22:03:49-22:02:40-22:35:45-image-20201125223544817.png)

扫码关注我们一起学习，一起成长，更多请访问[github](https://github.com/kingcall/all-kinds-book)或者[官网](http://troubleclear.com/)