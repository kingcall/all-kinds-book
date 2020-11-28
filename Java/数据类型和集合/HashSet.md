[TOC]

## 一. HashSet概述

HashSet是Java集合Set的一个实现类，Set是一个接口，其实现类除HashSet之外，还有TreeSet，并继承了Collection，HashSet集合很常用，同时也是程序员面试时经常会被问到的知识点，下面是结构图

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/22:59:52-1677914-20190728092246829-963791496.png)

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{}
```

## 二. HashSet构造

HashSet有几个重载的构造方法，我们来看一下

```java
private transient HashMap<E,Object> map;
//默认构造器
public HashSet() {
    map = new HashMap<>();
}
//将传入的集合添加到HashSet的构造器
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
//明确初始容量和装载因子的构造器
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}
//仅明确初始容量的构造器（装载因子默认0.75）
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}
```

通过上面的源码，我们发现了HashSet就TM是一个皮包公司，它就对外接活儿，活儿接到了就直接扔给HashMap处理了。因为底层是通过HashMap实现的，这里简单提一下:

HashMap的数据存储是通过数组+链表/红黑树实现的，存储大概流程是通过hash函数计算在数组中存储的位置，如果该位置已经有值了，判断key是否相同，相同则覆盖，不相同则放到元素对应的链表中，如果链表长度大于8，就转化为红黑树，如果容量不够，则需扩容（注：这只是大致流程）。

如果对HashMap原理不太清楚的话，可以先去了解一下

[HashMap原理(一) 概念和底层架构](https://www.cnblogs.com/LiaHon/p/11142958.html)

[HashMap原理(二) 扩容机制及存取原理](https://www.cnblogs.com/LiaHon/p/11149644.html)

## 三. add方法

HashSet的add方法时通过HashMap的put方法实现的，不过HashMap是key-value键值对，而HashSet是集合，那么是怎么存储的呢，我们看一下源码

```java
private static final Object PRESENT = new Object();

public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

看源码我们知道，HashSet添加的元素是存放在HashMap的key位置上，而value取了默认常量PRESENT，是一个空对象，至于map的put方法，大家可以看[HashMap原理(二) 扩容机制及存取原理](https://www.cnblogs.com/LiaHon/p/11149644.html)。

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/22:59:52-1677914-20190728092314289-404893813.png)

## 四. remove方法

HashSet的remove方法通过HashMap的remove方法来实现

```java
//HashSet的remove方法
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
//map的remove方法
public V remove(Object key) {
    Node<K,V> e;
    //通过hash(key)找到元素在数组中的位置，再调用removeNode方法删除
    return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
}
/**
 * 
 */
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    //步骤1.需要先找到key所对应Node的准确位置，首先通过(n - 1) & hash找到数组对应位置上的第一个node
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        //1.1 如果这个node刚好key值相同，运气好，找到了
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        /**
         * 1.2 运气不好，在数组中找到的Node虽然hash相同了，但key值不同，很明显不对， 我们需要遍历继续
         *     往下找；
         */
        else if ((e = p.next) != null) {
            //1.2.1 如果是TreeNode类型，说明HashMap当前是通过数组+红黑树来实现存储的，遍历红黑树找到对应node
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                //1.2.2 如果是链表，遍历链表找到对应node
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        //通过前面的步骤1找到了对应的Node,现在我们就需要删除它了
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            /**
             * 如果是TreeNode类型，删除方法是通过红黑树节点删除实现的，具体可以参考【TreeMap原理实现
             * 及常用方法】
             */
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            /** 
             * 如果是链表的情况，当找到的节点就是数组hash位置的第一个元素，那么该元素删除后，直接将数组
             * 第一个位置的引用指向链表的下一个即可
             */
            else if (node == p)
                tab[index] = node.next;
            /**
             * 如果找到的本来就是链表上的节点，也简单，将待删除节点的上一个节点的next指向待删除节点的
             * next,隔离开待删除节点即可
             */
            else
                p.next = node.next;
            ++modCount;
            --size;
            //删除后可能存在存储结构的调整，可参考【LinkedHashMap如何保证顺序性】中remove方法
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

removeTreeNode方法具体实现可参考 TreeMap原理实现及常用方法

afterNodeRemoval方法具体实现可参考[LinkedHashMap如何保证顺序性](https://www.cnblogs.com/LiaHon/p/11180869.html)

## 五. 遍历

HashSet作为集合，有多种遍历方法，如普通for循环，增强for循环，迭代器，我们通过迭代器遍历来看一下

```java
public static void main(String[] args) {
    HashSet<String> setString = new HashSet<> ();
    setString.add("星期一");
    setString.add("星期二");
    setString.add("星期三");
    setString.add("星期四");
    setString.add("星期五");

    Iterator it = setString.iterator();
    while (it.hasNext()) {
        System.out.println(it.next());
    }
}
```

打印出来的结果如何呢？

```java
星期二
星期三
星期四
星期五
星期一
```

意料之中吧，HashSet是通过HashMap来实现的，HashMap通过hash(key)来确定存储的位置，是不具备存储顺序性的，因此HashSet遍历出的元素也并非按照插入的顺序。

## 六. 合计合计

按照我前面的规划，应该每一块主要的内容都单独写一下，如集合ArrayList，LinkedList，HashMap，TreeMap等。不过我在写这篇关于HashSet的文章时，发现有前面对HashMap的讲解后，确实简单，HashSet就是一个皮包公司，在HashMap外面加了一个壳，那么LinkedHashSet是否就是在LinkedHashMap外面加了一个壳呢，而TreeSet是否是在TreeMap外面加了一个壳？我们来验证一下

### 先看一下LinkedHashSet

最开始的结构图已经提到了LinkedHashSet是HashSet的子类，我们来看源码

```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable 
{
    
 	public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }

    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }

    public LinkedHashSet() {
        super(16, .75f, true);
    }

    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }

    public Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.DISTINCT | Spliterator.ORDERED);
    }
}
```

上面就是LinkedHashSet的所有代码了，是不是感觉智商被否定了，这基本上没啥东西嘛，构造器还全部调用父类的，下面就是其父类HashSet的对此的构造方法

```java
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

大家也看出来，和我们的猜测一样，没有深究下去的必要了。如果有兴趣可以看看[LinkedHashMap如何保证顺序性](https://www.cnblogs.com/LiaHon/p/11180869.html)



## 七. 总结

本来想三章的内容，一章就算完了，虽然Set实现有点赖皮，毕竟他祖辈是Collection而不是Map，在Map的实现类上穿了一层衣服就成了Set，然后出于某种目的埋伏在Collection中，哈哈，开个玩笑，本文主要介绍了HashSet的原理以及主要方法，同时简单介绍了LinkedHashSet和TreeSet，若有不对之处，请批评指正，望共同进步，谢谢！

