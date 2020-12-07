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
 * The amount by which the capacity of the vector is automatically incremented when its size becomes greater than its capacity.  If
 * the capacity increment is less than or equal to zero, the capacity
 * of the vector is doubled each time it needs to grow.
 *
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







## 三. Vector 的常用方法





### 遍历方法

Vector支持**4种遍历方式**。建议使用下面的第二种去遍历Vector，因为效率问题。

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

