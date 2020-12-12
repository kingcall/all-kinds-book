[TOC]

## 一. TreeSet 初识

其实学了前面几节[深度剖析HashMap](https://blog.csdn.net/king14bhhb/article/details/110294590)、[深度剖析LinkedHashMap](https://blog.csdn.net/king14bhhb/article/details/110294651)、[深度剖析HashSet](https://blog.csdn.net/king14bhhb/article/details/110679661) [深度剖析LinkedHashSet ](https://blog.csdn.net/king14bhhb/article/details/110703105) 到这里我们大概否能猜到这个数据结构是什么样了，LinkedHashMap 对HashMap 增加了双向链表维持了顺序，HashSet使用了HashMap弱化了Key-Value 中的Value 实现了去重且只有value集合的功能,LinkedHashSet 则是使用了LinkedHashMap，那么TreeSet 呢，也是使用了TreeMap 吗，其实就是的

因为前面我们已经详细解释过[TreeMap](https://blog.csdn.net/king14bhhb/article/details/110949085) 所以这篇我们主要探究一下原理和简单使用即可，不过多解释



### TreeSet 说明书

在此之前我们先看一下TreeSet 继承关系，先对它有一个直观的认识



![image-20201210125039785](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/10/12:50:49-image-20201210125039785.png)

```java
/**
 * A {@link NavigableSet} implementation based on a {@link TreeMap}.
 * The elements are ordered using their {@linkplain Comparable natural
 * ordering}, or by a {@link Comparator} provided at set creation
 * time, depending on which constructor is used.
 * 一个NavigableSet 的基于TreeMap的实现，所有的元素要么是基于Comparable的自然排序，要么是基于在创建时候提供的Comparator，
 * 当然这是要看创建的时候选择了那个构造方法
 * <p>This implementation provides guaranteed log(n) time cost for the basic
 * operations ({@code add}, {@code remove} and {@code contains}).
 * 这个实现保证了对于add 、remove、contains方法 log(n)的时间复杂度
 * <p>Note that the ordering maintained by a set (whether or not an explicit
 * comparator is provided) must be <i>consistent with equals</i> if it is to
 * correctly implement the {@code Set} interface.  (See {@code Comparable}
 * or {@code Comparator} for a precise definition of <i>consistent with
 * equals</i>.)  This is so because the {@code Set} interface is defined in
 * terms of the {@code equals} operation, but a {@code TreeSet} instance
 * performs all element comparisons using its {@code compareTo} (or
 * {@code compare}) method, so two elements that are deemed equal by this method
 * are, from the standpoint of the set, equal.  The behavior of a set
 * <i>is</i> well-defined even if its ordering is inconsistent with equals; it
 * just fails to obey the general contract of the {@code Set} interface.
 * 如果它正确的实现了Set接口，那么它维持的排序应该和equals保持一致，这是因为Set接口是在equals操作下定义的
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access a tree set concurrently, and at least one
 * of the threads modifies the set, it <i>must</i> be synchronized
 * externally.  This is typically accomplished by synchronizing on some
 * object that naturally encapsulates the set.
 * If no such object exists, the set should be "wrapped" using the
 * {@link Collections#synchronizedSortedSet Collections.synchronizedSortedSet}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the set: <pre>
 *   SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));</pre>
 * 重点注意的实这个实现不是线程同步的，如果有多个线程访问它，至少一个线程修改它，这种情况下需要被同步
 * 一个典型的实现就是在set 操作的外面通过一个对象进行同步，如果不存在这样的对象，那么set 需要被包装，
 * 可以使用SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));这个 操作最好是当set创建的时候就操作
 * 防止意外的非同步访问
 * <p>The iterators returned by this class's {@code iterator} method are
 * <i>fail-fast</i>: if the set is modified at any time after the iterator is
 * created, in any way except through the iterator's own {@code remove}
 * method, the iterator will throw a {@link ConcurrentModificationException}.
 * Thus, in the face of concurrent modification, the iterator fails quickly
 * and cleanly, rather than risking arbitrary, non-deterministic behavior at an undetermined time in the future.
 * iterator 方法返回的iterators 是快速失败的，如果set 在创建之后的任何时间除了iterator自己的remove 方法之外set 被修改了，则会抛出异常
 * 面临并发修改，的迭代器快速而干净地失败，而不是在将来某个不确定的时间有着随机的、不确定的行为的风险。
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw {@code ConcurrentModificationException} on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness:   <i>the fail-fast behavior of iterators
 * should be used only to detect bugs.</i>
 * 需要注意的是，迭代器的快速失效行为无法得到保证， 一般来说，无法有任何严格的保证在非同步的并发修改的场景下，快速失败抛出ConcurrentModificationException异常是尽最大努力了
 * 因此，编写为了保证自己的正确性而依赖于fail-fast异常的程序是不正确的，fail-fast 的行为应该仅用于检测bug
 * 
 */

public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{ ... ...}
```

 

### TreeSet 构造函数

TreeSet 主要有下面五种构造函数

![image-20201210125729515](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/10/12:57:30-image-20201210125729515.png)





#### 无参构造

```java
/**
 * Constructs a new, empty tree set, sorted according to the
 * natural ordering of its elements.  
 * 创建一个空的tree set，按照元素自己的自然顺序排序
 * All elements inserted into the set must implement the {@link Comparable} interface.
 * 所有插入set 中的元素必须实现Comparable 接口
 * Furthermore, all such elements must be <i>mutually comparable</i>: 
 * 除此之外，所有的元素必须是可相互比较的(这里的意思是类型是一致的)
 * {@code e1.compareTo(e2)} must not throw a  {@code ClassCastException} for any elements {@code e1} and
 * {@code e2} in the set.  
 * If the user attempts to add an element
 * to the set that violates this constraint (for example, the user
 * attempts to add a string element to a set whose elements are
 * integers), the {@code add} call will throw a @code ClassCastException}.
 * 当你尝试添加类型不匹配的元素的时候。add 方法则会抛出异常
 */
public TreeSet() {
    this(new TreeMap<E,Object>());
}
```



#### 指定Comparator

```java
/**
 * Constructs a new, empty tree set, sorted according to the specified
 * comparator. 
 * 建一个空的tree set，元素按照comparator进行排序
 * All elements inserted into the set must be <i>mutually comparable</i> by the specified comparator: {@code comparator.compare(e1, e2)} must not throw a {@code ClassCastException} for any elements
 * 所有插入的元素都必须是可比较的，也就是说comparator.compare(e1, e2)方法不能抛出ClassCastException异常
 * {@code e1} and {@code e2} in the set.  If the user attempts to add
 * an element to the set that violates this constraint, the
 * {@code add} call will throw a {@code ClassCastException}.
 *
 * @param comparator the comparator that will be used to order this set. If {@code null}, the {@linkplain Comparable natural
 * ordering} of the elements will be used. 如果没有提供comparator，那么元素就会按照自己的自然排序进行排序
 */
public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}
```

下面我给一个例子

```java
class Main {
    public static void main(String[] args) {

        // Creating a tree set with customized comparator
        TreeSet<String> animals = new TreeSet<>(new CustomComparator());

        animals.add("Dog");
        animals.add("Zebra");
        animals.add("Cat");
        animals.add("Horse");
        System.out.println("TreeSet: " + animals);
    }

    // Creating a comparator class
    public static class CustomComparator implements Comparator<String> {
        @Override
        public int compare(String animal1, String animal2) {
            int value =  animal1.compareTo(animal2);

            // elements are sorted in reverse order
            if (value > 0) {
                return -1;
            }
            else if (value < 0) {
                return 1;
            }
            else {
                return 0;
            }
        }
    }
}
```

输出结果

```
TreeSet: [Zebra, Horse, Dog, Cat]
```



#### 指定集合

```java
/**
 * Constructs a new tree set containing the elements in the specified
 * collection, sorted according to the <i>natural ordering</i> of its
 * elements.  All elements inserted into the set must implement the
 * {@link Comparable} interface.  Furthermore, all such elements must be
 * <i>mutually comparable</i>: {@code e1.compareTo(e2)} must not throw a
 * {@code ClassCastException} for any elements {@code e1} and
 * {@code e2} in the set.
 * 创建一个新的包含指定集合中全部元素的集合，所有插入集合中的元素必须实现Comparable接口，除此之外，所有的元素要可相互比较
 * @param c collection whose elements will comprise the new set
 * @throws ClassCastException if the elements in {@code c} are
 *         not {@link Comparable}, or are not mutually comparable
 * @throws NullPointerException if the specified collection is null
 */
public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c);
}
```



#### 指定有序集合

```java
/**
 * Constructs a new tree set containing the same elements and using the same ordering as the specified sorted set.
 * 创建一个新的包含指定集合中全部元素的集合，并且使用和该集合一样的排序
 * @param s sorted set whose elements will comprise the new set
 * @throws NullPointerException if the specified sorted set is null
 */
public TreeSet(SortedSet<E> s) {
  	// 可以看出这里其实是使用了该有序集合的排序器
    this(s.comparator());
    addAll(s);
}
```

## 二. TreeSet 的常用方法

### `add()` 和 `addAll()`

add 是添加一个元素到集合而addAll 则是添加一个指定集合的全部元素到set

```java
import java.util.TreeSet;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> evenNumbers = new TreeSet<>();
        // Using the add() method
        evenNumbers.add(2);
        evenNumbers.add(4);
        evenNumbers.add(6);
        System.out.println("TreeSet: " + evenNumbers);

        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(1);

        // Using the addAll() method
        numbers.addAll(evenNumbers);
        System.out.println("New TreeSet: " + numbers);
    }
}
```

**Output**

```
TreeSet: [2, 4, 6]
New TreeSet: [1, 2, 4, 6]
```

------

### 获取元素(遍历) 

因为是set 集合，所以获取元素的方法也主要是遍历



```java
import java.util.TreeSet;
import java.util.Iterator;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(2);
        numbers.add(5);
        numbers.add(6);
        System.out.println("TreeSet: " + numbers);

        // Calling iterator() method
        Iterator<Integer> iterate = numbers.iterator();
        System.out.print("TreeSet using Iterator: ");
        // Accessing elements
        while(iterate.hasNext()) {
            System.out.print(iterate.next());
            System.out.print(", ");
        }
    }
}
```

**Output**

```
TreeSet: [2, 5, 6]
TreeSet using Iterator: 2, 5, 6,
```



### `remove()` 和`removeAll()`

和add以及addAll 对应的实remove 和 removeAll ，分别是删除一个元素和删除指定集合里的全部元素,需要注意的实删除是有返回值的

```java
import java.util.TreeSet;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(2);
        numbers.add(5);
        numbers.add(6);
        System.out.println("TreeSet: " + numbers);

        // Using the remove() method
        boolean value1 = numbers.remove(5);
        System.out.println("Is 5 removed? " + value1);

        // Using the removeAll() method
        boolean value2 = numbers.removeAll(numbers);
        System.out.println("Are all elements removed? " + value2);
    }
}
```

**Output**

```
TreeSet: [2, 5, 6]
Is 5 removed? true
Are all elements removed? true
```



## 三. Navigation 接口

因为 `TreeSet` 实现了`NavigableSet` 接口，它提供了各种方法来查看或者操作Tree Set 的元素。

### 1. first() 和 last() 方法

`first()` 返回集合中的第一个元素，`last()` 返回集合中的最后一个元素,当然也提供了对应的获取后删除的版本，那就是pollFirst和pollLast

```java
import java.util.TreeSet;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(2);
        numbers.add(5);
        numbers.add(6);
        System.out.println("TreeSet: " + numbers);

        // Using the first() method
        int first = numbers.first();
        System.out.println("First Number: " + first);

        // Using the last() method
        int last = numbers.last();
        System.out.println("Last Number: " + last);
    }
}
```

**Output**

```
TreeSet: [2, 5, 6]
First Number: 2
Last Number: 6
```

接下来我们看一下 pollFirst和pollLast

```java
public static void main(String[] args) {
    TreeSet<Integer> numbers = new TreeSet<>();
    numbers.add(2);
    numbers.add(5);
    numbers.add(6);
    System.out.println("TreeSet: " + numbers);

    // Using the first() method
    int first = numbers.pollFirst();
    System.out.println("First Number: " + first);

    first = numbers.first();
    System.out.println("First Number: " + first);

}
```

**Output**

```
TreeSet: [2, 5, 6]
First Number: 2
First Number: 5
```



### 2. ceiling(), floor(), higher() and lower()方法

- **higher(element)** - 返回大于参数的最小元素(其实就是大于)
- **lower(element)** -  返回小于参数的最大元素
- **ceiling(element)** - 返回大于参数的最小元素，如果参数在集合中存在存在则返回该参数(其实就是大于等于)
- **floor(element) **   - 返回小于参数的最大元素，如果参数在集合中存在存在则返回该参数(其实就是小于等于)

```java
import java.util.TreeSet;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(2);
        numbers.add(5);
        numbers.add(4);
        numbers.add(6);
        System.out.println("TreeSet: " + numbers);

        // Using higher()
        System.out.println("Using higher: " + numbers.higher(4));

        // Using lower()
        System.out.println("Using lower: " + numbers.lower(4));

        // Using ceiling()
        System.out.println("Using ceiling: " + numbers.ceiling(4));

        // Using floor()
        System.out.println("Using floor: " + numbers.floor(3));

    }
}
```

**Output**

```
TreeSet: [2, 4, 5, 6]
Using higher: 5
Using lower: 2
Using ceiling: 4
Using floor: 2
```



------

### 3. headSet(), tailSet() and subSet() Methods

------

#### headSet(element, booleanValue)

`headSet()` 方法返回TreeSet 中指定元素之前的全部元素，booleanValue 是可选的，如果是True 则返回包含该元素之前的元素，否则只返回该元素之前的元素

可以将其认为是**小于和小于等于**或者是**大于和大于等于**

```java
import java.util.TreeSet;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(2);
        numbers.add(5);
        numbers.add(4);
        numbers.add(6);
        System.out.println("TreeSet: " + numbers);

        // Using headSet() with default boolean value
        System.out.println("Using headSet without boolean value: " + numbers.headSet(5));

        // Using headSet() with specified boolean value
        System.out.println("Using headSet with boolean value: " + numbers.headSet(5, true));
    }
}
```

**Output**

```
TreeSet: [2, 4, 5, 6]
Using headSet without boolean value: [2, 4]
Using headSet with boolean value: [2, 4, 5]
```



#### tailSet(element, booleanValue)

 `tailSet()` 则返回特定元素之后的集合，booleanValue 的效果还是是否包含该元素的含义



```java
import java.util.TreeSet;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(2);
        numbers.add(5);
        numbers.add(4);
        numbers.add(6);
        System.out.println("TreeSet: " + numbers);

        // Using tailSet() with default boolean value
        System.out.println("Using tailSet without boolean value: " + numbers.tailSet(4));

        // Using tailSet() with specified boolean value
        System.out.println("Using tailSet with boolean value: " + numbers.tailSet(4, false));
    }
}
```

**Output**

```
TreeSet: [2, 4, 5, 6]
Using tailSet without boolean value: [4, 5, 6]
Using tailSet with boolean value: [5, 6]
```

------

#### subSet(e1, bv1, e2, bv2)

 `subSet()`  方法返回e1 和 e2 之间的元素，bv1 和 bv2 是两个可选参数，bv1 默认是true,bv2 默认是 false,当然这里主要还是决定是否包含e1和e2 ，可以看出默认是一个左闭右开的区间



```java
import java.util.TreeSet;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(2);
        numbers.add(5);
        numbers.add(4);
        numbers.add(6);
        System.out.println("TreeSet: " + numbers);

        // Using subSet() with default boolean value
        System.out.println("Using subSet without boolean value: " + numbers.subSet(4, 6));

        // Using subSet() with specified boolean value
        System.out.println("Using subSet with boolean value: " + numbers.subSet(4, false, 6, true));
    }
}
```

**Output**

```
TreeSet: [2, 4, 5, 6]
Using subSet without boolean value: [4, 5]
Using subSet with boolean value: [5, 6]
```

------

### Set Operations

接下来我们看一下一些集合相关的操作

------

#### Union of Sets

可以使用`addAll()` 来完成Union的操作

```java
import java.util.TreeSet;;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> evenNumbers = new TreeSet<>();
        evenNumbers.add(2);
        evenNumbers.add(4);
        System.out.println("TreeSet1: " + evenNumbers);

        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
        System.out.println("TreeSet2: " + numbers);

        // Union of two sets
        numbers.addAll(evenNumbers);
        System.out.println("Union is: " + numbers);

    }
}
```

**Output**

```
TreeSet1: [2, 4]
TreeSet2: [1, 2, 3]
Union is: [1, 2, 3, 4]
```

------

#### Intersection of Sets

使用 `retainAll` 完成交集的操作

```java
import java.util.TreeSet;;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> evenNumbers = new TreeSet<>();
        evenNumbers.add(2);
        evenNumbers.add(4);
        System.out.println("TreeSet1: " + evenNumbers);

        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
        System.out.println("TreeSet2: " + numbers);

        // Intersection of two sets
        numbers.retainAll(evenNumbers);
        System.out.println("Intersection is: " + numbers);
    }
}
```

**Output**

```
TreeSet1: [2, 4]
TreeSet2: [1, 2, 3]
Intersection is: [2]
```

------

#### Difference of Sets

使用`removeAll` 完成差集的操作

```java
import java.util.TreeSet;;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> evenNumbers = new TreeSet<>();
        evenNumbers.add(2);
        evenNumbers.add(4);
        System.out.println("TreeSet1: " + evenNumbers);

        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
        numbers.add(4);
        System.out.println("TreeSet2: " + numbers);

        // Difference between two sets
        numbers.removeAll(evenNumbers);
        System.out.println("Difference is: " + numbers);
    }
}
```

**Output**

```
TreeSet1: [2, 4]
TreeSet2: [1, 2, 3, 4]
Difference is: [1, 3]
```

------

#### Subset of a Set

To check if a set is a subset of another set or not, we use the `containsAll()` method. For example,

```java
import java.util.TreeSet;

class Main {
    public static void main(String[] args) {
        TreeSet<Integer> numbers = new TreeSet<>();
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
        numbers.add(4);
        System.out.println("TreeSet1: " + numbers);

        TreeSet<Integer> primeNumbers = new TreeSet<>();
        primeNumbers.add(2);
        primeNumbers.add(3);
        System.out.println("TreeSet2: " + primeNumbers);

        // Check if primeNumbers is subset of numbers
        boolean result = numbers.containsAll(primeNumbers);
        System.out.println("Is TreeSet2 subset of TreeSet1? " + result);
    }
}
```

**Output**

```
TreeSet1: [1, 2, 3, 4]
TreeSet2: [2, 3]
Is TreeSet2 subset of TreeSet1? True
```

------

### 4. TreeSet 的其他方法



| Method       | Description                                                  |
| :----------- | :----------------------------------------------------------- |
| `clone()`    | Creates a copy of the TreeSet                                |
| `contains()` | Searches the TreeSet for the specified element and returns a boolean result |
| `isEmpty()`  | Checks if the `TreeSet` is empty                             |
| `size()`     | Returns the size of the `TreeSet`                            |
| `clear()`    | Removes all the elements from the `TreeSet`                  |



## 总结

### TreeSet Vs. HashSet

虽然`TreeSet`和`HashSet`都是set 集合，但是由于它们的实现原理不同，所以它们还是有很多差异的

- 不同于`HashSet` ，`TreeSet` 里面的元素都是有序存储的，因为`TreeSet` 实现了`SortedSet` 接口
- `TreeSet`还提供了一些快速获取数据的方法， `first()`, `last()`, `headSet(`), `tailSet()`这是因为TreeSet 也实现了 NavigableSet 接口.

- `HashSet` 是比 `TreeSet`  要快的，例如一些基础的操作，像 add, remove, contains and size.





