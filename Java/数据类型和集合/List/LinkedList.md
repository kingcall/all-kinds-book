[TOC]



## 一. LinkedList初始

LinkedList和ArrayList一样是集合List的实现类，虽然较之ArrayList，其使用场景并不多，但同样有用到的时候，那么接下来，我们来认识一下它。**其实它们两个都同属于List阵营，只不过实现方式有所差异，ArrayList 就是借助Array 实现的List,LinkedList 就是借助双向链表(Linked) 实现的List**

首先我们回顾一下ArrayList，ArrayList 底层是数组，添加或者删除都是针对数组上某一个下标上的元素进行操作，就是你记住它的底层实现依赖数组。

LinkedList类是双向列表,列表中的每个节点都包含了对前一个和后一个元素的引用.因为是双向链表作为主要结构，所以这里不再依赖数组了，数据是封装在了一个节点里，而节点与节点之间的关系，是通过节点之间的指针表示的，而在ArrayList中节元素其实更准确的是数据或者是元素本身)并没有被封装成节点，而是**裸露**着的，因为节点与节点之间的关系，是通过数组这个容器的属性表示的，主要就是下标维护了先后顺序。



最终LinkedList 的形态如下所示





![image-20201203162552545](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/16:25:53-image-20201203162552545.png)





### 1. LinkedList 的说明书

在看说明书之前，我们还是先看一下整个类的继承关系



![image-20201206100001020](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/10:00:02-image-20201206100001020.png)



LinkedList继承了AbstractSequentialList类，实现了List接口，AbstractSequentialList中已经实现了很多方法，如get(int index)、set(int index, E element)、add(int index, E element) 和 remove(int index)，这些方法是我们集合操作时使用最多的，不过这些方法在LinkedList中都已经被重写了，而抽象方法在LinkedList中有了具体实现。



接下来我们还是按照国际惯例，先看一下LinkedList 的**说明书** 也就是类注释，让我们对这个类有一个大致的了解

```java
/**
 * Doubly-linked list implementation of the {@code List} and {@code Deque} interfaces.  Implements all optional list operations, and permits all elements (including {@code null}).
 * 双向链表List（这里已经告诉你它的实现是 Doubly-linked） 实现了 List 和 Deque接口，实现了list 接口的所有操作，并且允许所有的值，包括null 
 * All of the operations perform as could be expected for a doubly-linked list.  Operations that index into the list will traverse the list from the beginning or the end, whichever is closer to the specified index.
 * 它的所有操作都符合对双向链表的预期 ,关于index(下标，或者是索引)的操作都会从头或者从尾部遍历整个链表，直到遍历到这个下标
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access a linked list concurrently, and at least
 * one of the threads modifies the list structurally, it <i>must</i> be
 * synchronized externally.  (A structural modification is any operation
 * that adds or deletes one or more elements; merely setting the value of
 * an element is not a structural modification.)  This is typically
 * accomplished by synchronizing on some object that naturally
 * encapsulates the list.
 * 上面这一段就是将它不是线程安全的，前面我们也翻译了好几次了，这里就不翻译了，简单说一下就是你如果需要多线程环境使用，就需要在对它操作之前通过对某一个对象加锁来控制对它的访问
 * If no such object exists, the list should be "wrapped" using the
 * {@link Collections#synchronizedList Collections.synchronizedList}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the list:<pre> List list = Collections.synchronizedList(new LinkedList(...));</pre>
 * 这一段接的是上一段，如果你没有这样可加锁的对象，你可以使用LinkedList 的包装类对象，也就是调用List list = Collections.synchronizedList(new LinkedList(...))来创建包装类对象，具体翻译可以看前面的关于集合的文章
 * <p>The iterators returned by this class's {@code iterator} and
 * {@code listIterator} methods are <i>fail-fast</i>: if the list is
 * structurally modified at any time after the iterator is created, in
 * any way except through the Iterator's own {@code remove} or
 * {@code add} methods, the iterator will throw a {@link
 * ConcurrentModificationException}.  Thus, in the face of concurrent
 * modification, the iterator fails quickly and cleanly, rather than
 * risking arbitrary, non-deterministic behavior at an undetermined
 * time in the future.
 * 这一段讲的实快速失败的特性，以及如何避免，你也可以看前面的文章找到其准确的翻译
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw {@code ConcurrentModificationException} on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness:   <i>the fail-fast behavior of iterators
 * should be used only to detect bugs.</i>
 * @author  Josh Bloch
 * @see     List
 * @see     ArrayList
 * @since 1.2
 * @param <E> the type of elements held in this collection
 */

public class LinkedList<E>  extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
... ...
}
```



### 2. LinkedList 的内部构成

#### size

```java
 transient int size = 0;
```

表示的是LinkedList 存储的元素的多少，或者说是集合的长度

#### Node

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```



LinkedList是通过双向链表实现的，而双向链表就是通过Node类来体现的，通过next变量指向下一个节点，通过prev变量指向上一个节点

LinkedList 对数据的封装，就是说把数据封装成Node 对象，类中通过item变量保存了当前节点的值，而ArrayList 就是存储的真实数据(数据原来的形态)，没有封装

> 所以可以看出，ArrayList 对空间的消耗是要低于LinkedList，因为LinkedList要维护前后指针，还要对数据进行封装

#### first last

```java
/**
  * Pointer to first node.
  */
 transient Node<E> first;

 /**
  * Pointer to last node.
  */
 transient Node<E> last;
```

因为是双向链表，所以这里指的是双向链表的头尾节点



#### node 方法

其实就是返回特定位置的节点，但是这个方法的命名不好，没能做到见名知意个人觉得应该叫**nodeAtIndex()**，之所以将这个方法，单独拿出来是因为下面多次用到了，先在这里看一下，后面就不解释了

```java
/**
 * Returns the (non-null) Node at the specified element index. 其实就是返回特定位置的节点，链表中的节点，所以不可能为空
 */
Node<E> node(int index) {
  	// 这里进判断是从头开始遍历还是从尾部开始遍历，说明书里依然提到了这一点，如果index<size/2 则从头遍历，否则从尾部开始遍历
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```



### 3. LinkedList 的构造方法

因为底层不依赖数组的支持，所以LinkedList 也没有初始容量的这一说法

#### 无参构造

```java
/**
 * Constructs an empty list. 创建一个空的
 */
public LinkedList() {
}
```



#### 基于集合的构造

```java
/**
 * Constructs a list containing the elements of the specified collection, in the order they are returned by the collection's iterator.
 * 创建一个包含特定集合全部元素的list ,元素的顺序和特定集合的遍历顺序一致
 * @param  c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```



## 三. LinkedList常用方法



### 1. add(E e) 

大家都在说LinkedList插入、删除操作效率比较高，以stringList.add(“猪八戒”)为例来看到底发生了什么？

```java
@Test
public void add() {
    LinkedList xiyouList = new LinkedList();
    xiyouList.add("猪八戒");
}
```

在LinkedList中我们找到add(E e)方法的源码

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

/**
 * 设置元素e为最后一个元素，这个方法在添加元素到指定位置的时候也会被调用
*/
void linkLast(E e) {
    final Node<E> l = last;
  	// 添加e 到list 的尾部，那么此时的last 则成为e 的前置，那么e 的后置则是null
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
      	// last 为null 意味着是空表，此时e(newNode) 则成为first
        first = newNode;
    else
      	// 否则 e(newNode) 则成为 last 的后置
        l.next = newNode;
  	// 记录当前元素数的多少和记录修改
    size++;
    modCount++;
}
```

针对上面的if-else 两种情况，我们下面分别梳理一下：

**情况1**：假如xiyouList为空，那么添加进来的node就是first，也是last，这个node的prev和next都为null;

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:03:16-1677914-20190629172301085-1570355202.png)

**情况2**：假如xiyouList不为空，那么添加进来的node就是last，node的prev指向以前的最后一个元素，node的next为null；同时以前的最后一个元素的next.

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:03:16-1677914-20190629172320084-727843647.png)



### 2. add(int index, E element)

接下来我们看一下add 的一个变体方法  add(int index, E element)，例如xiyouList(1, “猪八戒”)

```java
@Test
public void add() {
    LinkedList xiyou = new LinkedList();
    xiyou.add(1,"猪八戒");
}
```

通过跟踪源代码，可以看到它调用了这个方法

```java
/**
 * Inserts the specified element at the specified position in this list. Shifts the element currently at that position (if any) and any subsequent elements to the right (adds one to their indices).
 * 在此列表的指定位置插入指定的元素。将当前位于该位置的元素（如果有）和任何后续元素向右移动，然后将要插入的元素进行插入
 * @param index index at which the specified element is to be inserted
 * @param element element to be inserted
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
//在指定位置添加一个元素
public void add(int index, E element) {
  	// 检测下标的合法性
    checkPositionIndex(index);
  	// 下标合法之后，则判断是不是添加到集合尾部，index=size 的时候则是添加到集合的尾部
    if (index == size)
        linkLast(element);
  	// 否则则是添加到集合的index 位置(index 不是最后一个位置)
    else
      // 因为不是添加到最后，所以这个里可以得到这个位置当前的元素，node(index) 
        linkBefore(element, node(index));
}
```



#### checkPositionIndex

我们先看一下，这个检测下标合法性的方法

```java
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
      	// 这个就是如果下标不合法的时候抛出的异常
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
/**
 * Tells if the argument is the index of a valid position for an iterator or an add operation.
 * 在调用add或者 遍历 方法时候，判断一个下标是否合法
 */
private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}
```



#### linkLast

这个时候，其实和add(E element) 方法一样了，因为index=size ,本质上就是添加到链表末尾，前面已经解释过这个方法的代码了，这里就不解释了

```java
/**
 * Links e as last element. 将e 作为last 元素连接起来，其实就是添加到链表末尾
 */
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```



#### linkBefore

因为这个时候，已经知道不是添加在last 的位置了，所以一定是在非空节点之前添加的

```java
/**
 * 在一个非空节点前插入一个元素，succ 是传入的当前位置上已经存在的元素，接下来要做的就是将e 和 succ 连接起来
 */
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
  	// succ 的前置
    final Node<E> pred = succ.prev;
  	// succ 作为e 的后置，succ 的前置作为e 的前置，其实就是将  succ_pree <-> succ <-> succ_next  变成  succ_pree  <-> e <->succ <-> succ_next
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
  	// 如果pree是null ,则e 要代替succ 成为新的first
    if (pred == null)
        first = newNode;
    else
      	// 否则的话，e 成为succ 前置的后置
        pred.next = newNode;
  	// 记录修改
    size++;
    modCount++;
}
```



其实从代码中看到和add(E e)的代码实现没有本质区别，都是通过新建一个Node实体，同时指定其prev和next来实现，**不同点在于需要调用node(int index)通过传入的index来定位到要插入的位置，这个也是比较耗时的**

其实看到这里，大家也都明白了。

> LinkedList插入效率高是相对的，因为它省去了ArrayList插入数据可能的数组扩容和数据元素移动时所造成的开销，但数据扩容和数据元素移动却并不是时时刻刻都在发生的。



### 3. get(int index)

我们知道随机读取元素不是LinkedList所擅长的，读取效率比起ArrayList也低得多，那么我们来看一下为什么，其实开始之前我们大概都能猜到为什么了，因为我们在类的说明书里，已经解释了一部分了因为它的本质还是遍历操作，其实就是一个O(n) 的操作而不是一个O(1) 的操作

下面我们还是具体看一下

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

checkElementIndex 方法就是在检测下标是否合法，而node 方法前面我们已经说过了，就是返回特定位置的节点，所以下面我们看一下具体是怎么检测的

```java
private void checkElementIndex(int index) {
    if (!isElementIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
 * Tells if the argument is the index of an existing element.
 */
private boolean isElementIndex(int index) {
    return index >= 0 && index < size;
}
```

其实就是限定了下标的范围，其实如果你仔细看了add 那一部分的代码的话，你会发现这一部分代码和上面的非常相似，唯一的区别在于目的上，一个是检测插入的位置是否合法，而这个方法是检测获取的位置时是否合法，所以叫isElementIndex而不是叫isPositionIndex，因为目的的不同导致了条件的不同，获取时候index必须小于size,因为size 的地方是空，但是插入的时候则就可以



从上述代码中我们可以看到get(int index)方法是通过node(int index)来实现的，而node它的实现机制是：

比较传入的索引参数index与集合长度size/2，如果是index小，那么从第一个顺序循环，直到找到为止；如果index大，那么从最后一个倒序循环，直到找到为止。也就是说越靠近中间的元素，调用get(int index方法遍历的次数越多，效率也就越低，而且随着集合的越来越大，get(int index)执行性能也会指数级降低。

因此在使用LinkedList的时候，我们不建议使用这种方式读取数据，可以使用**getFirst()，getLast()**方法，将直接用到类中的first和last变量。

### 4. removeFirst()和removeLast()

这里removeFirst()和removeLast()非常相似，会用到类中定义的first和last变量，所以这里我们只说removeFirst，大家可以自行去看removeLast

```java
/**
 * Removes and returns the first element from this list. 移除list 中的第一个元素
 *
 * @return the first element from this list
 * @throws NoSuchElementException if this list is empty
 */
public E removeFirst() {
    final Node<E> f = first;
  	// 如果f是null 的话，则抛出异常，这里个人觉得不应该抛出异常，直接返回null 就行了，否则你每次调用这个方法的时候还得判断一下是否是空list
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

/**
 * Unlinks non-null first node f. 取消first 的连接
 */
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
   // help GC，为啥f 不直接赋值成null 呢
    f.item = null;
    f.next = null;
    first = next;
    if (next == null)
        last = null;
    else
      // 因为next 已经成为first 了，应该是没有前置了
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```



### 5. remove(Object o) 和 remove(int index)

我们看一下remove(Object o) 和 remove(int index)源码

```java
//删除某个对象
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
//删除某个位置的元素
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
//删除某节点，并将该节点的上一个节点（如果有）和下一个节点（如果有）关联起来
E unlink(Node<E> x) {
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

其实实现都非常简单，先找到要删除的节点，remove(Object o)方法遍历整个集合，通过 == 或 equals方法进行判断；remove(int index)通过node(index)方法。

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:03:16-1677914-20190629172337033-933154031.png)

### 6. LinkedList遍历性能对比

我们主要列举一下三种常用的遍历方式，普通for循环，增强for循环，Iterator迭代器，当然遍历中海油一个问题就是快速失败的问题，但是这里就不再举例子了，因为前面文章已经写了好多了，大家可以自行尝试，也可以参考前面其他集合的文章

```java
public static void main(String[] args) {
    LinkedList<Integer> list = getLinkedList();
    //通过快速随机访问遍历LinkedList
    listByNormalFor(list);
    //通过增强for循环遍历LinkedList
    listByStrengThenFor(list);
    //通过快迭代器遍历LinkedList
    listByIterator(list);
}

/**
 * 构建一个LinkedList集合,包含元素50000个
 * @return
 */
private static LinkedList<Integer> getLinkedList() {
    LinkedList list = new LinkedList();
    for (int i = 0; i < 50000; i++){
        list.add(i);
    }
    return list;
}

/**
 * 通过快速随机访问遍历LinkedList
 */
private static void listByNormalFor(LinkedList<Integer> list) {
    // 记录开始时间
    long start = System.currentTimeMillis();
    int size = list.size();
    for (int i = 0; i < size; i++) {
        list.get(i);
    }
    // 记录用时
    long interval = System.currentTimeMillis() - start;
    System.out.println("listByNormalFor：" + interval + " ms");
}

/**
 * 通过增强for循环遍历LinkedList
 * @param list
 */
public static void listByStrengThenFor(LinkedList<Integer> list){
    // 记录开始时间
    long start = System.currentTimeMillis();
    for (Integer i : list) { }
    // 记录用时
    long interval = System.currentTimeMillis() - start;
    System.out.println("listByStrengThenFor：" + interval + " ms");
}

/**
 * 通过快迭代器遍历LinkedList
 */
private static void listByIterator(LinkedList<Integer> list) {
    // 记录开始时间
    long start = System.currentTimeMillis();
    for(Iterator iter = list.iterator(); iter.hasNext();) {
        iter.next();
    }
    // 记录用时
    long interval = System.currentTimeMillis() - start;
    System.out.println("listByIterator：" + interval + " ms");
}
```

执行结果如下：

```java
listByNormalFor：1067 ms
listByStrengThenFor：3 ms
listByIterator：2 ms
```

通过普通for循环随机访问的方式执行时间远远大于迭代器访问方式，这个我们可以理解，在前面的get(int index)方法中已经有过说明，那么为什么增强for循环能做到迭代器遍历差不多的效率？

通过反编译工具后得到如下代码

```java
public static void listByStrengThenFor(LinkedList<Integer> list)
  {
    long start = System.currentTimeMillis();
    Integer localInteger;
    for (Iterator localIterator = list.iterator(); localIterator.hasNext(); 
         localInteger = (Integer)localIterator.next()) {}
    long interval = System.currentTimeMillis() - start;
    System.out.println("listByStrengThenFor：" + interval + " ms");
}
```

很明显了，增强for循环遍历时也调用了迭代器Iterator，不过多了一个赋值的过程。

还有类似于pollFirst()，pollLast()取值后删除的方法也能达到部分的遍历效果。



## 四. ArrayList 和 LinkedList 对比

前面我们说到ArrayList 和 LinkedList 都属于List 阵营，都实现了List 接口，但是LinkedList 实现了Deque接口，在很多场景下我们都是使用ArrayList的，而且可以很好的满足我们的需求

### 实现原理

ArrayList 是适应双链表实现的，而LinkedList则是使用一个可以动态扩容的数组实现的，所以在LinkedList里面则没有什么 initial capacity or default capacity 的概念

### 性能

对性能和使用场景的考虑，获取实我们在日常使用中选择的主要因素了，下面我们就存存储、添加、插入、删除的角度谈谈二者

#### 存储

LinkedList 因为每个插入的数据都需要包装，而且还要维护前后指针，而指针也是有存储开销的，所以LinkedList 的存储会比ArrayList 高

#### 插入

LinkedList 的插入性能是比较高的，因为它只需要**打破已有的链表结构，然后重新连接起来**；但是当你要添加到指定位置的时候，就会涉及到遍历，然后确定位置的过程

ArrayList 默认的插入也是插入到尾部的，**但是当要扩容的时候，会涉及到数据的迁移**；**当你插入到指定位置的时候就会涉及到数据迁移**

#### 获取

LinkedList 的读取涉及到链表的遍历，所以性能不高，时间复杂度**O(n)**

ArrayList 会利用数组的特性，直接获取指定下标位置的数据，时间复杂度**O(1)**

#### 删除

LinkedList 删除,无论是下标删除还是根据元素删除，都会涉及到链表遍历，所以平均时间复杂度**O(n)**

ArrayList 删除会涉及到到元素的迁移，所以平均时间复杂度也是**O(n)**



## 四. 总结

前面我们学习了ArrayList 和 LinkedList，分别研究了他们的原理、底层实现、和方法使用、以及性能对比

### 你觉得LinkedList 还有什么可以改进的地方吗，欢迎讨论

和上一节一样这里我依然给出这个思考题，虽然我们的说法可能不对，可能我们永远也站不到源代码作者当年的高度，但是我们依然积极思考，大胆讨论

虽然java 源代码的山很高，如果你想跨越，至少你得有登山的勇气，这里我给出自己的一点点愚见，希望各位不吝指教

**node(index) 这个方法的命名太不符合规范了**

**下面这段代码可以重构，相似度太高了，而且命名也不怎么清楚**

```
private void checkElementIndex(int index) {
    if (!isElementIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
 * Tells if the argument is the index of a valid position for an
 * iterator or an add operation.
 */
private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}

/**
 * Tells if the argument is the index of an existing element.
 */
private boolean isElementIndex(int index) {
    return index >= 0 && index < size;
}

```

**像下面这个remove 方法我觉得不应该抛出异常，而是应该返回null 值就可以了，客户端可以更具是否为null 进行业务处理，否则的话我认为你的迭代方法也应该抛出异常**

```
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
```

