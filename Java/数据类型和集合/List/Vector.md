[TOC]

## 一. Vector 初识

vector类称作向量类，它实现了动态数组，用于元素数量变化的对象数组。像数组一样，vector类也用从0开始的下标表示元素的位置；但和数组不同的是，当vector对象创建后，数组的元素个数会随着vector对象元素个数的增大和缩小而自动变化。

但是在Java 中拿Vector 来和数组比较是不合理的，因为数组在java 集合中的地位，往往是作为底层存储使用的，也就是是很多java 中的集合底层使用的都是数组，因为按照内存的划分方式，可以分为连续内存和不连续内存，这个时候就对应两种数据结构，数组和链表，例如在java 中ArrayList、HashMap 等数据结构都是借助数组实现的，当然也有很多借助HashMap 实现的数据结构，那其实也是借助数组实现的。

**其实Vector和ArrayList一样，都是基于数组实现的List，其主要的区别是在于线程安全上**，前面讲ArrayList的时候我们讲到ArrayList不是线程安全的，一种寻求线程安全的解决方式就是ArrayList 的封装类`Collections.synchronizedList(new ArrayList<>());`就像这样，还有一个解决方法就是使用Vector。

还有一点就是，Vector 虽然和ArrayList 都是List 的实现，但是Vector 是比ArrayList出现的要早的，Vector 在JDK1.0 的时候就存在的，但是ArrayList是在JDK1.2 的时候才出现的

### 1. Vector 的说明书

在看说明书之前，我们还是先看一下整个类的继承关系

![image-20201207221229060](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/07/22:12:30-image-20201207221229060.png)

从这里我们可以看到，它的确是属于List家族的



```java
/**
 * The {@code Vector} class implements a growable array of objects. Like an array, it contains components that can be
 * accessed using an integer index. However, the size of a {@code Vector} can grow or shrink as needed to accommodate
 * adding and removing items after the {@code Vector} has been created.
 * Vector 实现了动态数组，就像数组一样，它的元素可以使用整型下标进行访问，但是它的大小在它创建之后，伴随着添加和删除，可以增加也可以减少
 * <p>Each vector tries to optimize storage management by maintaining a {@code capacity} and a {@code capacityIncrement}. The
 * {@code capacity} is always at least as large as the vector size; 
 * vector 一致通过维持capacity和capacityIncrement来优化存储管理，capacity 至少要和size 一样大
 * it is usually larger because as components are added to the vector, the vector's storage increases in chunks the size of 
 * {@code capacityIncrement}. 
 * capacity 通常是比size 要大的，那是因为当元素被添加到vector的时候，vector的存储通常是以capacityIncrement的大小增加
 * An application can increase the capacity of a vector before inserting a large number of components; this reduces the amount of
 * incremental reallocation.
 * 一个应用程序可以在插入大量数据之前增加vector的capacity，这样可以减少增量重新分配的开销
 * <p><a name="fail-fast"> The iterators returned by this class's {@link #iterator() iterator} and {@link #listIterator(int) listIterator} 
 * methods are <em>fail-fast</em></a>: if the vector is structurally modified at any time after the iterator is created, in any way 
 * except  through the iterator's own {@link ListIterator#remove() remove} or {@link ListIterator#add(Object) add} methods, the iterator 
 * will throw a {@link ConcurrentModificationException}.  Thus, in the face of
 * concurrent modification, the iterator fails quickly and cleanly, rather than risking arbitrary, non-deterministic behavior at 
 * an undetermined  time in the future.  
 * 上面这一段主要江的实快速失败，就是如果你在遍历的过程中，集合发生了修改，你的遍历就会失败，所以你可以使用iterator 的修改方法，而不是集合的修改方法
 * The {@link Enumeration Enumerations} returned by the {@link #elements() elements} method are <em>not</em> fail-fast.
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification.  
 * Fail-fast   iterators  throw {@code ConcurrentModificationException} on a best-effort basis. Therefore, it would be wrong to write a
 * program that depended on this
 * exception for its correctness:  <i>the fail-fast behavior of iterators  should be used only to detect bugs.</i>
 * 关于这一段，前面也翻译过好几次了，因为每次都是一样的，这里就不翻译了
 * <p>As of the Java 2 platform v1.2, this class was retrofitted to implement the {@link List} interface, making it a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html"> Java Collections Framework</a>.  Unlike the new collection
 * implementations, {@code Vector} is synchronized.  If a thread-safe implementation is not needed, it is recommended to use {@link
 * ArrayList} in place of {@code Vector}.
 * 在java1.2的版本中也就是java2的平台上，对该类进行了改进，实现了List 接口，使其成为了java Collections 的一员，与新集合(ArrayList)的实现不同的是，
 * Vector 是同步的，如果不需要一个线程安全的实现，推荐使用ArrayList代替Vector
 * @author  Lee Boynton
 * @author  Jonathan Payne
 * @see Collection
 * @see LinkedList
 * @since   JDK1.0
 */
public class Vector<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{ ... ... }
```



### 2.Vector 的内部构成



#### elementData

elementData 就是存储Vector中的元素的，本身是一个数组

```java
/**
 * The array buffer into which the components of the vector are stored. The capacity of the vector is the length of this array buffer,
 * and is at least large enough to contain all the vector's elements.
 * elementData 是一个数组缓存，用来存存储vector的元素，vectord的容量就是array buffer 的大小，这个大小至少是要能够存储vector 中的全部元素
 * <p>Any array elements following the last element in the Vector are null.
 * @serial
 */
  protected Object[] elementData;
```



#### elementCount

Vector 存储的元素个数，其实就是ArrayList 中的size ，只不过名字不一样而已

```java
/**
 * The number of valid components in this {@code Vector} object. Components {@code elementData[0]} through
 * {@code elementData[elementCount-1]} are the actual items.
 * Vector 中有效的元素个数，元素在elementData数组中是从0也就是elementData[0]到elementCount-1也就是elementData[elementCount-1]
 * @serial
 */
protected int elementCount;
```



#### capacityIncrement

capacityIncrement 是动态数组的增长系数，你也可以理解为增量。如果在创建Vector时，指定了capacityIncrement的大小；则，每次当Vector中动态数组容量增加时，增加的大小都是capacityIncrement。这个类的注解中也有提到过

```java
/**
 * The amount by which the capacity of the vector is automatically incremented when its size becomes greater than its capacity.  
 * 当vector的size( elementCount) 大于vector 的capacity(容量)的时候，它的容量会自动增长
 * If the capacity increment is less than or equal to zero, the capacity
 * of the vector is doubled each time it needs to grow.
 * 如果容量增量(capacityIncrement) 小于等于0，则vector每次在需要扩容的时候增长为原来的两倍
 * @serial
 */
protected int capacityIncrement;
```



### 3. Vector 的构造方法

```java
// 默认构造函数
Vector()
// capacity是Vector的默认容量大小。当由于增加数据导致容量增加时，每次容量会增加一倍。
Vector(int capacity)
// capacity是Vector的默认容量大小，capacityIncrement是每次Vector容量增加时的增量值。
Vector(int capacity, int capacityIncrement)
// 创建一个包含collection的Vector
Vector(Collection<? extends E> collection)
```

#### 无参构造

```java
    /**
     * Constructs an empty vector so that its internal data array has size {@code 10} and its standard capacity increment is zero.
     * 创建一个空的vector,因此其内部的数组的大小是10并且其容量的增量是0
     */
    public Vector() {
        this(10);
    }
```



#### 指定初始容量

```java
    /**
     * Constructs an empty vector with the specified initial capacity and
     * with its capacity increment equal to zero.
     * 创建一个指定初始容量和容量增量为0的空的 vector
     * @param   initialCapacity   the initial capacity of the vector
     * @throws IllegalArgumentException if the specified initial capacity is negative 当参数是负的时候，抛出异常
     */
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }
```



#### 指定初始容量和增量

其实上面的构造方法都是调用的是这个构造方法

```java
    /**
     * Constructs an empty vector with the specified initial capacity and
     * capacity increment.
     * 创建一个指定初始容量和增量的空vector
     * @param   initialCapacity     the initial capacity of the vector
     * @param   capacityIncrement   the amount by which the capacity is
     *                              increased when the vector overflows
     * @throws IllegalArgumentException if the specified initial capacity is negative
     * 初始容量如果是负的话抛出异常
     */
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }
```



#### 基于其他集合

```java

    /**
     * Constructs a vector containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     * 创建一个包含指定集合内容元素的vector,vector中的元素按照集合的遍历顺序存储
     * @param c the collection whose elements are to be placed into this vector
     * @throws NullPointerException if the specified collection is null 如果参数集合为空则抛出异常
     * @since   1.2
     */
    public Vector(Collection<? extends E> c) {
        // 因为这里调用了该集合的toArray 方法，所以如果集合为空，则抛出异常
        Object[] a = c.toArray();
        elementCount = a.length;
        // 下面则完成对集合元素的添加，如果c 是ArrayList 的话，则直接完成赋值，否则的话使用Arrays 的copy 方法
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            elementData = Arrays.copyOf(a, elementCount, Object[].class);
        }
    }
```



## 二. Vector 的常用方法



### 1. add 方法

```java
    @Test
    public void testAdd() {
        Vector<String> vector = new Vector(10);
        vector.add("a");
    }

```

下面我们跟踪一下add 方法的源码

```java
    /**
     * Appends the specified element to the end of this Vector.
     * 添加特定的元素到Vector的尾部，需要注意的是这个方法是synchronized
     * @param e element to be appended to this Vector
     * @return {@code true} (as specified by {@link Collection#add})
     * @since 1.2
     */
    public synchronized boolean add(E e) {
        // 记录修改
        modCount++;
        // 保证容量，这个就是在添加元素之前要要保证内部的数组大小足够可以容纳该元素,在ArrayList 里面也有，是这样的  ensureCapacityInternal(size + 1)，(elementCount + 1) 可以认为是需要的
        ensureCapacityHelper(elementCount + 1);
        // 容量可以保证之后，我们就将这个元素添加到数组的指定位置
        elementData[elementCount++] = e;
        return true;
    }
```

接下来，我们看一下`ensureCapacityHelper`的代码实现

```java
    /**
     * This implements the unsynchronized semantics of ensureCapacity.
     * Synchronized methods in this class can internally call this
     * method for ensuring capacity without incurring the cost of an
     * extra synchronization.
     *
     * @see #ensureCapacity(int)
     */
    private void ensureCapacityHelper(int minCapacity) {
        // 判断需要扩容吗，如果所需的最小容量大于实际可存储的容量则需要扩容
        if (minCapacity - elementData.length > 0)
            // 扩容方法
            grow(minCapacity);
    }
```

下面则是具体的扩容方法

```java
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 如果capacityIncrement>0 则新的容量是oldCapacity+capacityIncrement ；否则2倍的oldCapacity
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ? capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 完成扩容和数据迁移
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```



### 2. add(int index, E element)

这个方法主要是用来将元素**插入到指定位置**，而 **add(E e)则是将元素添加到vector 的尾部**

```java
    /**
     * Inserts the specified element at the specified position in this Vector.
     * Shifts the element currently at that position (if any) and any
     * subsequent elements to the right (adds one to their indices).
     * 插入一个指定的元素到Vector的指定位置，如果当前位置有元素的话，则需要将当前位置的元素和其后面的元素移动
     * 到前位置 的右边
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index > size()})
     * @since 1.2
     */   
 	public void add(int index, E element) {
        insertElementAt(element, index);
    }
```

接下来我么看一下insertElementAt 的具体实现

```java
    /**
     * Inserts the specified object as a component in this vector at the
     * specified {@code index}. Each component in this vector with
     * an index greater or equal to the specified {@code index} is
     * shifted upward to have an index one greater than the value it had
     * previously.
     *
     * <p>The index must be a value greater than or equal to {@code 0}
     * and less than or equal to the current size of the vector. (If the
     * index is equal to the current size of the vector, the new element
     * is appended to the Vector.)
     *
     * <p>This method is identical in functionality to the
     * {@link #add(int, Object) add(int, E)}
     * method (which is part of the {@link List} interface).  Note that the
     * {@code add} method reverses the order of the parameters, to more closely
     * match array usage.
     *
     * @param      obj     the component to insert
     * @param      index   where to insert the new component
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index > size()})
     */
    public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        ensureCapacityHelper(elementCount + 1);
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        elementData[index] = obj;
        elementCount++;
    }
```



### 3. get 方法

```java
    /**
     * Returns the element at the specified position in this Vector.
     * 返回Vector中特定位置的的元素(需要注意的是这个方法，依然是加了synchronized 修饰的)
     * @param index index of the element to return
     * @return object at the specified index
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *            ({@code index < 0 || index >= size()})
     * @since 1.2
     */
    public synchronized E get(int index) {
        //  如果index >= elementCount 则抛出异常，因为最后一个元素是elementData[elementCount-1]
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
    }
    /**
    * 返回特定的数据
    */
    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }
```



### 4. 遍历方法

#### 四种遍历方式

Vector支持**4种遍历方式**。可能在效率上存在些许差别，但是差别不大

第一种，通过**迭代器**遍历。即通过Iterator去遍历。

```java
Integer value = null;
Iterator<int> size = vec.iterator();
while(size.hasNext()){
value = size.next();
}
```

 第二种，**随机访问**，通过索引值去遍历。由于Vector实现了RandomAccess接口，它支持通过索引值去随机访问元素。

```java
Integer value = null;
int size = vec.size();
for (int i=0; i<size; i++) {
    value = (Integer)vec.get(i);        
}
```

 第三种，**另一种for循环**

```java
Integer value = null;
for (Integer integ:vec) {
    value = integ;
}
```

第四种，**Enumeration遍历**

```java
Integer value = null;
Enumeration enu = vec.elements();
while (enu.hasMoreElements()) {
    value = (Integer)enu.nextElement();
}
```



#### 遍历性能测试

测试代码，在下边，这里先把测试结果贴出来，可以看出来整体性能差异不大

```
iteratorThroughRandomAccess：313 ms
iteratorThroughIterator：279 ms
iteratorThroughFor2：287 ms
iteratorThroughEnumeration：289 ms
```



```java
  @Test
    public  void iterator() {
        Vector vec = new Vector();
        for (int i = 0; i < 10000000; i++) {
            vec.add(i);
        }
        iteratorThroughRandomAccess(vec);
        iteratorThroughIterator(vec);
        iteratorThroughFor2(vec);
        iteratorThroughEnumeration(vec);

    }


    private static void isRandomAccessSupported(List list) {
        if (list instanceof RandomAccess) {
            System.out.println("RandomAccess implemented!");
        } else {
            System.out.println("RandomAccess not implemented!");
        }

    }

    public static void iteratorThroughRandomAccess(List list) {

        long startTime;
        long endTime;
        startTime = System.currentTimeMillis();
        int size = list.size();
        // 这里注意一下，你不要把i < list.size() 直接放在for 循环里，因为这样每次都会调用size 方法就会有性能损耗
        for (int i = 0; i < size; i++) {
            list.get(i);
        }
        endTime = System.currentTimeMillis();
        long interval = endTime - startTime;
        System.out.println("iteratorThroughRandomAccess：" + interval + " ms");
    }

    public static void iteratorThroughIterator(List list) {

        long startTime;
        long endTime;
        startTime = System.currentTimeMillis();
        for (Iterator iter = list.iterator(); iter.hasNext(); ) {
            iter.next();
        }
        endTime = System.currentTimeMillis();
        long interval = endTime - startTime;
        System.out.println("iteratorThroughIterator：" + interval + " ms");
    }


    public static void iteratorThroughFor2(List list) {

        long startTime;
        long endTime;
        startTime = System.currentTimeMillis();
        for (Object obj : list)
            ;
        endTime = System.currentTimeMillis();
        long interval = endTime - startTime;
        System.out.println("iteratorThroughFor2：" + interval + " ms");
    }

    public static void iteratorThroughEnumeration(Vector vec) {

        long startTime;
        long endTime;
        startTime = System.currentTimeMillis();
        for (Enumeration enu = vec.elements(); enu.hasMoreElements(); ) {
            enu.nextElement();
        }
        endTime = System.currentTimeMillis();
        long interval = endTime - startTime;
        System.out.println("iteratorThroughEnumeration：" + interval + " ms");
    }
```

#### 对比ArrayList

我们还是先贴出对比结果，上面是Vector 的遍历耗时，下面是ArrayList 的遍历耗时，这里我们就可以看出ArrayList 的17ms 远远的碾压了Vector 的247ms

```
iteratorThroughIterator：247 ms
iteratorThroughIterator：17 ms
```

这里，我么贴出遍历的代码

```java
    @Test
    public  void iterator2() {
        Vector vec = new Vector();
        ArrayList arr = new ArrayList();
        for (int i = 0; i < 10000000; i++) {
            vec.add(i);
            arr.add(i);
        }
        //iteratorThroughIterator 还是用的是上面的代码
        iteratorThroughIterator(vec);
        iteratorThroughIterator(arr);
    }
```



#### 对比 Collections.synchronizedList()

这里与其说是对比，不如说是给大家一个提醒，那就是当你在遍历Collections.synchronizedXXX() 的返回集合的时候，你需要注意线程安全，因为这里返回的对象，它的遍历方法并不是同步的

```
iteratorThroughIterator：221 ms
iteratorThroughIterator：12 ms
```

下面是它的源代码

```java
public ListIterator<E> listIterator() {
    return list.listIterator(); // Must be manually synched by user
}
```



下面是测试代码

```java

    @Test
    public  void iterator3() {
        Vector vec = new Vector();
        ArrayList arr = new ArrayList();
        for (int i = 0; i < 10000000; i++) {
            vec.add(i);
            arr.add(i);
        }
        iteratorThroughIterator(vec);
        iteratorThroughIterator(Collections.synchronizedList(arr));
    }
```



## 三 总结

**其实Vector和ArrayList一样，都是基于数组实现的List，也就是说都是属于List 阵营的，其主要的区别是在于线程安全上**，二者的底层实现都是基于数组的

Vector 实现线程安全的方式就是给方法上加synchronized 锁，**所以线程安全的情况下请使用ArrayList，多线程的情况下使用Vector**

