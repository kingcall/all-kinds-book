[TOC]

## EnumMap

前面我们学习了枚举，主要讲解了枚举的实现原理和主要使用场景，这一节我们学习一个和枚举相关的集合——EnumMap,`EnumMap` 是Java 集合框架提供的一个每个元素都是枚举类型的Map 集合，在`EnumMap`中枚举元素被用作key。

### 一.  EnumMap 的说明书

还是按照国际惯例，先看一下EnumMap 的`说明书`，其实往往很多时候你的困惑都在`说明书`里写着呢，但是在此之前我们还是先看一下它的继承关系，让我们有一个大概的认识

![image-20210117092256850](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210117092256850.png)

首先我们看到它是数据集合体系下的Map 家族，然后看到它除了继承了AbstractMap 抽象类，还实现了`Serializable` 和`Cloneable` 已经我们今天的重点`Enum` 接口

下面我们就看一下类的注释，来大致了解一下EnumMap。

```java
/**
 * A specialized {@link Map} implementation for use with enum type keys.  All
 * of the keys in an enum map must come from a single enum type that is
 * specified, explicitly or implicitly, when the map is created.  Enum maps
 * are represented internally as arrays.  This representation is extremely
 * compact and efficient.
 * 它是使用enum类型作为key的一种特殊的Map 实现类，不论是显式还是隐式的，所有的key都必须是来自同一个枚举类的枚举变量这就是特殊的所在之处
 * Enum maps 底层是通过数组实现的，这个底层实现方式是极其经凑和高效的
 * <p>Enum maps are maintained in the <i>natural order</i> of their keys
 * (the order in which the enum constants are declared).  This is reflected
 * in the iterators returned by the collections views ({@link #keySet()},
 * {@link #entrySet()}, and {@link #values()}).
 * Enum maps 维持了自然key 的自然顺序，这个顺序就是枚举常量的声明顺序，这个特性也反应在了keySet，entrySet，values 方法返回的collection 集合中
 * <p>Iterators returned by the collection views are <i>weakly consistent</i>:
 * they will never throw {@link ConcurrentModificationException} and they may
 * or may not show the effects of any modifications to the map that occur while
 * the iteration is in progress.
 * 需要注意的是Enum maps的迭代器返回的collection 视图是弱一致性的，也就是说在遍历的过程中如果发生了修改是不会抛出异常的，但是对于其他Map 的实现就会的
 * <p>Null keys are not permitted.  Attempts to insert a null key will
 * throw {@link NullPointerException}.  Attempts to test for the
 * presence of a null key or to remove one will, however, function properly.
 * Null values are permitted.
 *  Null keys 是不允许的。尝试插入或者检测是否存在甚至是删除都会抛出NullPointerException,但是这些在功能上是合理的。Null values 是允许的
 * <P>Like most collection implementations <tt>EnumMap</tt> is not
 * synchronized. If multiple threads access an enum map concurrently, and at
 * least one of the threads modifies the map, it should be synchronized
 * externally.  This is typically accomplished by synchronizing on some
 * object that naturally encapsulates the enum map.  If no such object exists,
 * the map should be "wrapped" using the {@link Collections#synchronizedMap}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access:
 * 和其他大多数集合一样EnumMap是线程不安全的. 所以如果有多个线程在访问并且不止一个线程在修改EnumMap，EnumMap应该在这些操作的外面通过在一个object 上面加锁来保证线程安全
 * 如果没有这样的对象，你可以通过使用 Collections#synchronizedMap 方法，返回其对应的线程安全的封装类
 * <p>Implementation note: All basic operations execute in constant time.
 * They are likely (though not guaranteed) to be faster than their
 * {@link HashMap} counterparts.
 * 它比它的兄弟HashMap快一些，
 * @author Josh Bloch
 * @see EnumSet
 * @since 1.5
 */
public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V>
    implements java.io.Serializable, Cloneable
{
    //Class对象引用
    private final Class<K> keyType;

    //存储Key值的数组
    private transient K[] keyUniverse;

    //存储Value值的数组
    private transient Object[] vals;

    //map的size
    private transient int size = 0;

    //空map
    private static final Enum<?>[] ZERO_LENGTH_ENUM_ARRAY = new Enum<?>[0];

    //构造函数
    public EnumMap(Class<K> keyType) {
        this.keyType = keyType;
        keyUniverse = getKeyUniverse(keyType);
        vals = new Object[keyUniverse.length];
    }

}
```



在学习`EnumMap` 之前，建议先学习[枚举初识](https://blog.csdn.net/king14bhhb/article/details/111224216) 和 [枚举进阶](https://blog.csdn.net/king14bhhb/article/details/111249702)

------

### 二.  EnumMap的使用

#### 创建一个 EnumMap

为了创建一个EnumMap，我们首先需要引入`java.util.EnumMap`类，一旦我们引入了就可以按照下面的方式创建EnumMap

```java
enum Size {
    SMALL, MEDIUM, LARGE, EXTRALARGE
}

EnumMap<Size, Integer> sizes = new EnumMap<>(Size.class);
```

上面我们创建了一个EnumMap，Size类型作为EnumMap的key 的类型，Integer 作为EnumMap的Value 的类型



------

####  EnumMap 插入

- `put()` - 插入指定的key-value 对到EnumMap
- `putAll()` - 插入一个指定的map 的 key-value 对 EnumMap

For example,

```java
@Test
public void insert(){
    EnumMap<Size, Integer> sizes = new EnumMap<>(Size.class);
    sizes.put(Size.SMALL, 28);
    sizes.put(Size.MEDIUM, 32);
    System.out.println("EnumMap1: " + sizes);

    EnumMap<Size, Integer> sizes2 = new EnumMap<>(Size.class);

    // Using the putAll() Method
    sizes2.putAll(sizes);
    sizes2.put(Size.LARGE, 36);
    System.out.println("EnumMap2: " + sizes2);
}
```

**Output**

```
EnumMap1: {SMALL=28, MEDIUM=32}
EnumMap2: {SMALL=28, MEDIUM=32, LARGE=36}
```

------

#### 获取 EnumMap 的元素

**1. 使用 entrySet(), keySet() and values()**

- `entrySet()` - 返回一个set 集合包含EnumMap的元素
- `keySet()` - 返回一个set 集合包含EnumMap 中的keys
- `values()` - 返回一个set 集合包含EnumMap 中的values

```java
@Test
public void access(){
    // Creating an EnumMap of the Size enum
    EnumMap<Size, Integer> sizes = new EnumMap<>(Size.class);
    sizes.put(Size.SMALL, 28);
    sizes.put(Size.MEDIUM, 32);
    sizes.put(Size.LARGE, 36);
    sizes.put(Size.EXTRALARGE, 40);
    System.out.println("EnumMap: " + sizes);

    // Using the entrySet() Method
    System.out.println("Key/Value mappings: " + sizes.entrySet());

    // Using the keySet() Method
    System.out.println("Keys: " + sizes.keySet());

    // Using the values() Method
    System.out.println("Values: " + sizes.values());
}
```

**Output**

```
EnumMap: {SMALL=28, MEDIUM=32, LARGE=36, EXTRALARGE=40}
Key/Value mappings: [SMALL=28, MEDIUM=32, LARGE=36, EXTRALARGE=40]
Keys: [SMALL, MEDIUM, LARGE, EXTRALARGE]
Values: [28, 32, 36, 40]
```

**这里也验证了我们在类注释中说的有序性**

**2. 使用 get() 方法**

 `get()` 方法会返回对应key 的值，如果找不到对应的key,则返回null

```java
@Test
public void access(){
    // Creating an EnumMap of the Size enum
    EnumMap<Size, Integer> sizes = new EnumMap<>(Size.class);
    sizes.put(Size.SMALL, 28);
    sizes.put(Size.MEDIUM, 32);
    sizes.put(Size.LARGE, 36);
    sizes.put(Size.EXTRALARGE, 40);
    System.out.println("EnumMap: " + sizes);
    

    int value = sizes.get(Size.MEDIUM);
    System.out.println("Value of MEDIUM: " + value);
}
```

**Output**

```
EnumMap: {SMALL=28, MEDIUM=32, LARGE=36, EXTRALARGE=40}
Value of MEDIUM: 32
```

------

#### 删除 EnumMap 元素

- `remove(key)` -  删除特定key 对应的key-value,并且返回value 
- `remove(key, value)` - 删除特定的key和value 都相等的key-value，并且返回true

```java
@Test
public void remove(){
    // Creating an EnumMap of the Size enum
    EnumMap<Size, Integer> sizes = new EnumMap<>(Size.class);
    sizes.put(Size.SMALL, 28);
    sizes.put(Size.MEDIUM, 32);
    sizes.put(Size.LARGE, 36);
    sizes.put(Size.EXTRALARGE, 40);
    System.out.println("EnumMap: " + sizes);

    // Using the remove() Method
    int value = sizes.remove(Size.MEDIUM);
    System.out.println("Removed Value: " + value);

    boolean result = sizes.remove(Size.SMALL, 28);
    System.out.println("Is the entry {SMALL=28} removed? " + result);

    System.out.println("Updated EnumMap: " + sizes);
}
```

**Output**

```
EnumMap: {SMALL=28, MEDIUM=32, LARGE=36, EXTRALARGE=40}
Removed Value: 32
Is the entry {SMALL=28} removed? True
Updated EnumMap: {LARGE=36, EXTRALARGE=40}
```

------

#### 替换 EnumMap 的元素

- `replace(key, value)` - 使用新值代替特定key 的旧值
- `replace(key, old, new)` - 使用新值代替特定key 的旧值，只有当提供的旧值和特定key 的旧值相等时猜替换
- `replaceAll(function)` -使用特定函数的返回值替代所有的value 

```java
@Test
public void replace(){
    // Creating an EnumMap of the Size enum
    EnumMap<Size, Integer> sizes = new EnumMap<>(Size.class);
    sizes.put(Size.SMALL, 28);
    sizes.put(Size.MEDIUM, 32);
    sizes.put(Size.LARGE, 36);
    sizes.put(Size.EXTRALARGE, 40);
    System.out.println("EnumMap: " + sizes);

    // Using the replace() Method
    sizes.replace(Size.MEDIUM, 30);
    sizes.replace(Size.LARGE, 36, 34);
    System.out.println("EnumMap using replace(): " + sizes);

    // Using the replaceAll() Method
    sizes.replaceAll((key, oldValue) -> oldValue + 3);
    System.out.println("EnumMap using replaceAll(): " + sizes);
}
```

**Output**

```
EnumMap: {SMALL=28, MEDIUM=32, LARGE=36, EXTRALARGE=40}
EnumMap using replace(): {SMALL=28, MEDIUM=30, LARGE=34, EXTRALARGE=40}
EnumMap using replaceAll(): {SMALL=31, MEDIUM=33, LARGE=37, EXTRALARGE=43}
```

需要注意的是函数的声明，这个函数遍历了整个EnumMap然后替换了所有的值

```
sizes.replaceAll((key, oldValue) -> oldValue + 3);
```

------

#### 其他方法

| Method            | Description                                                  |
| :---------------- | :----------------------------------------------------------- |
| `clone()`         | Creates a copy of the `EnumMap`                              |
| `containsKey()`   | Searches the `EnumMap` for the specified key and returns a boolean result |
| `containsValue()` | Searches the `EnumMap` for the specified value and returns a boolean result |
| `size()`          | Returns the size of the `EnumMap`                            |
| `clear() `        | Removes all the entries from the `EnumMap`                   |

------

### 三. EnumSet 对比 EnumMap

`EnumSet` 和 `EnumMap` 都是存储enum 类型的数据结构，但是它们之间还是有下面的不同

- EnumSet和其他的Set 类型，例如HashSet 不一样，EnumSet不是依赖器对应的HashMap 的而是通过一些列的bits 实现的，EnumMap 和其他Map 一样例如HashMap ，底层依赖的都是数组，具体可以看[深度剖析HashSet](https://blog.csdn.net/king14bhhb/article/details/110679661) 和[深度剖析HashMap](https://blog.csdn.net/king14bhhb/article/details/110294590)
- EnumSet 使用的实静态方法 `allOf()`, `noneOf()`, `of()` 创建的，而EnumMap 是通过构造方法创建的



### 三. EnumMap 的使用场景

其实学到了现在，我们都没有介绍EnumMap的使用场景

我们前面学Enum 的时候介绍了Enum主要用来管理常量的定义，主要用在一些状态变量的定义，而Map 主要用来记录一些key-value 对，例如日活的统计，那么EnumMap主要就是用来存储对一些常量的统计信息或者其他对应的信息。

先思考这样一个问题，现在我们有一堆size大小相同而颜色不同的数据，需要统计出每种颜色的数量是多少以便将数据录入仓库，定义如下枚举用于表示颜色Color:

```java
enum Color {
    GREEN,RED,BLUE,YELLOW
}
```

我们有如下解决方案，使用Map集合来统计，key值作为颜色名称，value代表衣服数量，如下：

```java
public class EnumMapExample {
    enum Color {
        GREEN,RED,BLUE,YELLOW
    }
    @AllArgsConstructor
    @Getter
    private class Clothes {
        String name;
        Color color;
    }

    @Test
    public void test() {
        List<Clothes> list = new ArrayList<>();
        list.add(new Clothes("C001",Color.BLUE));
        list.add(new Clothes("C002",Color.YELLOW));
        list.add(new Clothes("C003",Color.RED));
        list.add(new Clothes("C004",Color.GREEN));
        list.add(new Clothes("C005",Color.BLUE));
        list.add(new Clothes("C006",Color.BLUE));
        list.add(new Clothes("C007",Color.RED));
        list.add(new Clothes("C008",Color.YELLOW));
        list.add(new Clothes("C009",Color.YELLOW));
        list.add(new Clothes("C010",Color.GREEN));
        //方案1:使用HashMap
        Map<String,Integer> map = new HashMap<>();
        for (Clothes clothes:list){
            String colorName=clothes.getColor().name();
            Integer count = map.get(colorName);
            if(count!=null){
                map.put(colorName,count+1);
            }else {
                map.put(colorName,1);
            }
        }

        System.out.println(map.toString());

        System.out.println("---------------");

        //方案2:使用EnumMap
        Map<Color,Integer> enumMap=new EnumMap<>(Color.class);

        for (Clothes clothes:list){
            Color color=clothes.getColor();
            Integer count = enumMap.get(color);
            if(count!=null){
                enumMap.put(color,count+1);
            }else {
                enumMap.put(color,1);
            }
        }

        System.out.println(enumMap.toString());
    }
    
}
```

输出

```
{RED=2, BLUE=3, YELLOW=3, GREEN=2}
---------------
{GREEN=2, RED=2, BLUE=3, YELLOW=3}
```

代码比较简单，我们使用两种解决方案，一种是HashMap，一种EnumMap，虽然都统计出了正确的结果，但是EnumMap作为枚举的专属的集合，我们没有理由再去使用HashMap，毕竟EnumMap要求其Key必须为Enum类型，因而使用Color枚举实例作为key是最恰当不过了，也避免了获取name的步骤，更重要的是EnumMap效率更高，因为其内部是通过数组实现的。

**注** 例子来源于网络

### 四. 源码与扩展

#### put /get 方法的源码

put 方法的返回值还是和HashMap 一样的，都是返回对应的value,但是插入的实现却是简单了很多，

```java
public V put(K key, V value) {
   //检测key的类型
    typeCheck(key);
  	//获取存放value值的数组下标
    int index = key.ordinal();
    //获取旧值
    Object oldValue = vals[index];
  	 //设置新的value值
    vals[index] = maskNull(value);
  	// 如果key 不存在也就是oldValue是null 则标记map大小的变量加1
    if (oldValue == null)
        size++;
    return unmaskNull(oldValue);
}
```



这里通过typeCheck方法进行了key类型检测，判断是否为枚举类型，如果类型不对，会抛出异常

```java
private void typeCheck(K key) {
   Class<?> keyClass = key.getClass();//获取类型信息
   if (keyClass != keyType && keyClass.getSuperclass() != keyType)
       throw new ClassCastException(keyClass + " != " + keyType);
}12345
```

接着通过`int index = key.ordinal()`的方式获取到该枚举实例的顺序值，利用此值作为下标，把值存储在vals数组对应下标的元素中即`vals[index]`，这也是为什么EnumMap能维持与枚举实例相同存储顺序的原因，我们发现在对vals[]中元素进行赋值和返回旧值时分别调用了maskNull方法和unmaskNull方法



get 方法的源码更简单

```
public V get(Object key) {
    return (isValidKey(key) ?
            unmaskNull(vals[((Enum<?>)key).ordinal()]) : null);
}
```



#### EnumMap 的有序性

```java
@Test
public void order(){
    // Creating an EnumMap of the Size enum
    EnumMap<Size, Integer> sizes = new EnumMap<>(Size.class);
    HashMap< Size, Integer> sizes2=new HashMap<>();
  	// 按照枚举变量的声明顺序插入
    sizes.put(Size.SMALL, 28);
    sizes.put(Size.MEDIUM, 32);
    sizes.put(Size.LARGE, 36);
    sizes.put(Size.EXTRALARGE, 40);

    sizes2.put(Size.SMALL, 28);
    sizes2.put(Size.MEDIUM, 32);
    sizes2.put(Size.LARGE, 36);
    sizes2.put(Size.EXTRALARGE, 40);

    System.out.println("EnumMap: " + sizes);
    System.out.println("HashMap: " + sizes2);

    sizes.clear();
    sizes2.clear();
  	// 随机插入
    sizes.put(Size.LARGE, 36);
    sizes.put(Size.EXTRALARGE, 40);
    sizes.put(Size.SMALL, 28);
    sizes.put(Size.MEDIUM, 32);

    sizes2.put(Size.LARGE, 36);
    sizes2.put(Size.EXTRALARGE, 40);
    sizes2.put(Size.SMALL, 28);
    sizes2.put(Size.MEDIUM, 32);


    System.out.println("EnumMap: " + sizes);
    System.out.println("HashMap: " + sizes2);

}
```

输出

```
EnumMap: {SMALL=28, MEDIUM=32, LARGE=36, EXTRALARGE=40}
HashMap: {SMALL=28, EXTRALARGE=40, LARGE=36, MEDIUM=32}
EnumMap: {SMALL=28, MEDIUM=32, LARGE=36, EXTRALARGE=40}
HashMap: {SMALL=28, EXTRALARGE=40, LARGE=36, MEDIUM=32}
```

我们看到不论是按照枚举变量的声明顺序插入，还是随机插入EnumMap 的输出顺序都是一样的，那就是按照枚举变量的声明顺序排序，但是HashMap就是无序的



#### 没有 fail-fast

前面我们学习其他集合的时候，不止一次的提到了这个问题和演示了它，之所以fail-fast是处于一种设计理念，那就是宁愿失败也不要计算结果错误，但是EnumMap却没有保证这一点，接下啦我们演示一下它

```java
@Test
public void noFailFast(){
    // Creating an EnumMap of the Size enum
    EnumMap<Size, Integer> sizes = new EnumMap<>(Size.class);
    HashMap< Integer, Integer> sizes2=new HashMap<>();

    sizes.put(Size.SMALL, 28);
    sizes.put(Size.MEDIUM, 32);
    sizes.put(Size.LARGE, 36);
    sizes.put(Size.EXTRALARGE, 40);

    sizes2.put(Size.SMALL.ordinal(), 28);
    sizes2.put(Size.MEDIUM.ordinal(), 32);
    sizes2.put(Size.LARGE.ordinal(), 36);
    sizes2.put(Size.EXTRALARGE.ordinal(), 40);

    Iterator<Map.Entry<Integer, Integer>> it=sizes2.entrySet().iterator();
    try {
        while (it.hasNext()){
            Map.Entry<Integer,Integer> entry=it.next();
            if (entry.getKey().compareTo(Size.MEDIUM.ordinal())==0){
                sizes2.remove(Size.MEDIUM.ordinal());
            }
            System.out.println(entry);
        }
    }catch (ConcurrentModificationException e){
        e.printStackTrace();
    }
    System.out.println("========================");
    Iterator<Map.Entry<Size, Integer>> it2=sizes.entrySet().iterator();
    while (it2.hasNext()){
        Map.Entry<Size,Integer> entry=it2.next();
        if (entry.getKey().compareTo(Size.MEDIUM)==0){
            sizes.remove(Size.MEDIUM);
        }
        System.out.println(entry);
    }
    System.out.println(sizes.size());
    System.out.println(sizes2.size());
    
}
```



输出

```
0=28
1=32
java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextNode(HashMap.java:1445)
	at java.util.HashMap$EntryIterator.next(HashMap.java:1479)
	at java.util.HashMap$EntryIterator.next(HashMap.java:1477)
========================
SMALL=28
MEDIUM=null
LARGE=36
EXTRALARGE=40
3
3
```

我们看到EnumMap并没失败

##### **Fail-Fast实现原理**

有一个变量modCount来指示集合被修改的次数。在创建Iterator迭代器的时候，会给这个变量赋值给expectedModCount。当集合方法修改集合元素时，例如集合的remove()方法时，此时会修改modCount值，但不会同步修改expectedModCount值。当使用迭代器遍历元素操作时，会首先对比expectedModCount与modCount是否相等。如果不相等，则马上抛出java.util.ConcurrentModificationException异常。而通过Iterator的remove()方法移除元素时，会同时更新expectedModCount的值，将modCount的值重新赋值给expectedModCount，这样下一次遍历时，就不会发抛出java.util.ConcurrentModificationException异常。

##### **Fail-Fast源码实现——HashMap**

首先我们看一下HashMap的实现，下面是迭代器相关的代码

```java
abstract class HashIterator {
    Node<K,V> next;        // next entry to return
    Node<K,V> current;     // current entry
    int expectedModCount;  // for fast-fail
    int index;             // current slot

    HashIterator() {
        //在构造迭代器的时候，将modCount值赋值给expectedModCount
        expectedModCount = modCount;
        Node<K,V>[] t = table;
        current = next = null;
        index = 0;
        if (t != null && size > 0) { // advance to first entry
            do {} while (index < t.length && (next = t[index++]) == null);
        }
    }

    public final boolean hasNext() {
        return next != null;
    }

    final Node<K,V> nextNode() {
        Node<K,V>[] t;
        Node<K,V> e = next;
        //在获取下一个节点前，先判定modCount值是否修改，如果被修改了则抛出ConcurrentModificationException异常，从前面可以知道，当修改了HashMap的时候，都会修改modCount值。
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        if ((next = (current = e).next) == null && (t = table) != null) {
            do {} while (index < t.length && (next = t[index++]) == null);
        }
        return e;
    }

    //迭代器的删除操作，会重新给exceptedModCount赋值，因此不会导致fast-fail
    public final void remove() {
        Node<K,V> p = current;
        if (p == null)
            throw new IllegalStateException();
        //先判定modCount值是否被修改了
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        current = null;
        K key = p.key;
        removeNode(hash(key), key, null, false, false);
        //将modCount值重新赋值给expectedModCount，这样下次迭代时，不会出现fast-fail
        expectedModCount = modCount;
    }
}
```



##### **Fail-Fast源码实现——EnumMap**

我们看到EnumMap的Iterator的实现，并没有相关的检测

```java
private abstract class EnumMapIterator<T> implements Iterator<T> {
    // Lower bound on index of next element to return
    int index = 0;

    // Index of last returned element, or -1 if none
    int lastReturnedIndex = -1;

    public boolean hasNext() {
        while (index < vals.length && vals[index] == null)
            index++;
        return index != vals.length;
    }

    public void remove() {
        checkLastReturnedIndex();

        if (vals[lastReturnedIndex] != null) {
            vals[lastReturnedIndex] = null;
            size--;
        }
        lastReturnedIndex = -1;
    }

    private void checkLastReturnedIndex() {
        if (lastReturnedIndex < 0)
            throw new IllegalStateException();
    }
}
```



#### Null keys 和 Null values

 Null keys 是不允许的。尝试插入或者检测是否存在甚至是删除都会抛出NullPointerException。Null values 是允许的

```java
@Test
public void nullCheck(){
    // Creating an EnumMap of the Size enum
    EnumMap<Size, Integer> sizes = new EnumMap<>(Size.class);
    sizes.put(Size.SMALL, 28);
    sizes.put(Size.MEDIUM, 32);
    sizes.put(Size.LARGE, 36);
    sizes.put(Size.EXTRALARGE, 40);
    try {
        sizes.put(null, 1);
    }catch (NullPointerException e){
        System.out.println("插入null key异常");
    }

    try {
        boolean flag=sizes.containsKey(null);
        System.out.println(flag);
    }catch (NullPointerException e){
        System.out.println("检测null存在异常");
    }

    try {
        int result=sizes.remove(null);
        System.out.println(result);
    }catch (NullPointerException e){
        System.out.println("删除null key异常");
    }

    try {
        int result=sizes.put(Size.EXTRALARGE,null);
        System.out.println(result);
    }catch (NullPointerException e){
        System.out.println("插入 null value 异常");
    }
}
```

输出

```
插入null key异常
false
删除null key异常
40
```



#### EnumMap 为什么更加高效

EnumMap 相比于HashMap 的高效体现在了很多方面，下面仅仅只是列出了主要的点

##### 1. EnumMap枚举值确定的个数决定了其不用扩容和空间浪费

不用扩容就意味着没有数据的迁移和新数组的创建，以及一些列的方法执行检测什么的，而且枚举的数组存储空间是全部使用的，HashMap 是可能有的`空位`的，下面是我们在学习HashMap的时候画的一张图

![image-20201128094513006](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/09:45:14-image-20201128094513006.png)



下面是EnumMap的构造方法，你可以通过构造方法来一下空间和浪费的问题

```java
public EnumMap(Class<K> keyType) {
    this.keyType = keyType;
    keyUniverse = getKeyUniverse(keyType);
    vals = new Object[keyUniverse.length];
}
```

##### 2. EnumMap枚举值的唯一性使其没有链表红黑树等的开销

EnumMap 因为其大小是确定，并且枚举值是唯一的，所以没有hash 冲突，所以没有红黑树、链表等数据结构的操作

##### 3. EnumMap基于枚举的设计使其减少了对key-value 封装的开销

下面是HashMap 中的一段代码，也就是HashMap中数据的实际存储的实现，但是EnumMap直接存储的对应的key 和 value 没有多余的开销

```java
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

##### 4.  EnumMap基于枚举的设计使其操作更加简单

EnumMap的所有操作都是根据Enum的ordinal值直接对数组操作，所以性能比数组实现的HashMap 更好

## 总结

1. 首先我们得清楚EnumMap的使用场景，其次我们得知道EnumMap能实现的，HashMap 都能实现

2. EnumMap的性能更好，这是因为EnumMap的Enum特性决定了不论是从设计上，实现上，存储上都决定了其有很好的性能

3. EnumMap还有其特殊的一些特性，例如维持了自然顺序。

   

