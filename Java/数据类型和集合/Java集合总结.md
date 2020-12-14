[TOC]

##  一. 集合框架

Java 集合框架一些列的接口和类来实现很多常见的数据结构和算法，例如 `LinkedList` 就是集合框架提供的实现了双向链表的数据结构，关于这一篇文章建议大家收藏，我会不断地完善和扩充它的内容，例如最下面的系列文章我以后也会对它进行不断的更新

### 集合框架的接口

集合框架提供了很多接口，这些接口都包含了特定的方法来实现对集合上的特定操作

![image-20201213141107767](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/14:11:08-image-20201213141107767.png)

我们将要学习这些接口以及子接口和它们的各种实现类，在开始之前我们先简单学习一下这些广泛运用的接口，可以看到整个集合框架，总共有三个顶级接口Collection 和 Map 以及Iterator,首先上面这图你需要记住，因为这张图你记住了，那你至少对整个Java 集合框架是有了一个框架上的认识，接下来就是慢慢的将它填充，使其更加具体和细化



#### Collections Framework 和 Collection Interface 的区别

很多人可能对这二者感到困惑，`Collection` interface  是Collections Framework  的顶级接口，集合框架提供可一些列的实现了数据结构和算法的接口和实现类，顶级的接口除了`Collection`还有`Map` 和 `Iterator`.



#### Collections Framework  的意义

前面说到了集合框架实现了数据和算法的实现，它们可以被直接使用，，对使用者而言有两方面的意义

1、 我们不需要自己去写代码去实现这些数据结构和算法

2、即使我们实现了这些代码，我们也要面临如何去优化这些代码使其变得更加高效



除此之外集合框架还还允许我们针对特殊的数据使用不同的数据集合，例如

1、如果你想要数据是去重的，或者是唯一的，你可易使用`Set`集合

2、如果你想存储 **key/value** 对，你可以使用 `Map` 集合

3、 `ArrayList`提供了动态扩容的数组

### Map 接口

在Java中， `Map` 接口允许元素以 **key-value**对的形式存储，其中key作为获取特定元素的唯一方法，每个key 都有和其对应的value，也就是说Map 中的元素是以 **key-value**对存储的



![image-20201213192330901](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/19:23:31-image-20201213192330901.png)



###  Iterator 接口

在java 中，Iterator 接口是用来访问集合中的元素的，并且它有一个子接口`ListIterator`

![image-20201213194048093](/Users/liuwenqiang/Library/Application%20Support/typora-user-images/image-20201213194048093.png)



所有的java 集合都有`iterator()`方法，这个方法返回一个iterator实例，用来遍历集合中的全部元素

### Collection 接口

 `Collection` 接口是  Java collections framework 的顶级接口，整个java集合框架中也没有 `Collection` 接口的直接实现，Java集合框架中提供的是它的子接口的实现，就像`List`, `Set` 和 `Queue `

例如java 的`ArrayList` 类就是实现了`List`接口，而`List`接口就是`Collection`接口的子接口



![image-20201213140854528](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/14:08:55-image-20201213140854528.png)

#### Collection的子接口

就像前面提到的，在java 中有很多`Collection` 接口的子接口的实现类

##### 1.List Interface

 `List` interface 是一个有序的集合，允许像数组一样添加或者删除元素

![image-20201213195250946](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/19:52:51-image-20201213195250946.png)



List 接口的方法

```java
add() - adds an element to a list
addAll() - adds all elements of one list to another
get() - helps to randomly access elements from lists
iterator() - returns iterator object that can be used to sequentially access elements of lists
set() - changes elements of lists
remove() - removes an element from the list
removeAll() - removes all the elements from the list
clear() - removes all the elements from the list (more efficient than removeAll())
size() - returns the length of lists
toArray() - converts a list into an array
contains() - returns true if a list contains specified element
```



##### 2.Set Interface

`Set`接口允许我们将元素存储在不同的集合中，类似于数学中的集合,它不能有重复的元素

Set 的实现类

![image-20201213201606159](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/20:16:06-image-20201213201606159.png)

Set的子接口

![image-20201213195608648](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/19:56:09-image-20201213195608648.png)



Set 接口的方法

```java
add() - adds the specified element to the set
addAll() - adds all the elements of the specified collection to the set
iterator() - returns an iterator that can be used to access elements of the set sequentially
remove() - removes the specified element from the set
removeAll() - removes all the elements from the set that is present in another specified set
retainAll() - retains all the elements in the set that are also present in another specified set
clear() - removes all the elements from the set
size() - returns the length (number of elements) of the set
toArray() - returns an array containing all the elements of the set
contains() - returns true if the set contains the specified element
containsAll() - returns true if the set contains all the elements of the specified collection
hashCode() - returns a hash code value (address of the element in the set)
```



##### 3.Queue Interface

 `Queue` 接口主要用在当我们想要 以**First In, First Out(FIFO)** 的方式存储和访问集合中的元素的时候,在队列中元素从队尾添加，从队头删除

![image-20201213201000889](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/20:10:01-image-20201213201000889.png)

Queue的主要实现类

![image-20201213200818475](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/20:08:19-image-20201213200818475.png)

Queue的子接口

![image-20201213201511102](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/20:15:11-image-20201213201511102.png)

Queue的主要方法

```java
add() - Inserts the specified element into the queue. If the task is successful, add() returns true, if not it throws an exception.
offer() - Inserts the specified element into the queue. If the task is successful, offer() returns true, if not it returns false.
element() - Returns the head of the queue. Throws an exception if the queue is empty.
peek() - Returns the head of the queue. Returns null if the queue is empty.
remove() - Returns and removes the head of the queue. Throws an exception if the queue is empty.
poll() - Returns and removes the head of the queue. Returns null if the queue is empty.
```

 

#### Collection 接口的方法

The `Collection` interface includes various methods that can be used to perform different operations on objects. These methods are available in all its subinterfaces.

- `add()` - inserts the specified element to the collection
- `size()` - returns the size of the collection
- `remove()` - removes the specified element from the collection
- `iterator()` - returns an iterator to access elements of the collection
- `addAll()` - adds all the elements of a specified collection to the collection
- `removeAll()` - removes all the elements of the specified collection from the collection
- `clear()` - removes all the elements of the collection

## 二. List 体系



![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/30/08:55:30-1677914-20190630110527653-156301420.png)

首先上面的框架图可以表明顺序的关联关系，但并不全面，如ArrayList在继承了AbstractList抽象类的同时还实现了List接口。

1. List是一个接口，继承了Collection，同时Collection继承了Iterable，表明List的实现类都是可用迭代遍历的；

2. AbstractList是一个抽象类，实现了List接口，同时继承了AbstractCollection，针对一些常用方法，如add()，set()，remove()，给了默认实现，当然在具体的实现类中基本都重写了，该类中没有get()，size()方法。

3. AbstractSequentialList是一个抽象类，继承了AbstractList抽象类，实现了很多双向链表中根据索引操作的方法。

4. ArrayList、Vector、LinkedList、Stack都是具体的实现类。

   

![image-20201213140626929](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/14:06:27-image-20201213140626929.png)



### 1 . ArrayList和Vector对比分析

| 类型      | 线程安全 | 内部结构     |      | 扩容规则                                     | 执行效率 | 序列化 |
| --------- | -------- | ------------ | ---- | -------------------------------------------- | -------- | ------ |
| ArrayList | 否       | 数组Object[] | 10   | 数组足够最小长度*1.5                         | 高       | 是     |
| Vector    | 是       | 数组Object[] | 10   | 默认数组足够最小长度*2，可自定义每次扩容数量 | 低       | 是     |

Vertor扩容方法：

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    //capacityIncrement参数可通过构造函数传递进来，若没传递该参数，则数组大小设置为elementData.length * 2
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //扩容有上限
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### 2. ArrayList和LinkedList对比分析

| 类型       | 内部结构     | 插入效率(正常情况) | 删除效率(正常情况) | 顺序遍历效率 | 随机遍历效率 | 占用内存 | 序列化 |
| ---------- | ------------ | ------------------ | ------------------ | ------------ | ------------ | -------- | ------ |
| ArrayList  | 数组Object[] | 低                 | 低                 | 高           | 高           | 低       | 是     |
| LinkedList | 双向链表Node | 高                 | 高                 | 高           | 低           | 高       | 是     |

上述的对比都是基于大数据量的情况下，如果只是几个元素或几十个元素，它们之间并没有多大区别。

------

问：插入效率为何说正常情况下ArrayList低，LinkedList高呢？

答：我们清楚ArrayList之所以插入效率低，有两个原因会造成时间的消耗。

 第一，当底层数组空间不足时需要扩容，扩容后需进行数组拷贝

 第二，当不在数组末尾插入数据，那么就需要移动数组元素

 知道了其插入效率低的原因后，那么很明显，数据扩容及拷贝只有在数组空间不足时才发生，如果我们正确使用，就像《阿里巴巴Java开发手册》中提到我们在创建集合对象时，就传递参数预先设置好数组大小，那么插入效率是非常高的；而90%的情况下我们在添加元素时都调用的是add(E e)，直接在末尾添加元素，很少调用add(int index, E e)在数组中部添加元素，这样其实移动数组元素就很少发生，因此插入效率也很高。

------

问：删除效率为何说正常情况下ArrayList低，LinkedList高呢？

答：因为删除效率高、低不是绝对的。其实删除操作可以分为两部分。

 第一：找到要删除的元素，这个通过索引找，ArrayList的执行效率要远高于LinkedList的执行效率；通过equals找则需要遍历整个集合，ArrayList和LinkedList执行效率基本一致。

 第二：删除元素及后续操作，这个如果删除是最后一个元素，执行效率基本一致；如果是删除的中间元素，那么ArrayList需进行数组元素移动，而LinkedList只需搭建起该元素的上一个节点和下一个节点的关系即可，LinkedList执行效率高于ArrayList。

 因此，需根据实际情况才可判断实际的执行效率。

------

问：遍历效率这个问题怎么说？

答：ArrayList通过数组实现，天然可以通过数组下标读取数据，顺序遍历、随机遍历效率都非常高；LinkedList通过双向链表实现，顺序遍历时，可直接通过本节点.next()直接找到相关联的下一个节点，效率很高，而如果LinkedList随机遍历时，首先需判断（传递的索引值与集合长度/2）的大小，来确定接下来是应该从第一个节点开始找还是最后节点开始找，越是靠近集合中部、集合越大，随机遍历执行效率越低。

------

ArrayList 也是有顺序的



##三.  Map 体系



![image-20201213193723345](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/19:37:23-image-20201213193723345.png)





![image-20201213193803054](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/19:38:03-image-20201213193803054.png)

###   HashMap 、TreeMap 、LinkedHashMap



一般情况下，使用最多的是 HashMap。
HashMap：在 Map 中插入、删除和定位元素时；
TreeMap：在需要按自然顺序或自定义顺序遍历键的情况下；
LinkedHashMap：在需要输出的顺序和输入的顺序相同的情况下。

- **capacity** - The capacity of this hash set is 8. Meaning, it can store 8 elements.
- **loadFactor** - The load factor of this hash set is 0.6. This means, whenever our hash table is filled by 60%, the elements are moved to a new hash table of double the size of the original hash table.

从衍生时间上进行对比，线程安全、有序性、底层实现，性能上进行对比



## 四. Set 体系

![image-20201213201606159](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/13/20:18:16-20:16:06-image-20201213201606159.png)

### 1. LinkedHashSet和HashSet对比

Both `LinkedHashSet` and `HashSet` implements the `Set` interface. However, there exist some differences between them.

- `LinkedHashSet` maintains a linked list internally. Due to this, it maintains the insertion order of its elements.
- The `LinkedHashSet` class requires more storage than `HashSet`. This is because `LinkedHashSet` maintains linked lists internally.
- The performance of `LinkedHashSet` is slower than `HashSet`. It is because of linked lists present in `LinkedHashSet`.

------

------

### 2. LinkedHashSet和TreeSet对比

Here are the major differences between `LinkedHashSet` and `TreeSet`:

- The `TreeSet` class implements the `SortedSet` interface. That's why elements in a tree set are sorted. However, the `LinkedHashSet` class only maintains the insertion order of its elements.
- A `TreeSet` is usually slower than a `LinkedHashSet`. It is because whenever an element is added to a `TreeSet`, it has to perform the sorting operation.
- `LinkedHashSet` allows the insertion of null values. However, we cannot insert a null value to `TreeSet`.

## 五. 总结

### Java 集合的深度剖析系列文章

[深度剖析ArrayList](https://blog.csdn.net/king14bhhb/article/details/110671464)

[深度剖析LinkedList](https://blog.csdn.net/king14bhhb/article/details/110761550)

[深度剖析Vector](https://blog.csdn.net/king14bhhb/article/details/110881567)

[深度剖析Stack](https://blog.csdn.net/king14bhhb/article/details/111105644)



[深度剖析HashMap](https://blog.csdn.net/king14bhhb/article/details/110294590)

[深度剖析LinkedHashMap](https://blog.csdn.net/king14bhhb/article/details/110294651)

[深度剖析HashTable](https://blog.csdn.net/king14bhhb/article/details/110356606)

[深度剖析TreeMap](https://blog.csdn.net/king14bhhb/article/details/110949085)



[深度剖析HashSet](https://blog.csdn.net/king14bhhb/article/details/110679661) 

[深度剖析LinkedHashSet](https://blog.csdn.net/king14bhhb/article/details/110703105)

[深度剖析TreeSet](https://blog.csdn.net/king14bhhb/article/details/110972115)



[集合工具类Collections深度解析](https://blog.csdn.net/king14bhhb/article/details/110574609)

[集合工具类Arrays深度解析](https://blog.csdn.net/king14bhhb/article/details/110574609)

### Java 集合的实战系列文章






