### ConcurrentSkipListSet in Java With Examples

ConcurrentSkipListSet in Java is a scalable concurrent set in Java which uses [ConcurrentSkipListMap](https://www.netjstech.com/2016/03/concurrentskiplistmap-in-java.html) internally. Though concurrent collections like [ConcurrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html) and [CopyOnWriteArraySet](https://www.netjstech.com/2016/03/copyonwritearrayset-in-java-concurrency.html) were added in Java 1.5, **ConcurrentSkipListSet** and the similar map implementation ConcurrentSkipListMap were added in Java 1.6.

### ConcurrentSkipListSet in Java

Since ConcurrentSkipListSet implements **NavigableSet** in Java, it is a sorted set just like [TreeSet](https://www.netjstech.com/2015/10/treeset-in-java.html) with *added feature of being concurrent*. Which essentially means it is a sorted data structure which can be used by multiple [threads](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html) concurrently where as TreeSet is not thread safe.

The elements of the ConcurrentSkipListSet are kept sorted according to their **natural ordering**, or by a [Comparator](https://www.netjstech.com/2015/10/difference-between-comparable-and-comparator-java.html) provided at set creation time, depending on which [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html) is used.

### Java ConcurrentSkipListSet constructors

- **ConcurrentSkipListSet()**- Constructs a new, empty set that orders its elements according to their natural ordering.
- **ConcurrentSkipListSet(Collection<? extends E> c)**- Constructs a new set containing the elements in the specified collection, that orders its elements according to their natural ordering.
- **ConcurrentSkipListSet(Comparator<? super E> comparator)**- Constructs a new, empty set that orders its elements according to the specified comparator.
- **ConcurrentSkipListSet(SortedSet<E> s)**- Constructs a new set containing the same elements and using the same ordering as the specified sorted set.Paste your text here.

### Java ConcurrentSkipListSet performance

ConcurrentSkipListSet implementation provides expected average log(n) time cost for the **contains**, **add**, and **remove** operations and their variants. Insertion, removal, and access operations safely execute concurrently by multiple threads.

### No Nulls in ConcurrentSkipListSet

ConcurrentSkipListSet does not permit the use of null elements, because null arguments and return values cannot be reliably distinguished from the absence of elements.

### ConcurrentSkipListSet Java example

Let's see an example where we add some values in a **ConcurrentSkipListSet** and in the output it can be seen that the elements are sorted. In this example elements are of type String and for String natural ordering is ascending alphabetical order. So when you iterate the set you'll see it is in sorted same way.

Note that **ConcurrentSkipListSet** like any other set implementation i.e. [HashSet](https://www.netjstech.com/2015/09/how-hashset-works-internally-in-java.html) can only store unique elements. Also, as mentioned internally it uses [ConcurrentSkipListMap](https://www.netjstech.com/2016/03/concurrentskiplistmap-in-java.html) so when you call add method of ConcurrentSkipListSet it will in turn call **putIfAbsent()** method of the concurrentMap, *that way element is stored only if it is not there already*.

```
import java.util.Iterator;
import java.util.NavigableSet;
import java.util.concurrent.ConcurrentSkipListSet;

public class CSSDemo {
  public static void main(String[] args) {
    NavigableSet<String> citySet = new ConcurrentSkipListSet<String>();
    citySet.add("New Delhi");
    citySet.add("Mumbai");
    citySet.add("Chennai");
    citySet.add("Hyderabad");
    
    System.out.println("---- Traversing the set-----");
    Iterator<String> itr = citySet.iterator();
    while(itr.hasNext()){
      System.out.println("Value -  " + itr.next());
    }
  }
}
```

**Output**

```
---- Traversing the set-----
Value -  Chennai
Value -  Hyderabad
Value -  Mumbai
Value -  New Delhi
```

### Navigable methods in ConcurrentSkipListSet

As already mentioned **ConcurrentSkipListSet** in Java implements **NavigableSet** [interface](https://www.netjstech.com/2015/05/marker-interface-in-java.html) so it has many navigation methods returning the closest matches for given search targets. Let's see some of them in **example code**.

- **higher(E e)**- Returns the least element in this set strictly greater than the given element, or null if there is no such element.
- **lower(E e)**- Returns the greatest element in this set strictly less than the given element, or null if there is no such element.
- **tailSet(E fromElement)**- Returns a view of the portion of this set whose elements are greater than or equal to fromElement.

```
import java.util.Iterator;
import java.util.NavigableSet;
import java.util.Set;
import java.util.concurrent.ConcurrentSkipListSet;

public class CSSDemo {
  public static void main(String[] args) {
    NavigableSet<String> citySet = new ConcurrentSkipListSet<String>();
    citySet.add("New Delhi");
    citySet.add("Mumbai");
    citySet.add("Chennai");
    citySet.add("Hyderabad");
    
    System.out.println("---- Traversing the set-----");
    Iterator<String> itr = citySet.iterator();
    while(itr.hasNext()){
      System.out.println("Value -  " + itr.next());
    }
        
    System.out.println("Higher - " + citySet.higher("C"));    
    System.out.println("Lower - " + citySet.lower("Mumbai"));    
    System.out.println("---- Tail Set -----");
    
    Set<String> set = citySet.tailSet("Hyderabad");    
    itr = set.iterator();
    while(itr.hasNext()){
      System.out.println("Value -  " + itr.next());
    }
  }
}
```

**Output**

```
---- Traversing the set-----
Value -  Chennai
Value -  Hyderabad
Value -  Mumbai
Value -  New Delhi
Higher - Chennai
Lower - Hyderabad
---- Tail Set -----
Value -  Hyderabad
Value -  Mumbai
Value -  New Delhi
```

Here **higher** as the description says is returning the least element in this set strictly greater than the given element. Since given element is "C" so returned value is "Chennai". Note that passed element doesn't have to be the one already present in set as here "C" is passed which is not an element of the Set.

**lower** as the description says is returning the greatest element in this set strictly less than the given element. Passed element is "Mumbai" so that returned element is "Hyderabad".

That's all for this topic **ConcurrentSkipListSet in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!