[toc]
## 概论

- HashMap 是无论在工作还是面试中都非常常见常考的数据结构。比如 Leetcode 第一题 Two Sum 的某种变种的最优解就是需要用到 HashMap 的，高频考题 LRU Cache 是需要用到 LinkedHashMap 的。HashMap 用起来很简单，所以今天我们来从源码的角度梳理一下Hashmap
- 随着JDK（Java Developmet Kit）版本的更新，JDK1.8对HashMap底层的实现进行了优化，例如引入红黑树的数据结构和扩容的优化等。
- HashMap：它根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。
-  HashMap最多只允许一条记录的键为null，允许多条记录的值为null。
- HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用 **Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap**。

### Hasmap 的继承关系

![image-20201126201445602](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/20:14:46-image-20201126201445602.png)



### hashmap 的原理

1. 对于 HashMap 中的每个 key，首先通过 hash function 计算出一个 hash 值,这个hash值经过取模运算就代表了在 buckets 里的编号 buckets 实际上是用数组来实现的，所以把这个hash值模上数组的长度得到它在数组的 index，就这样把它放在了数组里。
2. 如果果不同的元素算出了相同的哈希值，那么这就是**哈希碰撞**，即多个 key 对应了同一个桶。这个时候就是解决hash冲突的时候了，展示真正技术的时候到了。



#### 解决Hash冲突的方法



##### 开放定址法

这种方法也称再散列法，其基本思想是：当关键字key的哈希地址p=H(key）出现冲突时，以p为基础，产生另一个哈希地址p1，如果p1仍然冲突，再以p为基础，产生另一个哈希地址p2，…，直到找出一个不冲突的哈希地址pi ，将相应元素存入其中。这种方法有一个通用的再散列函数形式：

Hi=（H（key）+di）% m i=1，2，…，n

其中H(key）为哈希函数，m 为表长，di称为增量序列。增量序列的取值方式不同，相应的再散列方式也不同。主要有三种 线性探测再散列，二次探测再散列，伪随机探测再散列

##### 再哈希法

这种方法是同时构造多个不同的哈希函数

Hi=RH1（key） i=1，2，…，k

当哈希地址Hi=RH1（key）发生冲突时，再计算Hi=RH2（key）……，直到冲突不再产生。这种方法不易产生聚集，但增加了计算时间

##### 链地址法

这种方法的基本思想是将所有哈希地址为i的元素构成一个称为同义词链的单链表，并将单链表的头指针存在哈希表的第i个单元中，因而查找、插入和删除主要在同义词链中进行。

链地址法适用于经常进行插入和删除的情况。



##### 建立公共溢出区

这种方法的基本思想是：将哈希表分为基本表和溢出表两部分，凡是和基本表发生冲突的元素，一律填入溢出表。



#### hashmap 最终的形态

一顿操作猛如虎，搞得原本还是很单纯的hashmap 变得这么复杂，难倒了无数英雄好汉，由于链表长度过程，会导致查询变慢，所以链表慢慢最后演化出了红黑树的形态

`HashMap`主体上就是一个数组结构，每一个索引位置英文叫做一个 bin，我们这里先管它叫做桶，比如你定义一个长度为 8 的 `HashMap`，那就可以说这是一个由 8 个桶组成的数组。

当我们像数组中插入数据的时候，大多数时候存的都是一个一个 Node 类型的元素，Node 是 `HashMap`中定义的静态内部类



![image-20201126201530233](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/20:15:30-image-20201126201530233.png)



### Hashmap 的返回值

很多人以为Hashmap 是没有返回值的，或者也没有关注过Hashmap 的返回值，其实在你调用Hashmap的put(key,value) 方法 的时候，它会将当前key 已经有的值返回，然后把你的新值放到对应key 的位置上

```
public class JavaHashMap {
    public static void main(String[] args) {
        HashMap<String, String> map = new HashMap<String, String>();
        String oldValue = map.put("java大数据", "数据仓库");
        System.out.println(oldValue);
        oldValue = map.put("java大数据", "实时数仓");
        System.out.println(oldValue);
    }
}
```

运行结果如下，因为一开始是没有值的，所以返回null,后面有值了，put 的时候就返回了旧的值

“![image-20201126202457415](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/20:25:06-image-20201126202457415.png)

这里有一个问题需要注意一下，因为Map的Key,Value 的类型都是引用类型，所以在没有值的情况下一定返回的是null，而不是0 等初始值。



## HashMap 的关键内部元素

![](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/20:16:11-image-20201126201610892.png)

### 存储容器 table;

因为`HashMap`内部是用一个数组来保存内容的， 它的定义 如下

`transient Node<K,V>[]  table`

如果哈希桶数组很大，即使较差的Hash算法也会比较分散，如果哈希桶数组数组很小，即使好的Hash算法也会出现较多碰撞，所以就需要在空间成本和时间成本之间权衡，其实就是在根据实际情况确定哈希桶数组的大小，并在此基础上设计好的hash算法减少Hash碰撞。那么通过什么方式来控制map使得Hash碰撞的概率又小，哈希桶数组（Node[] table）占用空间又少呢？答案就是好的Hash算法和扩容机制。



在HashMap中，哈希桶数组table的长度length大小必须为2的n次方(一定是合数)，这是一种非常规的设计，常规的设计是把桶的大小设计为素数。相对来说素数导致冲突的概率要小于合数，具体证明可以参考http://blog.csdn.net/liuqiyao_01/article/details/14475159，Hashtable初始化桶大小为11，就是桶大小设计为素数的应用（Hashtable扩容后不能保证还是素数）。HashMap采用这种非常规设计，主要是为了在取模和扩容时做优化，同时为了减少冲突，HashMap定位哈希桶索引位置时，也加入了高位参与运算的过程。

### size 元素个数

size这个字段其实很好理解，就是HashMap中实际存在的键值对数量。注意和table的长度length、容纳最大键值对数量threshold的区别

### Node

```
 static class Node<K,V> implements Map.Entry<K,V> {
     final int hash;
     final K key;
     V value;
     Node<K,V> next;

     Node(int hash, K key, V value, Node<K,V> next) {
         this.hash = hash;
         this.key = key;
         this.value = value;
         this.next = next;
     }
}
```
- Node是HashMap的一个内部类，实现了Map.Entry接口，本质是就是一个映射(键值对),主要包括 hash、key、value 和 next 的属性。比如之后我们使用 put 方法像其中加键值对的时候，就会转换成 Node 类型。

### TreeNode

```
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
}
    
```



说起TreeNode ，就不得不说其他三个相关参数 TREEIFY_THRESHOLD=8 和 UNTREEIFY_THRESHOLD=6 以及 MIN_TREEIFY_CAPACITY=64

TREEIFY_THRESHOLD=8 指的是链表的长度大于8 的时候进行树化， UNTREEIFY_THRESHOLD=6  说的是当元素被删除链表的长度小于6 的时候进行退化，由红黑树退化成链表

MIN_TREEIFY_CAPACITY=64 意思是数组中元素的个数必须大于等于64之后才能进行树化

### modCount

modCount字段主要用来记录HashMap内部结构发生变化的次数，**主要用于迭代的快速失败**。强调一点，内部结构发生变化指的是结构发生变化，例如put新键值对，但是某个key对应的value值被覆盖不属于结构变化。



## debug 源码

```
public class JavaHashMap {
    public static void main(String[] args) {
        HashMap<String, String> map = new HashMap<String, String>();
        String oldValue = map.put("java大数据", "数据仓库");
    }
}
```
### 调用put()方法

这个方法没什么好说的，是hashmap 提供给用户调用的方法，很简单

### 调用 putval()

-  Put 方法实际上调用的实  putval() 方法

![image-20201126204454960](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/20:44:56-image-20201126204454960.png)

可以看出在进入putval() 方法之间，需要借助hash 方法先计算出key 的hash 值，然后将key 的hash值和key同时传入 

#### 调用hash() 方法

![image-20201126204634472](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/20:46:38-20:46:35-image-20201126204634472.png)
- 这个key的hashCode()方法得到其hashCode 值（该方法适用于每个Java对象），然后再通过Hash算法的后两步运算（高位运算和取模运算，下文有介绍）来定位该键值对的存储位置，有时两个key会定位到相同的位置，表示发生了Hash碰撞。当然Hash算法计算结果越分散均匀，Hash碰撞的概率就越小，map的存取效率就会越高。
- 在JDK1.8的实现中，优化了高位运算的算法，通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

### 进入 putval()

进入putval 方法之后，整体数据流程如下，下面会详细介绍每一步

![image-20201126204925231](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/20:49:26-image-20201126204925231.png)


```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 判断是否需要初始化数组
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
    		// 当前位置为空，则直接插入，同时意味着不走else 最后直接返回null
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
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
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    // 可以看出只有当前key 的位置为空的时候才判断时候需要reszie 已经返回 null 其他情况下都走了else 的环节
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```



#### 判断数组是否为空,需不需要调用resize 方法

第一次调用，这里table 是null,所以会走resize 方法

![image-20201126205708504](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/20:57:09-image-20201126205708504.png)

resize 方法本身也是比较复杂的，因为这里是第一次调用，所以这里进行了简化

```
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               
        		//  首次初始化 zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
           // 因为 oldTab 为null 所以不会进来这个if 判断，所以将这里的代码省略了
        }
        return newTab;
    }

```



##### table 为空首次初始化

如果是的话，初始化数组大小和threashold

```
newCap = DEFAULT_INITIAL_CAPACITY;
newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
```
初始化之后，将新创建的数组返回，在返回之前完成了对变量table 的赋值

![image-20201126211551514](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/21:15:52-image-20201126211551514.png)



##### table 不为空 不是首次初始化

如果不是的话就用当前数组的信息初始化新数组的大小

![image-20201126211919741](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/21:19:20-image-20201126211919741.png)“

最后完成table 的初始化，返回table ，这里其实还有数据迁移，但是为了保证文章的结构，所以将resize 方法的详细讲解单独提了出来

``` Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
table = newTab;
```
#### 判断当前位置是否有元素

##### 1 没有 直接放入当前位置

![image-20201126212145456](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/21:21:46-image-20201126212145456.png)

##### 2 有 将当前节点记做p



![](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/08:50:35-22:05:52-21:35:06-image-20201126213504797.png)

当前节点记做p 然后进入else 循环

```
else {
    Node<K,V> e; K k;
    if (p.hash == hash &&
        ((k = p.key) == key || (key != null && key.equals(k))))
        e = p;
    else if (p instanceof TreeNode)
        e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
    else {
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
    }
    if (e != null) { // existing mapping for key
        V oldValue = e.value;
        if (!onlyIfAbsent || oldValue == null)
            e.value = value;
        afterNodeAccess(e);
        return oldValue;
    }
 }
```



######  判断直接覆盖(判断是否是同一个key)

判断新的key 和老的key 是否相同，这里同时要求了hash 值和 实际的值是相等的情况下然后直接完成了e=p 的赋值，其实也就是完成了替换，因为key 是相同的。

如果不是同一个key 的话这里就要将当前元素插入链表或者红黑树了，因为是不同的key 了

###### 判断插入红黑树

如果当前元素是一个 TreeNode 则将当前元素放入红黑树，然后

![image-20201126220247642](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/22:02:48-image-20201126220247642.png)

###### 判断插入链表

- 如果不是同一key并且当前元素类型不是TreeNode 则将当前元素插入链表(因为key对应的位置已经有元素了，其实可以认为是链表的头元素)

- 可以看出采用的是尾插法，循环过程中**当下一个节点是null的时候则进行插入**，插入完毕之后判断是否需要**树化**

  > JDK 1.7 之前使用头插法、JDK 1.8 使用尾插法

- 其实主要是根据（e=p.next)==null 进行判断进入哪一个if ,因为每个 if 都含有break 语句，所以只能进入一个 然后就退出循环了

  

  ![image-20201126220940075](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/22:09:44-22:09:41-image-20201126220940075.png)

  

  ```
     if ((e = p.next) == null) {
         p.next = newNode(hash, key, value, null);
         if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
             treeifyBin(tab, hash);
         break;
     }
  ```

  1、这段代码也是上图中的第一个if这段代码的意思就是在遍历链表的过程中，一直都没有遇到和待插入key 相同的key(第二个if) 然后当前要插入的元素插入到了链表的尾部(当前if 语句)

  2、插入之后判断判断局部变量binCount 时候大于7(TREEIFY_THRESHOLD-1),这里需要注意的是binCount 是从0开始的，所以实际的意思是判断链表的长度**在插入新元素之前**是否大于等于8，如果是的话则进行树化

  3、并且这个时候变量e 的值是null ，因为是插入到链表的尾部的，所以这个时候key 是没有对应的oldValue 的，所以e是null 在最后面的判断返回中，也返回的是null

  4、关于树化，首先这是发生在插入链表的时刻,并且是插入链表尾部的时候，因为判断过程是在第一个if 中，为了保证文章的结构关于树化放在下面讲

  

  ```
  if (e.hash == hash &&
      ((k = e.key) == key || (key != null && key.equals(k))))
      break;
  // 这个赋值很有意思，它完成了你可以使用for 循环完成链表遍历的核心功能    
  p = e;
  ```

  1、这一段代码的意思是在遍历的过程中（e=p.next)!=null 的的时候，也就是在循环链表的过程中，判断是否有和当前key 相等的key，相等的话e 就是要覆盖的元素，如果不相等的话就继续循环，知道找到这样的e 或者是将链表循环结束，然后将元素插入到链表的尾部(第一个if)

  2、因为是当key 存在的时候则跳出循环，所以链表的长度没有发生变化，所以这里没有判断是否需要树化

######  最后 返回oldValue 完成新值替换

```
if (e != null) { // existing mapping for key
    V oldValue = e.value;
    if (!onlyIfAbsent || oldValue == null)
        e.value = value;
    afterNodeAccess(e);
    return oldValue;
}
```

这个时候e 就指向原来p 的位置了，因为e=p， 然后用新的value 覆盖掉了oldValue 完成了插入，最后将 oldValue 返回。

#### 最后 判断是否需要扩容 返回null 值

其实能走到这一步，是那就说明放入元素的时候，key 对应的位置是没有元素的，所以相当于数组中添加了一个新的元素，所以这里有判断是否需要resize 和返回空值。

```
 ++modCount;
 if (++size > threshold)
     resize();
 afterNodeInsertion(evict);
 return null;
```

### 单独讲解resize 方法

首选需要记住resize 方法是会返回扩容后的数组的

#### 第一部分初始化新数组

这一部分不论是不是首次调用resize 方法，都会有的，但是数据迁移部分在首次调用的时候是没有的

```
Node<K,V>[] oldTab = table;
int oldCap = (oldTab == null) ? 0 : oldTab.length;
int oldThr = threshold;
int newCap, newThr = 0;
// 判断是oldCap 是否大于0 因为可能是首次resize，如果不是的话 oldCap
if (oldCap > 0) {
	 // 到达扩容上限
    if (oldCap >= MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return oldTab;
    }
    // 这里是正常的扩容
    else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
             oldCap >= DEFAULT_INITIAL_CAPACITY)
        newThr = oldThr << 1; // double threshold
}
else if (oldThr > 0) // initial capacity was placed in threshold
    newCap = oldThr;
//第一次调用resize 方法，然后使用默认值进行初始化
else {
		// zero initial threshold signifies using defaults
    newCap = DEFAULT_INITIAL_CAPACITY;
    newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
}

if (newThr == 0) {
    float ft = (float)newCap * loadFactor;
    newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
              (int)ft : Integer.MAX_VALUE);
}
// 创建新的数组，下面
threshold = newThr;
@SuppressWarnings({"rawtypes","unchecked"})
Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
table = newTab;
```

1. 如果数组的大小 大于等于MAXIMUM_CAPACITY之后，则 threshold = Integer.MAX_VALUE; 然后不扩容直接返回当前数组，所以可以看出hashmap 的扩容上限就是MAXIMUM_CAPACITY（2<sup>30</sup>）
2. 如果数组的大小 在扩容之后小于MAXIMUM_CAPACITY 并且原始大小大于DEFAULT_INITIAL_CAPACITY（16） 则进行扩容(DEFAULT_INITIAL_CAPACITY 的大小限制是为了防止该方法的调用是在树化方法里调用的，这个时候数组大大小可能小于DEFAULT_INITIAL_CAPACITY)
3. 新的数组创建好之后，就可以根据老的数组是否有值决定是否进行数据迁移

#### 第二部分数据迁移

oldTab 也就是老的数组不为空的时候进行迁移

```
 if (oldTab != null) {
 			// 遍历oldTable，拿到每一个元素准备放入大新的数组中去
      for (int j = 0; j < oldCap; ++j) {
          Node<K,V> e;
          if ((e = oldTab[j]) != null) {
              oldTab[j] = null;
              // 当前元素只是单个元素，不是链表
              if (e.next == null)
                  // 重新计算每个元素在数组中的位置
                  newTab[e.hash & (newCap - 1)] = e;
              // 判断当前元素是否是树   
              else if (e instanceof TreeNode)
                  ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
              // 当前元素是链表，则遍历链表    
              else { // preserve order
                  Node<K,V> loHead = null, loTail = null;
                  Node<K,V> hiHead = null, hiTail = null;
                  Node<K,V> next;
                  do {
                      next = e.next;
                      if ((e.hash & oldCap) == 0) {
                          if (loTail == null)
                              loHead = e;
                          else
                              loTail.next = e;
                          loTail = e;
                      }
                      else {
                          if (hiTail == null)
                              hiHead = e;
                          else
                              hiTail.next = e;
                          hiTail = e;
                      }
                  } while ((e = next) != null);
                  if (loTail != null) {
                      loTail.next = null;
                      newTab[j] = loHead;
                  }
                  if (hiTail != null) {
                      hiTail.next = null;
                      newTab[j + oldCap] = hiHead;
                  }
              }
          }
      }
  }
```

- 判断当前元素的next 是否为空，不是则直接放入，其实就是只有一个元素
- 是的话，判断是不是TreeNode,不是的话则直接遍历链表进行拷贝
- 是的话则调用 TreeNode.split()  方法
- 完成数据的拷贝，返回新的数组

#### 第三部分 返回新的数组

```
 return newTab;
```

只要没有到达扩容上限，这一部分是肯定会走的，至于走不走数据迁移，需要潘丹是不是首次resize()

### 单独讲解树化treeifyBin方法



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

- 首先判断是符满足链表长度大于8(binCount 是否大于等于7) ,需要注意的是插入到链表的尾部导致链表的长度发生了变化的情况下，才判断是否需要树化
- 然后进入treeifyBin 方法中，进入树化方法之后又判断了,Hashmap 的大小是否大于64，如果不是的话，只是调用了resize 方法，让数组扩容，而不是树化

```
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```



## 总结

### resize 方法总结

#### resize(扩容) 的上限

resize 不是无限的，当到达resize 的上限，也就是2<sup>30</sup> 之后，不再扩容

#### resize 方法只有三种情况下调用

	- 第一种 是在**首次插入元素的时候完成数组的初始化**
	- 第二种 是在元素插入**完成后**判断是否需要数组扩容，如果是的话则调用
	- 第三种 是在元素插入链表尾部之后，进入树化方法之后，如果不树化则进行resize 

#### resize 的返回值

- 第一种情况下 返回老的数组也就是没有resize 因为已经达到resize 的上限了
- 第二种情况下 返回一个空的数组 也就是第一次调用resize方法
- 第三章情况下 返回一个扩容后的数组 完成了数据迁移后的数组 

### key 的判断

- 第一次判断是当前位置有元素的时候，如果两个key 相等则准备覆盖值
- 第二次判断是遍历链表的时候，决定能否覆盖链表中间key 相等的值而不是链表的尾部

### 树化

- 树化是发生在元素插入链表之后，并且这里是插入到链表的尾部导致链表的长度发生了变化的情况下(也就是走的for循环里的第一个if 语句)，而不是替换了链表里面的某一元素(也就是走的for循环里的第二个if 语句)

  ![image-20201127114314435](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/11:43:19-image-20201127114314435.png)

  

### for 循环遍历链表而不是while

这是源代码里面的一段，上面也解释过了，这里使用for 循环遍历链表，利用for 循环的index 进行计数，这里进行了删减

```
for (int binCount = 0; ; ++binCount) {
    if ((e = p.next) == null) {
    		doSomething();
        break;
    
    p = e;
}
```



### 你觉得Hashmap 还有什么可以改进的地方吗，欢迎讨论

虽然java 源代码的山很高，如果你想跨越，至少你得有登山的勇气，这里我给出自己的一点点愚见，希望各位不吝指教



## 番外篇

这里如果你不感兴趣可以不阅读:kissing_smiling_eyes:

###  hash 方法的实现方式

```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

JDK 1.8 中，是通过 hashCode() 的高 16 位异或低 16 位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度，功效和质量来考虑的，减少系统的开销，也不会造成因为高位没有参与下标的计算，从而引起的碰撞

**为什么要用异或运算符？** 保证了对象的 hashCode 的 32 位值只要有一位发生改变，整个 hash() 返回值就会改变。尽可能的减少碰撞。



### 链表法导致的链表过深问题为什么不用二叉查找树代替

之所以选择红黑树是为了解决二叉查找树的缺陷，二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢。

而红黑树在插入新数据后可能需要通过左旋，右旋、变色这些操作来保持平衡，引入红黑树就是为了查找数据快，解决链表查询深度的问题，我们知道红黑树属于平衡二叉树，但是为了保持“平衡”是需要付出代价的，但是该代价所损耗的资源要比遍历线性链表要少，所以当长度大于8的时候，会使用红黑树，**如果链表长度很短的话，根本不需要引入红黑树，引入反而会慢**。



### jdk8中对HashMap做了哪些改变

在java 1.8中，如果链表的长度超过了8，那么链表将转换为红黑树。（桶的数量必须大于64，小于64的时候只会扩容）

发生hash碰撞时，java 1.7 会在链表的头部插入，而java 1.8会在链表的尾部插入

在java 1.8中，Entry被Node替代(换了一个马甲)