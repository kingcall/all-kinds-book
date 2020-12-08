[TOC]

## 一. ArrayList 初识



ArrayList是集合的一种实现，实现了接口List，List接口继承了Collection接口。

ArrayList  是java 中最常用的集合类型，这是因为它使用起来非常简单，而且它提供了非常丰富的功能，并且性能非常好，这里需要注意的是性能是以牺牲了线程安全为代价的，ArrayList 好用又很大一部分来自它的动态扩容，不像数组那样你需要提前计算好数组的大小，ArrayList 会随着元素的增加自动扩容。

虽然在其内部它并不是真正的弹性数组在不断增长，但它就像拥有一个具有**初始容量**（默认为长度为10的数组）的数组一样简单。当超过这个限制**创建另一个数组，它是原始数组的1.5倍**，旧数组中的元素被复制到新数组中。

ArrayList使用非常广泛，不论是数据库表查询，excel导入解析，还是网站数据爬取都需要使用到，了解ArrayList原理及使用方法显得非常重要。



下面就是ArrayList 的真实形态了



![image-20201206095212963](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/09:52:13-image-20201206095212963.png)



### 1. ArrayList 的说明书

在看说明书之前，我们还是先看一下整个类的继承关系



![image-20201206112834609](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/11:28:35-image-20201206112834609.png)





我们读一下源码，看看定义ArrayList的重要属性，当然我们还是习惯性的读一下类注释，让我们先有一个大概的认识，最后我们再通过例子，对它有一个精准的认识

```java
/**
 * Resizable-array implementation of the <tt>List</tt> interface.  Implements all optional list operations, and permits all elements, including <tt>null</tt>.  
 * 可变化的数组实现了list 接口，实现了list 的所有操作，并且允许插入所有的元素，包括null 
 * In addition to implementing the <tt>List</tt> interface, this class provides methods to manipulate the size of the array that is used internally to store the list. 
 * 除了实现了list 接口之外，还提供了在内部使用的，操作存储了list元素数组的大小的方法
 * (This class is roughly equivalent to <tt>Vector</tt>, except that it is unsynchronized.)
 * 这个类大致上和Vector 类似，除了这个类不是同步的
 * <p>The <tt>size</tt>, <tt>isEmpty</tt>, <tt>get</tt>, <tt>set</tt>, <tt>iterator</tt>, and <tt>listIterator</tt> operations run in constant time. 
 * 上面这些方法的时间复杂度都是常数，其实就是O(1)
 * The <tt>add</tt> operation runs in <i>amortized constant time</i>,that is, adding n elements requires O(n) time. 
 * add 方法 均摊时间复杂度是O(1),添加n 个元素的时间复杂度就是O(n)
 * All of the other operations run in linear time (roughly speaking).  The constant factor is low compared to that for the <tt>LinkedList</tt> implementation.
 * 所有其他的方法的时间复杂度都是线性的，
 * <p>Each <tt>ArrayList</tt> instance has a <i>capacity</i>.  The capacity is the size of the array used to store the elements in the list.  It is always
 * at least as large as the list size. 
 * 每一个ArrayList的实例都有一个容量，这个容量就是list 里面用来存储元素的数组的大小，它是和list 的大小是一样大的
 * As elements are added to an ArrayList,its capacity grows automatically. 
 * 随着元素的添加，ArrayList 的大小在自动变化
 * The details of the growth policy are not specified beyond the fact that adding an element has constant amortized time cost.
 * 除了增加一个元素具有固定的均摊时间复杂度这一事实外，增长策略的细节没有被指定。
 * <p>An application can increase the capacity of an <tt>ArrayList</tt> instance before adding a large number of elements using the <tt>ensureCapacity</tt>
 * operation.  This may reduce the amount of incremental reallocation.
 * 应用程序可以在添加大量元素之前通过该指定ensureCapacity的操作来增加ArrayList的容量（其实就是通过构造方法），这样可以减少重新分配的次数(扩容的次数)
 * <p><strong>Note that this implementation is not synchronized.</strong> 
 * 重点注意这个实现不是线程安全的
 * The list should be "wrapped" using the  {@link Collections#synchronizedList Collections.synchronizedList} method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the list:<pre> List list = Collections.synchronizedList(new ArrayList(...));</pre>
 * 如果需要线程安全的对象，可以使用Collections.synchronizedList对其进行包装，最好在创建时执行此操作，以防止意外的对列表的非同步访问，List list = Collections.synchronizedList(new ArrayList(...))
 * <p><a name="fail-fast"> The iterators returned by this class's {@link #iterator() iterator} and {@link #listIterator(int) listIterator} methods are <em>fail-fast</em>:</a>
 * 这个类的iterator() 和 listIterator() 方法返回的迭代器都是fail-fast 的
 * if the list is structurally modified at any time after the iterator is created, in any way except through the iterator's own {@link ListIterator#remove() remove} or {@link ListIterator#add(Object) add} methods, the   iterator will throw a {@link ConcurrentModificationException}. 
 * 除了ListIterator#remove()和ListIterator#add(Object) 这两个方法之外的任何情况下，这个类对象的iterator在创建之后的任何时间，发生了结构上的修改则会抛出ConcurrentModificationException 的异常
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @since   1.2
 */

public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    /**
     * Default initial capacity.  默认的初始容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     * 指定参数初始容量，但是初始容量是0的时候 
     * elementData= EMPTY_ELEMENTDATA
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     * 无参构造的时候使用 elementData= DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. 
     * 其实就是实际存储元素的数组
     * Any empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     * 如果是 elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA 则在第一次添加元素的时候则会扩容到DEFAULT_CAPACITY
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     * 实际存储的元素多少
     * @serial
     */
    private int size;
}
```

其实源码里面已经很清晰了，底层是一个Object[]，添加到ArrayList中的数据保存在了elementData属性中。

- 例如当调用`new ArrayList<>()`时，将一个空数组 **DEFAULTCAPACITY_EMPTY_ELEMENTDATA**  赋值给了elementData，这个时候集合的长度size为默认长度0；
- 例如当调用`new ArrayList<>(100)`时，根据传入的长度，new一个Object[100]赋值给elementData，当然如果玩儿的话，传了一个0，那么将一个空数组 **EMPTY_ELEMENTDATA** 赋值给了elementData；
- 例如当调用new ArrayList<>(new HashSet())时，根据源码，我们可知，可以传递任何实现了Collection接口的类，将传递的集合调用toArray()方法转为数组内赋值给elementData;



### 2. ArrayList 的构造方法



#### 无参构造

```java
//默认创建一个ArrayList集合
List<String> list = new ArrayList<>();
/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

创建一个空的使用默认容量的list(默认是10)

#### 指定初始容量

```java
//创建一个初始化长度为100的ArrayList集合
List<String> initlist = new ArrayList<>(100);

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}
```

创建一个空的指定容量的list

#### 其他集合作为参数

```java
//将其他类型的集合转为ArrayList
List<String> setList = new ArrayList<>(new HashSet());

/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

构造一个包含指定集合元素的列表，其顺序由集合的迭代器返回。当传入的集合参数为空的话，抛出NullPointerException，因为它会调用该集合的toArray 方法，和HashTable 里面调用key 的hashcode 方法的原理一样

当集合是一个空的集合的话，elementData = EMPTY_ELEMENTDATA和指定0是initialCapacity的效果一样

**注意:**在传入集合的ArrayList的构造方法中，有这样一个判断

> if (elementData.getClass() != Object[].class)，
>
> 给出的注释是：c.toArray might (incorrectly) not return Object[] (see 6260652)，即调用toArray方法返回的不一定是Object[]类型，查看Collection接口的定义 

```
Object[] toArray();
```



> 我们发现返回的确实是Object[]，那么为什么还会有这样的判断呢？
>
> 如果有一个类CustomList继承了ArrayList，然后重写了toArray()方法呢。。

```java
public class CustomList<E> extends ArrayList {
    @Override
    public Integer [] toArray() {
        return new Integer[]{1,2};
    };
    
    public static void main(String[] args) {
        Object[] elementData = new CustomList<Integer>().toArray();
        System.out.println(elementData.getClass());
        System.out.println(Object[].class);
        System.out.println(elementData.getClass() == Object[].class);
    }
}
```

执行结果：

```
class [Ljava.lang.Integer;
class [Ljava.lang.Object;
false
```

接着说，如果传入的集合类型和我们定义用来保存添加到集合中值的Object[]类型不一致时，ArrayList做了什么处理？读源码看到，调用了`Arrays.copyOf(elementData, size, Object[].class);`，继续往下走

```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {    
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength); 
    System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
    return copy;
}
```

我们发现定义了一个新的数组，将原数组的数据拷贝到了新的数组中去。

### 3. ArrayList 内部构成

#### size

实际存储的元素个数，也就是数组中存储的元素个数，而不是数组的大小

#### DEFAULT_CAPACITY

默认的数组大小，就是在你不指定minCapacity变量的时候，它将使用DEFAULT_CAPACITY 作为数组的大小(但是需要注意的是它是在第一次添加元素的时候在使用的)，而不是像你指定了minCapacity在构造方法中那样，在创建ArrayList 的过程中就创建了

#### CAPACITY

首先说明一下，ArrayList 中没有这样的变量，但是我把它单独拿出来说，是想强调一点，CAPACITY 表示的实数组的大小，而不是实际存储的元素个数——size,它表示的实存储能力，size 表示已经使用

#### elementData

ArrayList 背后真正存储数据的结构，也就是存储数据的数组,可以说是ArrayList背后的巨人



#### DEFAULTCAPACITY_EMPTY_ELEMENTDATA

无参构造时候，elementData=DEFAULTCAPACITY_EMPTY_ELEMENTDATA 其实主要是为了区分elementData=它是无参构造的赋值，还是有参但是是0的时候的赋值

#### EMPTY_ELEMENTDATA

无参构造时候但是initialCapacity=0的时候，EMPTY_ELEMENTDATA 其实主要是为了区分elementData=它是无参构造的赋值，还是有参但是是0的时候的赋值

## 二. ArrayList常用方法

ArrayList有很多常用方法，add，addAll，set，get，remove，size，isEmpty等

首先定义了一个ArrayList,

```java
List<String> list = new ArrayList<>(10);
list.add('牛魔王');
list.add('蛟魔王');
...
list.add('美猴王');
```

Object[] elementData中数据如下：

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:06:19-1677914-20190626150620394-1213393047.png)

### 1. add(E element)

我们通过源码来看一下add("白骨精")到底发生了什么

```java
/**
 * Appends the specified element to the end of this list.
 * 添加一个元素到列表的末尾
 * @param e element to be appended to this list 被添加的元素
 * @return <tt>true</tt> (as specified by {@link Collection#add}) 固定的返回值 True
 */
public boolean add(E e) {
    // 保证容量，这个就是在添加元素之前要要保证内部的数组大小足够可以容纳该元素 
    ensureCapacityInternal(size + 1);
    // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

首先通过 `ensureCapacityInternal(size + 1)` 来保证底层Object[]数组有足够的空间存放添加的数据，然后将添加的数据存放到数组对应的位置上，我们看一下是怎么保证数组有足够的空间？

这里如果是首次添加元素的话，szie=0 然后minCapacity=1 的

```java
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

首先调用了下面的方法计算容量，见名知意吧(calculate Capacity)，如果elementData 不是空的话，直接返回minCapacity，也就是szie+1,是空的话就返回Math.max(DEFAULT_CAPACITY, minCapacity) 也就是10

```java
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 这里有个注意点就是，无参数构造的话，elementData =DEFAULTCAPACITY_EMPTY_ELEMENTDATA 也就是说这里会返回 10，但是有参构造的话，不满足这个条件,也就是如果是无参构造的话，在第一次添加元素的时候将直接扩容到DEFAULT_CAPACITY
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    // 否则的话则是有参构造或者不是第一次添加元素，那么这里返回的就是size+1，也就是说扩容到所需的szie+1 即可
    return minCapacity;
}
```

拿到返回的的capacity 之后，进行判断 确保足够的容量（ensure explicit capacity）

```java
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // 不满足要求的容量则进行扩容，也就是数组大小不够了
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

这里首先确定了Object[]足够存放添加数据的最小容量，然后通过 `grow(int minCapacity)` 来进行数组扩容

```java
/**
 * Increases the capacity to ensure that it can hold at least the number of elements specified by the minimum capacity argument.
 * 增加容量以保证至少可以存储输入参数那么多的元素
 * @param minCapacity the desired minimum capacity 确定需要最少容量的参数
 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

扩容规则为 **数组小容量 + （数组当前容量 / 2）**，即**数组当前容量 \* 1.5**，当然有最大值的限制。如果计算出来的结果也就是**数组当前容量 \* 1.5** 还是不足的话，直接使用newCapacity作为最小容量

这里需要注意一个问题，在老版本的代码中，它的计算公式是这样的

```
int newCapacity = (oldCapacity * 3)/2 + 1;
```



因为最开始定义了集合容量为10，故而本次不会进行扩容，直接将第8个位置（从0开始，下标为7）设置为“白骨精”，这时Object[] elementData中数据如下：

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:06:19-1677914-20190626150705934-1128204746.png)

当然add 方法还有很多变体，如果你感兴趣可以自己研究一下，因为add方法指定index的时候和set 方法很类似，所以你可以看完下面的set 方法再去看add 方法的变体

> `add("铁扇", 0);` //将数组中的元素各自往后移动一位，再将“铁扇”放到第一个位置上；
>
> ![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:06:19-1677914-20190626150743137-1264984389.png)
>
> `addAll(list..七个葫芦娃);` //将集合{七个葫芦娃}放到"白骨精"后，很明显当前数组的容量已经不够，需要扩容了，不执行该句代码了；
>
> `addAll(list..哪吒三兄弟, 4);`//从第五个位置将“哪吒三兄弟”插进去，那么数组第五个位置后的元素都需往后移动三位，数组按规则扩容为18。
>
> ![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:06:19-1677914-20190626150816841-278720793.png)
>
> 指定了插入位置的，会通过**rangeCheckForAdd(int index)**方法判断是否数组越界

**总结**

当添加一个新元素到ArrayList的时候，首选会验证一下ArrayList时候有足够的容量了来存储新的元素，如果不够的话则会创建一个新的数组是原来数组大小的1.5 倍，然后copy 数据到新的数组



**这里需要注意一下的是扩容这个过程，或者说是计算容量的过程，如果你是无参构造则一次性扩容到DEFAULT_CAPACITY，如果不是则都是根据所需容量(size+1)进行判断是否要扩容,如果是的话，则扩容为原来的1.5倍**



当然ArrayList 的add 也存在很多变体的方法，例如下面的两个方法，你可以研究一下，当然添加元素到指定位置的方法，和set 有点类似，只不过set 是覆盖，add 可能需要移动元素

```
arrayList.add(1,"a");
arrayList.addAll(new HashSet());
```

这里我还是对添加到指定位置进行简单说明一下，我们还是接着看源码

```java
/**
 * Inserts the specified element at the specified position in this list. Shifts the element currently at that position (if any) and
 * any subsequent elements to the right (adds one to their indices).
 * 插入指定元素到指定位置，如果当前待插入的位置有元素，则需要右移当前元素和其后的元素
 * @param index index at which the specified element is to be inserted
 * @param element element to be inserted
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
  	// 检查位置的合法性
    rangeCheckForAdd(index);
		// 扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
  	// 移动当前位置和其后置的元素
    System.arraycopy(elementData, index, elementData, index + 1,ize - index);
  	// 存储该元素
    elementData[index] = element;
  	// 修改元素数目
    size++;
}
```

这里的检查方法也是比较简单的，就是下标必须是已经存在的元素的位置

```java
/**
 * A version of rangeCheck used by add and addAll.
 */
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```



其实走到这个你就会发现，ArrayLis 是一个**密集的数据结构**，因为它不会差生空位，你可以对比HashMap

### 2. set(int index, E element)

因为ArrayList底层是由数组实现的，set实现非常简单，调用 `set(8, "猪八戒")` 通过传入的数字下标找到对应的位置，**替换其中的元素**，前提也需要首先判断传入的数组下标是否越界。*将“猕猴王”替换为“猪八戒”*。

```java
public E set(int index, E element) {
    rangeCheck(index);
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
// 需要注意的是这里不会给你扩容的，如果你越界直接抛出异常
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

返回值“猕猴王”，当前数组中数据：

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:06:19-1677914-20190626150842478-1367690033.png)

### 3. get(int index)

ArrayList中get方法也非常简单，通过下标查找即可，同时需要进行了类型转换，因为数组为Object[]，前提是需要判断传入的数组下标是否越界。

```java
public E get(int index) {
   // 当然也是检测下标是否越界
    rangeCheck(index);
    return elementData(index);
}
E elementData(int index) {
    return (E) elementData[index];
}
```

调用get(6)返回”哪吒“。

### 4. remove(int index)

首先说一下ArrayList通过下标删除的方法，我们看一下源码

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```

通过源码我们可以看到首先获取了待删除的元素，并最终返回了。这里的下标和数组一样，都是从0开始的

其次计算了数组中需要移动的位数 size - index - 1，那么很明显我们可以得出待删除的是最后一个元素的话，移到位数为0，否则移动位数大于0，那么通过数组元素的拷贝来实现往前移动相应位数。

如remove(10)，找到的元素为“美猴王”，那么移动位数 = 12-10-1 = 1；此时将原本在第12个位置上（数组下标为11）的“白骨精”往前移动一位，同时设置elementData[11] = null；这里通过设置null值让GC起作用。

### 5. remove(Object o)

删除ArrayList中的值对象，其实和通过下标删除很相似，只是多了一个步骤，遍历底层数组elementData，通过equals()方法或 == （特殊情况下）来找到要删除的元素，获取其下标，调用remove(int index)一样的代码即可。

```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```

当然 ArrayList 还给我们提供了另外一个方法，那就是removeIf()，它是一个函数式接口

```java
 List<String> cityList = new ArrayList<>(2);
 cityList.removeIf((String name )->name.equalsIgnoreCase("Bangalore"));
```



### 6. iterator 方法

这里主要演示一下它的快速失败属性

```java
@Test
public void itrator(){
    List<String> cityList = new ArrayList<>(2);
    cityList.add("London");
    cityList.add("Paris");
    cityList.add("Bangalore");
    cityList.add("Istanbul");
    Iterator<String> itr = cityList.iterator();
    while(itr.hasNext()){
        String city = itr.next();
        if(city.equals("Paris")){
            cityList.remove(city);
        }
        System.out.println(city);
    }
}
```

输出结果

```
London
Paris
java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
```

当然如果你就想在遍历的过程中删除元素，也不是不行,调用iterator 的remove 方法即可,然后ArrayList 中的元素也会被删除的

```java
@Test
public void itrator2(){
    List<String> cityList = new ArrayList<>(2);
    cityList.add("London");
    cityList.add("Paris");
    cityList.add("Bangalore");
    cityList.add("Istanbul");
    Iterator<String> itr = cityList.iterator();
    while(itr.hasNext()){
        String city = itr.next();
        if(city.equals("Paris")){
            itr.remove();
        }else {
            System.out.println(city);
        }
    }
}
```



### 7. 其他方法

**size()** : 获取集合长度，通过定义在ArrayList中的私有变量size得到

**isEmpty()**：是否为空，通过定义在ArrayList中的私有变量size得到

**contains(Object o)**：是否包含某个元素，通过遍历底层数组elementData，通过equals或==进行判断

**clear()**：集合清空，通过遍历底层数组elementData，设置为null

```java
/**
 * Removes all of the elements from this list.  The list will
 * be empty after this call returns.
 */
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;
    size = 0;
}
```

这里给大家一个思考题，为什么要遍历呢，而不是先创建一个同等大小的数组，然后将数组当前设置为null呢，其实我觉得也可以,而且更快

## 三.  ArrayList 的性能

**Adding an element**- 如果你使用的实  **add(E e)** 方法添加一个元素到ArrayList末尾 ，它的时间复杂度 **O(1)**；但是当空间不足引发扩容的时候，会导致新建数组然后拷贝数据，这个时候它的时间复杂度 **O(n)** ;当你使用 add(int index, E element)的时候它的算法复杂度是 **O(n - index)** 也就是 **O(n)**

**Retrieving an element**- 当你使用**get(int index)** 的时候，它的时间复杂度是 **O(1)**，因为数组可以直接根据下标进行定位

**Removing an element**- 当你使用 **remove(int index)** 它的时间复杂度是 **O(n - index) **，因为它涉及到移动元素

**Traverse** - 遍历的时间时间复杂度是**O(n)**,也就是依赖于Capacity 的大小，如果你比较重视遍历的性能，就请不要不要给它设置一个很大的初始容量



## 四. 总结

1. ArrayList 就是一个实现了List接口的课自动扩容的数组，**当添加元素的时候它会尝试扩容,扩容的标准是变为原来的1.5倍**，当删除元素的时候，它会左移元素，避免数组出现"空位"
2. ArrayList 都有容量，容量就是ArrayList里面数组的大小
3. ArrayList 是一个有序的集合，它的维持的顺序就是元素的插入顺序（可以对比HashMap）
4. ArrayList 可以存储重复值和null值
5. ArrayList 是快速失败的，在遍历的同时当集合被修改后会抛出**ConcurrentModificationException**，可以使用Iterator 的删除方法来避免这个问题
6. ArrayList 不是线程安全的，如果你想在多线程环境中使用，可以使用Vector 或者它的线程安全包装类

