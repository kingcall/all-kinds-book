[TOC]

## LinkedDeque

前面我们学习了Queue，在Java 中的实现其实就是LinkedList,我们的使用方式就是`Queue<Integer> numbers = new LinkedList<>(); `通过声明一个Queue接口窄化了对LinkedList的方法的访问权限，使得对象的使用更像是一个队列而不是一个原生的LinkedList

![image-20201206100001020](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/10:00:02-image-20201206100001020-20210120133444565.png)

我们发现LinkedList可以当做Queue使用那是因为LinkedList 实现了Queue接口，但是我们从上面的实现中注意到另外一个问题，那就是LinkedList不是直接实现了Queue接口，而是通过Deque 实现的，而Deque就是我们今天的主角——双向队列,因为是通过LinkedList实现的，所有我们可以将其称之为LinkedDeque

### Deque 的定义

 **Deque一个线性 collection，支持在两端插入和移除元素。**

我们看到List ,Set 和 Queue 都是Java 集合框架的顶级接口，虽然我们今天学习的Deque 是基于LinkedList 实现的，但是为了方便理解和梳理整个集合框架我们还是将其划分到Queue 家族，LinkedList实现了Deque接 口，Deque接口窄化了对LinkedList的方法的访问权限（即在方法中的参数类型如果是Deque时，就完全只能访问Deque接口所定义的方法 了，而不能直接访问 LinkedList的非Deque的方法），以使得只有恰当的方法才可以使用。





![image-20201213140854528](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/09:03:30-42f0ce9d810524544f90c5fbfa822cf1-20210120133052457.png)

接下来我们看一下 Deque 接口的定义

```java
public interface Deque<E> extends Queue<E> {
	 // *** Deque 特有的方法 ***
    void addFirst(E e);
    void addLast(E e);
    boolean offerFirst(E e);
    boolean offerLast(E e);
    E removeFirst();
    E removeLast();
    E pollFirst();
    E pollLast();
    E getFirst();
    E getLast();
    E peekFirst();
    E peekLast();
    boolean removeFirstOccurrence(Object o);
    boolean removeLastOccurrence(Object o);

    // *** Queue 的方法 ***
    boolean add(E e);
    boolean offer(E e);
    E remove();
    E poll();
    E element();
    E peek();

    // *** Collection 的方法 ***
    boolean remove(Object o);
    boolean contains(Object o);
    public int size();
    Iterator<E> iterator();
    Iterator<E> descendingIterator();

}
```

这个接口的定义其实很能说明问题，能说明什么问题呢，一个是Deque完全支持Queue的功能，其次就是由于Deque是双向队列，所以提供的方法都是成对的(xxxFirst,xxxLast)这些方法都是在队列的对首和对尾操作。

### 使用

因为底层是通过LinkedList 来实现的，前面我们又对LinkedList进行了非常详细的学习，所以这里我们就不探究原理什么的了，直接看一下怎么使用

```java
public static void main(String[] args) {
    // Creating Deque using the LinkedList class
    Deque<Integer> numbers = new LinkedList<>();

    // offer elements to the Queue
    numbers.offer(1);
    numbers.offer(2);
    numbers.offer(3);

    // use Deque methods
    numbers.addFirst(0);
    numbers.addLast(4);

    System.out.println("Queue: " + numbers);

    // Access elements of the Deque
    int accessedNumber = numbers.getFirst();
    System.out.println("Accessed First element: " + accessedNumber);
    accessedNumber = numbers.getLast();
    System.out.println("Accessed Last element: " + accessedNumber);

    // Remove elements from the Queue
    int removedNumber = numbers.removeFirst();
    System.out.println("Removed First Element: " + removedNumber);
    removedNumber = numbers.removeLast();
    System.out.println("Removed Last Element: " + removedNumber);


    System.out.println("Updated Queue: " + numbers);
}
```

输出

```
Queue: [0, 1, 2, 3, 4]
Accessed First element: 0
Accessed Last element: 4
Removed First Element: 0
Removed Last Element: 4
Updated Queue: [1, 2, 3]
```

## 总结

1. 这一节的知识很简单，主要就是想告诉大家Deque的实现方式以及它是怎么实现的，注意和Queue 进行对比。
2. LinkedList 实现的Deque 是一个无界的队列，本质上还是通过对链表的头部和尾部进行操作实现的。