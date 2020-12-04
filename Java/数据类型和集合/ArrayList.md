[TOC]



## ArrayList

ArrayList是集合的一种实现，实现了接口List，List接口继承了Collection接口。

Collection是所有集合类的父类。ArrayList使用非常广泛，不论是数据库表查询，excel导入解析，还是网站数据爬取都需要使用到，了解ArrayList原理及使用方法显得非常重要。

ArrayList is one of the most used Collections and why shouldn't it be? It's easy to use, has many implemented methods to provide all the important functionality and it is fast. But that **fast performance comes at a cost**, [ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html) is **not synchronized**. What does that mean? *That means sharing an instance of ArrayList among many threads where those [threads](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html) are modifying (by adding or removing the values) the collection may result in unpredictable behavior.*

### ArrayList 的构造方法



```java
//默认创建一个ArrayList集合
List<String> list = new ArrayList<>();
//创建一个初始化长度为100的ArrayList集合
List<String> initlist = new ArrayList<>(100);
//将其他类型的集合转为ArrayList
List<String> setList = new ArrayList<>(new HashSet());
```

我们读一下源码，看看定义ArrayList的的构造方法和核心属性，当然我们还是习惯性的读一下类注释，让我们先有一个大概的认识，最后我们再通过例子，对它有一个精准的认识

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
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;

    /**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

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
}
```

其实源码里面已经很清晰了，ArrayList非线程安全，底层是一个Object[]，添加到ArrayList中的数据保存在了elementData属性中。

- 当调用`new ArrayList<>()`时，将一个空数组{}赋值给了elementData，这个时候集合的长度size为默认长度0；
- 当调用`new ArrayList<>(100)`时，根据传入的长度，new一个Object[100]赋值给elementData，当然如果玩儿的话，传了一个0，那么将一个空数组{}赋值给了elementData；
- 当调用new ArrayList<>(new HashSet())时，根据源码，我们可知，可以传递任何实现了Collection接口的类，将传递的集合调用toArray()方法转为数组内赋值给elementData;

> **注意：**在传入集合的ArrayList的构造方法中，有这样一个判断
>
> if (elementData.getClass() != Object[].class)，
>
> 给出的注释是：c.toArray might (incorrectly) not return Object[] (see 6260652)，即调用toArray方法返回的不一定是Object[]类型，查看ArrayList源码
>
> ```java
> public Object[] toArray() {    return Arrays.copyOf(elementData, size);}
> ```
>
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

#### 1. add(E element)

我们通过源码来看一下add("白骨精")到底发生了什么

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);
    // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

首先通过 `ensureCapacityInternal(size + 1)` 来保证底层Object[]数组有足够的空间存放添加的数据，然后将添加的数据存放到数组对应的位置上，我们看一下是怎么保证数组有足够的空间？

```java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
private void ensureExplicitCapacity(int minCapacity) {
	modCount++;
	// overflow-conscious code
	if (minCapacity - elementData.length > 0)
		grow(minCapacity);
}
```

这里首先确定了Object[]足够存放添加数据的最小容量，然后通过 `grow(int minCapacity)` 来进行数组扩容

```java
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

扩容规则为“**数组当前足够的最小容量 + （数组当前足够的最小容量 / 2）**”，即**数组当前足够的最小容量 \* 1.5**，当然有最大值的限制。

因为最开始定义了集合容量为10，故而本次不会进行扩容，直接将第8个位置（从0开始，下标为7）设置为“白骨精”，这时Object[] elementData中数据如下：

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:06:19-1677914-20190626150705934-1128204746.png)

还有和add()类似的方法。空间扩容原理都是一样，如：

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

#### 2. set(int index, E element)

因为ArrayList底层是由数组实现的，set实现非常简单，调用 `set(8, "猪八戒")` 通过传入的数字下标找到对应的位置，替换其中的元素，前提也需要首先判断传入的数组下标是否越界。*将“猕猴王”替换为“猪八戒”*。

```java
public E set(int index, E element) {
    rangeCheck(index);
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

//返回值“猕猴王”，当前数组中数据：

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:06:19-1677914-20190626150842478-1367690033.png)

#### 3. get(int index)

ArrayList中get方法也非常简单，通过下标查找即可，同时需要进行了类型转换，因为数组为Object[]，前提是需要判断传入的数组下标是否越界。

```java
public E get(int index) {
    rangeCheck(index);
    return elementData(index);
}
E elementData(int index) {
    return (E) elementData[index];
}
```

调用get(6)返回”哪吒“。

#### 4. remove(int index)

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

通过源码我们可以看到首先获取了待删除的元素，并最终返回了。其次计算了数组中需要移动的位数 size - index - 1，那么很明显我们可以得出待删除的是最后一个元素的话，移到位数为0，否则移动位数大于0，那么通过数组元素的拷贝来实现往前移动相应位数。

如remove(10)，找到的元素为“美猴王”，那么移动位数 = 12-10-1 = 1；此时将原本在第12个位置上（数组下标为11）的“白骨精”往前移动一位，同时设置elementData[11] = null；这里通过设置null值让GC起作用。

#### 5. remove(Object o)

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

#### 6. 其他方法

size() : 获取集合长度，通过定义在ArrayList中的私有变量size得到

isEmpty()：是否为空，通过定义在ArrayList中的私有变量size得到

contains(Object o)：是否包含某个元素，通过遍历底层数组elementData，通过equals或==进行判断

clear()：集合清空，通过遍历底层数组elementData，设置为null

## 三. 总结

本文主要讲解了ArrayList原理，从底层数组着手，讲解了ArrayList定义时到底发生了什么，再添加元素时，扩容规则如何，删除元素时，数组的元素的移动方式以及一些常用方法的用途，若有不对之处，请批评指正，望共同进步，谢谢！







### ArrayList in Java With Examples

Java ArrayList is one of the most used collection and most of its usefulness comes from the fact that it **grows dynamically**. Contrary to [arrays](https://www.netjstech.com/2017/02/array-in-java.html) you don't have to anticipate in advance how many elements you are going to store in the ArrayList. As and when elements are added ArrayList keeps growing, if required.

Though internally it is not really some "*elastic*" array which keeps growing, it is as simple as having an array with an **initial capacity** (default is array of length 10). When that limit is crossed **another array is created which is 1.5 times the original array** and the elements from the old array are copied to the new array.

- Refer [How does ArrayList work internally in Java](https://www.netjstech.com/2015/08/how-arraylist-works-internally-in-java.html) to know more about how does ArrayList work internally in Java.

**Table of contents**

1. [Hierarchy of the ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html#hierarchyarraylist)
2. [Features of ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html#arraylistfeatures)
3. [Java ArrayList constructors](https://www.netjstech.com/2015/09/arraylist-in-java.html#arraylistconstructors)
4. [Creating ArrayList and adding elements to it](https://www.netjstech.com/2015/09/arraylist-in-java.html#arraylistaddelements)
5. [Java ArrayList allows duplicates](https://www.netjstech.com/2015/09/arraylist-in-java.html#arraylistallowsduplicates)
6. [Java ArrayList allows any number of nulls](https://www.netjstech.com/2015/09/arraylist-in-java.html#arraylistallowsnulls)
7. [Removing elements from an ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html#arraylistelementremovals)
8. [ArrayList is not synchronized](https://www.netjstech.com/2015/09/arraylist-in-java.html#arraylistelementnotsynchronized)
9. [Java ArrayList iterator](https://www.netjstech.com/2015/09/arraylist-in-java.html#arraylistiterator)
10. [Performance of Java ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html#arraylistperformance)



### Hierarchy of the ArrayList

To know the hierarchy of java.util.ArrayList you need to know about 2 interfaces and 2 abstract classes.

- **Collection Interface**- Collection interface is the core of the Collection Framework. It must be implemented by any class that defines a collection.
- **List interface**- List interface extends Collection interface. Apart from extending all the methods of the Collection interface, List interface defines some methods of its own.
- **AbstractCollection**- Abstract class which implements most of the methods of the Collection interface.
- **AbstractList**- Abstract class which extends AbstractCollection and implements most of the List interface.

ArrayList extends AbstractList and implements List [interface](https://www.netjstech.com/2015/05/interface-in-java.html) too. Apart from List interface, ArrayList also implements **RandomAccess**, **Cloneable**, **java.io.Serializable** interfaces.

### Features of ArrayList

In this post we'll see in detail some of the salient features of the ArrayList in Java which are as follows.

- ArrayList is a Resizable-array implementation of the List interface. It can grow dynamically if more elements are to be added after the capacity is reached. Same way when the elements are removed from the ArrayList it shrinks by shifting the other elements to fill the space created by the removed element.
- Each ArrayList instance has a capacity. The capacity is the size of the array used to store the elements in the list.
- In ArrayList insertion order of the elements is maintained which means it is an ordered collection.
- ArrayList in Java can contain duplicate values. Any number of null elements are also allowed.
- ArrayList in Java is not thread-safe. In a multi-threaded environment if multiple threads access an ArrayList instance concurrently, and at least one of the threads modifies the list structurally, it must be synchronized externally.
- The iterators returned by ArrayList's iterator and listIterator methods are fail-fast. If the list is structurally modified at any time after the iterator is created, in any way except through the iterator's own remove or add methods, the iterator will throw a **ConcurrentModificationException**.

### Java ArrayList constructors

ArrayList class has the following three constructors.

- **ArrayList()**- Constructs an empty list with an initial capacity of ten.
- **ArrayList(Collection<? extends E> c)**- Constructs a list containing the elements of the specified collection, in the order they are returned by the collection's iterator.
- **ArrayList(int initialCapacity)**- Constructs an empty list with the specified initial capacity.

### Creating ArrayList and adding elements to it

List provides a method **add(E e)** which appends specified element to the end of the list. Using add(E e) method will mean keep **adding elements sequentially** to the list.

Apart from that there is another add method-

- **add(int index, E element)**- This method inserts the specified element at the specified position in this list.

There are other variants of add method too that can add the specified collection into the List. You can get the list of all methods in ArrayList class [here](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/ArrayList.html).

```
public class ArrayListDemo {
  public static void main(String[] args) {
    // List with initial capacity as 2
    List<String> cityList = new ArrayList<>(2);
    cityList.add("London");
    cityList.add("Paris");
    cityList.add("Bangalore");
    // With index
    cityList.add(3, "Istanbul");
    for(String name : cityList){
      System.out.println("City Name - " + name);
    }
  }
}
```

**Output**

```
City Name - London
City Name - Paris
City Name - Bangalore
City Name - Istanbul
```

In the code you can see that the ArrayList is created with the initial capacity as 2 still 4 elements are added to it. Internally ArrayList has been resized to accommodate more elements. Also from the output you can see that the elements are inserted in the same order as they are added, so the insertion order is maintained.

### Java ArrayList allows duplicates

ArrayList in Java allows *duplicate elements to be added*.

```
public class LoopListDemo {
  public static void main(String[] args) {
    // Using Diamond operator, so with ArrayList 
    // don't need to provide String, this option is available from 
    // Java 7 onward
    List<String> cityList = new ArrayList<>();
    cityList.add("Delhi");
    cityList.add("Mumbai");
    cityList.add("Bangalore");
    cityList.add("Mumbai");
    cityList.add("Mumbai");
            
    // Using for-each loop 
    System.out.println("With for-each loop - Java 5");
    for(String name : cityList){
      System.out.println("City Name - " + name);
    }
  }
}
```

**Output**

```
With for-each loop - Java 5
City Name - Delhi
City Name - Mumbai
City Name - Bangalore
City Name - Mumbai
City Name - Mumbai
```

Here it can be seen that Mumbai is added 3 times and when I am [looping the list](https://www.netjstech.com/2015/08/how-to-loop-iterate-arraylist-in-java.html) and displaying the elements in the list it is showing Mumbai 3 times.

### Java ArrayList allows any number of nulls

In ArrayList any number of nulls can be added. Let's see it with an example.

```
public class LoopListDemo {
  public static void main(String[] args) {
    // Using Diamond operator, so with ArrayList 
    // don't need to provide String, this option is available from 
    // Java 7 onwards
    List<String> cityList = new ArrayList<>();
    cityList.add("Delhi");
    cityList.add("Mumbai");
    cityList.add("Bangalore");
    cityList.add("Mumbai");
    cityList.add(null);
    cityList.add("Mumbai");
    cityList.add(null);
    
    // Using for-each loop 
    System.out.println("With for-each loop - Java 5");
    for(String name : cityList){
      System.out.println("City Name - " + name);
      //cityList.remove(2);
    } 
  }
}
```

**Output**

```
With for-each loop - Java 5
City Name - Delhi
City Name - Mumbai
City Name - Bangalore
City Name - Mumbai
City Name - null
City Name - Mumbai
City Name - null
```

It can be seen here that two null elements are added in the AraryList.

### Removing elements from an ArrayList

ArrayList provides several methods to remove elements from the List. Since ArrayList internally uses array to store elements, one point to note here is that *when an element is removed from the List internally the remaining elements are shifted to fill the gap created in the underlying array*.

- **clear()**- Removes all of the elements from this list.
- **remove(int index)**- Removes the element at the specified position in this list.
- **remove(Object o)**- Removes the first occurrence of the specified element from this list, if it is present.
- **removeAll(Collection<?> c)**- Removes from this list all of its elements that are contained in the specified collection.
- **removeIf(Predicate<? super E> filter)**- Removes all of the elements of this collection that satisfy the given predicate. **Note** that removeIf is added in Java 8.

```
public class ArrayListDemo {
  public static void main(String[] args) {
    // List with initial capacity as 2
    List<String> cityList = new ArrayList<>(2);
    cityList.add("London");
    cityList.add("Paris");
    cityList.add("Bangalore");
    cityList.add("Istanbul");
    cityList.add("Delhi");
    cityList.add("Houston");
    System.out.println("Original List- ");
    for(String name : cityList){
      System.out.println("City Name - " + name);
    }
    // Removing element at index 3
    String cityName = cityList.remove(3);
    System.out.println("Removed from the List- " + cityName);
    // using removeIf with a predicate
    cityList.removeIf((String name )->name.equalsIgnoreCase("Bangalore"));
    
    System.out.println("List after removal of elements-");
    
    for(String name : cityList){
      System.out.println("City Name - " + name);
    }
  }
}
```

**Output**

```
Original List- 
City Name - London
City Name - Paris
City Name - Bangalore
City Name - Istanbul
City Name - Delhi
City Name - Houston
Removed from the List- Istanbul
List after removal of elements-
City Name - London
City Name - Paris
City Name - Delhi
City Name - Houston
```

Note that parameter for the removeIf is of type Predicate which is a [functional interface](https://www.netjstech.com/2015/06/functional-interfaces-and-lambda-expression-in-java-8.html), so it's method can be implemented using [lambda expression](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html).

- Refer [How to remove elements from an ArrayList in Java](https://www.netjstech.com/2015/08/how-to-remove-elements-from-arraylist-java.html) to see Java program for removing elements from Arraylist.

### ArrayList is not synchronized

ArrayList in Java is not synchronized. That means sharing an instance of ArrayList among many [threads](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html) where those threads are modifying the collection (adding or removing the values) may result in unpredictable behaviour. If we need to [synchronize](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) an ArrayList you can use **synchronizedList** method provided by **Collections class**, which returns a synchronized (thread-safe) list backed by the specified list.

- Refer [How and why to synchronize ArrayList in Java](https://www.netjstech.com/2015/09/how-and-why-to-synchronize-arraylist-in-java.html) to read more about Synchronization and ArrayList.
- Refer [CopyOnWriteArrayList in Java](https://www.netjstech.com/2016/01/copyonwritearraylist-in-java.html) to know about a thread-safe variant of ArrayList.

### Java ArrayList iterator

ArrayList provides **iterator** to traverse the list in a sequential manner. Since ArrayList implements **List interface** so it provides **ListIterator** too *which is different from the iterator in a way that it provides iteration in both directions*.

- Refer [List iterator in Java](https://www.netjstech.com/2015/08/list-iterator-in-java.html) to know more about List Iterator in Java.
- Refer [How to loop/iterate an arraylist in Java](https://www.netjstech.com/2015/08/how-to-loop-iterate-arraylist-in-java.html) to know more about How to iterate a List in Java.

One point to note here is that **both iterator and listiterator are fail fast**, *fail-fast iterator fails if the underlying collection is structurally modified at any time after the iterator is created*, thus the iterator will throw a **ConcurrentModificationException** if the underlying collection is structurally modified in any way except through the iterator's own remove or add (if applicable as in list-iterator) methods.

**Structurally modifying ArrayList while iterating**

```
public class ArrayListDemo {
  public static void main(String[] args) {
    // List with initial capacity as 2
    List<String> cityList = new ArrayList<>(2);
    cityList.add("London");
    cityList.add("Paris");
    cityList.add("Bangalore");
    cityList.add("Istanbul");
    Iterator<String> itr = cityList.iterator();
    while(itr.hasNext()){
      String city = itr.next();
      if(city.equals("Paris")){
        // removing using remove method 
        // of the ArrayList class
        cityList.remove(city);
      }
    }
  }
}
```

**Output**

```
Exception in thread "main" java.util.ConcurrentModificationException
 at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:901)
 at java.util.ArrayList$Itr.next(ArrayList.java:851)
 at org.netjs.examples.ArrayListDemo.main(ArrayListDemo.java:18)
```

As you can see ConcurrentModificationException is thrown here as there is an attempt to remove an element from the ArrayList.

**Modifying ArrayList using iterator's remove method**

```
public class ArrayListDemo {
  public static void main(String[] args) {
    // List with initial capacity as 2
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
      }
    }
    // iterating after removal
    for(String name : cityList){
      System.out.println("City Name - " + name);
    }
  }
}
```

**Output**

```
City Name - London
City Name - Bangalore
City Name - Istanbul
```

Now ConcurrentModificationException is not thrown as iterator's remove method is used to remove element from the ArrayList.

- Refer [fail-fast Vs fail-safe iterator in Java](https://www.netjstech.com/2015/05/fail-fast-vs-fail-safe-iterator-in-java.html) to know more about fail-fast and fail-safe iterator.

### Performance of Java ArrayList

- **Adding an element**- If you are adding at the end using **add(E e)** method it is **O(1)**. Even in the case of adding at the last ArrayList may give O(n) performance in the worst case. That will happen if you add more elements than the capacity of the underlying array, as in that case *a new array (1.5 times the last size) is created, and the old array is copied to the new one*. If you are using **add(int index, E element)** then it is **O(n - index)** and it'll become O(n) if every time element is added at the beginning of the list.
- **Retrieving an element**- Since ArrayList internally uses an array to store elements so **get(int index)** means going to that index directly in the array. So, for ArrayList get(int index) is **O(1)**.
- **Removing an element**- If you are removing using the **remove(int index)** method then, in case of ArrayList getting to that index is fast *but removing will mean shuffling the remaining elements to fill the gap created by the removed element with in the underlying array*. It ranges from O(1) for removing the last element to O(n). Thus it can be said remove(int index) operation is O(n - index) for the arraylist.

- Refer [Difference between ArrayList and LinkedList in Java](https://www.netjstech.com/2015/08/difference-between-arraylist-and-linkedlist-in-java.html) to know about the performance of ArrayList and LinkedList for various operations.





### How ArrayList Works Internally in Java

ArrayList arguably would be the most used collection along with the [HashMap](https://www.netjstech.com/2015/11/difference-between-hashmap-and-hashtable-java.html). Many of us programmers whip up code everyday which contains atleast one of these data structures to hold objects. I have already discussed [how HashMap works internally in Java](https://www.netjstech.com/2015/05/how-hashmap-internally-works-in-java.html), in this post I'll try to explain **how ArrayList internally works in Java**.

As most of us would already be knowing that **ArrayList is a Resizable-array implementation** of the List interface i.e. ArrayList grows dynamically as the elements are added to it. So let's try to get clear idea about the following points-

- How ArrayList is internally implemented in Java.
- What is the backing data structure for an [ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html).
- How it grows dynamically and ensures that there is always room to add elements.

Because of all these side questions it is also a very important [Java Collections interview question](https://www.netjstech.com/2015/11/java-collections-interview-questions.html).



**Note** - Code of ArrayList used here for reference is from **Java 10**.

**Table of contents**

1. [Where does ArrayList internally store elements](https://www.netjstech.com/2015/08/how-arraylist-works-internally-in-java.html#arraylistinternalstorage)
2. [What happens when ArrayList is created](https://www.netjstech.com/2015/08/how-arraylist-works-internally-in-java.html#arraylistcreation)
3. [How does ArrayList grow dynamically](https://www.netjstech.com/2015/08/how-arraylist-works-internally-in-java.html#arraylistdynamic)
4. [What happens when an element is removed from ArrayList](https://www.netjstech.com/2015/08/how-arraylist-works-internally-in-java.html#arraylistelementremoval)



### Where does ArrayList internally store elements

Basic data structure used by Java ArrayList to store objects is an [array](https://www.netjstech.com/2017/02/array-in-java.html) of [Object class](https://www.netjstech.com/2017/06/object-class-in-java.html), which is defined as follows -

```
transient Object[] elementData;
```

I am sure many of you would be thinking why [transient](https://www.netjstech.com/2017/04/transient-in-java.html) and how about [serializing](https://www.netjstech.com/2017/04/serialization-in-java.html) an ArrayList then?
ArrayList provides its own version of **readObject** and **writeObject** methods so no problem in serializing an ArrayList and that is the reason, I think, of making this Object array as **transient**.

### What happens when ArrayList is created

ArrayList class in Java provides **3 constructors** to create an ArrayList.

- public ArrayList(int initialCapacity)

   

  \- When this

   

  constructor

   

  is used we can provide some initial capacity rather than depending on the default capacity as defined in the ArrayList class.

  As example -

  ```
  List<String> myList = new ArrayList<String>(7);
  ```

  Code in the ArrayList class is as -

  ```
  public ArrayList(int initialCapacity) {
       if (initialCapacity > 0) {
          this.elementData = new Object[initialCapacity];
       } else if (initialCapacity == 0) {
          this.elementData = EMPTY_ELEMENTDATA;
       } else {
          throw new IllegalArgumentException("Illegal Capacity: "+
                                                 initialCapacity);
       }
  }
  ```

  Where **EMPTY_ELEMENTDATA** is defined as -

  ```
  private static final Object[] EMPTY_ELEMENTDATA = {};
  ```

  It is easy to see that, if provided capacity is greater than zero then the elementData array will be created with that capacity, in case provided capacity is zero then elementData array is initialized with an empty Object array. In that case ArrayList will grow when first element is added.

- public ArrayList()

   

  \- In case

   

  default constructor

   

  is used i.e. ArrayList is created like -

  ```
   myList = new ArrayList();
  ```

  Code in the ArrayList class is as -

  ```
  public ArrayList() {
      this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
  }
  ```

  Where **DEFAULTCAPACITY_EMPTY_ELEMENTDATA** is defined as

  ```
  /**
   * Shared empty array instance used for default sized empty instances. We
   * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
   * first element is added.
   */
  private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
  ```

  So you can see initially it will be initialized with an empty array, it will grow only when first element is added to the list.

- public ArrayList(Collection<? extends E> c)

   

  \- If we want to construct a list containing the elements of the specified collection we can use this

   

  constructor

  . In this constructor implementation checks for the length of the collection passed as parameter, if length is greater than zero then

   

  Arrays.copyOf

   

  method is used to copy the collection to the elementData array.

  ```
  elementData = Arrays.copyOf(elementData, size, Object[].class);
  ```

### How does ArrayList grow dynamically

When we add an element to an ArrayList it first verifies whether it has that much capacity in the array to store new element or not, in case there is not then the new capacity is calculated which is 50% more than the old capacity and the array is increased by that much capacity (Actually uses Arrays.copyOf which returns the original array increased to the new length).

Code in the Java ArrayList implementation is like this-

```
public boolean add(E e) {
     ensureCapacityInternal(size + 1);  // Increments modCount!!
     elementData[size++] = e;
     return true;
}
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
```

Where DEFAULT_CAPACITY is defined as -

```
private static final int DEFAULT_CAPACITY = 10;
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
       grow(minCapacity);
}
```

You can see here it is determined if there is a need to increase the size of the array, if yes then grow method is called.

```
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

Note that till **Java 6** the new capacity calculation used to be like this -

```
int newCapacity = (oldCapacity * 3)/2 + 1;
```

Which is changed in **Java 7** to use right shift operator. With right shift operator also it will grow by 50% of old capacity.
Let's see it with the help of a small program

```
public class Test {
    public static void main(String args[])  {
       int a = 10;
       System.out.println(a>>1);   
    }    
}
```

**Output**

5

If the default capacity was 10 then

```
int newCapacity = oldCapacity + (oldCapacity >> 1);
```

will return 15.

### What happens when an element is removed from ArrayList

When elements are removed from an ArrayList in Java using either **remove(int i)** (i.e using index) or **remove(Object o)**, gap created by the removal of an element has to be filled in the underlying array. That is done by Shifting any subsequent elements to the left (subtracts one from their indices). **System.arrayCopy** method is used for that.

```
System.arraycopy(elementData, index+1, elementData, index, numMoved);
```

Here index+1 is the source position and index is the destination position. Since element at the position index is removed so elements starting from index+1 are copied to destination starting from index.

**Points to note**

1. ArrayList in Java is a Resizable-array implementation of the List interface.
2. Internally ArrayList class uses an array of Object class to store its elements.
3. When initializing an ArrayList you can provide initial capacity then the array would be of the size provided as initial capacity.
4. If initial capacity is not specified then default capacity is used to create an array. Default capacity is 10.
5. When an element is added to an ArrayList it first verifies whether it can accommodate the new element or it needs to grow, in case capacity has to be increased then the new capacity is calculated which is 50% more than the old capacity and the array is increased by that much capacity.
6. When elements are removed from an ArrayList space created by the removal of an element has to be filled in the underlying array. That is done by Shifting any subsequent elements to the left.