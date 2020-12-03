### ConcurrentSkipListMap in Java With Examples

ConcurrentSkipListMap in Java is a scalable concurrent map which implements **ConcurrentNavigableMap** [interface](https://www.netjstech.com/2015/05/interface-in-java.html). Though concurrent collections like [ConcurrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html) and [CopyOnWriteArrayList](https://www.netjstech.com/2016/01/copyonwritearraylist-in-java.html) were added in Java 1.5, ConcurrentSkipListMap and the similar set implementation [ConcurrentSkipListSet](https://www.netjstech.com/2016/03/concurrentskiplistset-in-java.html) were added in Java 1.6.

### ConcurrentNavigableMap interface

ConcurrentNavigableMap interface in Java is a **ConcurrentMap** supporting NavigableMap operations, and recursively so for its navigable sub-maps. It was added in Java 1.6.

ConcurrentNavigableMap interface in turn extends NavigableMap interface. Where **NavigableMap** is a SortedMap extended with navigation methods returning the closest matches for given search targets. Methods **lowerEntry**, **floorEntry**, **ceilingEntry**, and **higherEntry** return [Map.Entry](https://www.netjstech.com/2018/09/mapentry-interface-in-java.html) objects associated with keys respectively less than, less than or equal, greater than or equal, and greater than a given key, returning null if there is no such key. Similarly, methods **lowerKey**, **floorKey**, **ceilingKey**, and **higherKey** return only the associated keys. *All of these methods are designed for locating, not traversing entries*.

### ConcurrentSkipListMap in Java

Since ConcurrentSkipListMap implements ConcurrentNavigableMap, it is a sorted map just like [TreeMap in Java](https://www.netjstech.com/2015/11/treemap-in-java.html) (Which also implements NavigableMap interface), with added functionality of being concurrent.

ConcurrentSkipListMap is sorted according to the natural ordering of its keys, or by a [Comparator](https://www.netjstech.com/2015/10/difference-between-comparable-and-comparator-java.html) provided at map creation time, depending on which [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html) is used.

### Java ConcurrentSkipListMap constructors

ConcurrentSkipListMap in Java provides four constructors, which are as follows-

- **ConcurrentSkipListMap()**- Constructs a new, empty map, sorted according to the natural ordering of the keys.
- **ConcurrentSkipListMap(Comparator<? super K> comparator)**- Constructs a new, empty map, sorted according to the specified comparator.
- **ConcurrentSkipListMap(Map<? extends K,? extends V> m)**- Constructs a new map containing the same mappings as the given map, sorted according to the natural ordering of the keys.
- **ConcurrentSkipListMap(SortedMap<K,? extends V> m)**- Constructs a new map containing the same mappings and using the same ordering as the specified sorted map.

ConcurrentSkipListMap class in Java implements a concurrent variant of **SkipLists** data structure providing expected average log(n) time cost for the containsKey, get, put and remove operations and their variants. Insertion, removal, update, and access operations safely execute concurrently by multiple threads.

All **Map.Entry** pairs returned by methods in ConcurrentSkipListMap class and its views represent snapshots of mappings at the time they were produced.

### No nulls in ConcurrentSkipListMap

Note that ConcurrentSkipListMap class in Java does not permit the use of null keys or values because some null return values cannot be reliably distinguished from the absence of elements.

### Java example creating ConcurrentSkipListMap

Let's see an example where we add some values in a ConcurrentSkipListMap and in the output it can be seen that it is sorted based on the natural ordering of its keys. In this example keys are Strings and for [String](https://www.netjstech.com/2016/07/string-in-java.html) natural ordering is ascending alphabetical order. So when you [loop the map](https://www.netjstech.com/2015/05/how-to-loop-iterate-hash-map-in-java.html) you'll see it is sorted based on the keys.

```
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentNavigableMap;
import java.util.concurrent.ConcurrentSkipListMap;

public class CSMDemo {
  public static void main(String[] args) {
    ConcurrentNavigableMap<String, String> cityMap = new ConcurrentSkipListMap<String, String>();
    cityMap.put("ND", "New Delhi");
    cityMap.put("MU", "Mumbai");
    cityMap.put("CH", "Chennai");
    cityMap.put("HD", "Hyderabad");
    Set<Map.Entry<String, String>> citySet = cityMap.entrySet();
    citySet.forEach((m)->System.out.println("key " + m.getKey() 
              + " value " + m.getValue()));
  }
}
```

**Output**

```
key CH value Chennai
key HD value Hyderabad
key MU value Mumbai
key ND value New Delhi
```

Here it can be seen that ConcurrentNavigableMap is sorted on the keys.

### ConcurrentSkipListMap with Comparator

If you want sorting to be done in reverse order then you can pass a Comparator as a parameter when constructing a ConcurrentSkipListMap.

```
public class CSMDemo {
  public static void main(String[] args) {
    // With Comparator
    ConcurrentNavigableMap<String, String> cityMap = new ConcurrentSkipListMap<String, String>((String s1, String s2) -> s1.compareTo(s2));
    cityMap.put("ND", "New Delhi");
    cityMap.put("MU", "Mumbai");
    cityMap.put("CH", "Chennai");
    cityMap.put("HD", "Hyderabad");
    Set<Map.Entry<String, String>> citySet = cityMap.entrySet();
    citySet.forEach((m)->System.out.println("key " + m.getKey() 
             + " value " + m.getValue()));
  }
}
```

**Output**

```
key CH value Chennai
key HD value Hyderabad
key MU value Mumbai
key ND value New Delhi
```

Here it can be seen that elements in the ConcurrentNavigableMap are now sorted in reversed order.

### Navigational methods in Java ConcurrentSkipListMap

As already mentioned **ConcurrentSkipListMap** in Java implements **ConcurrentNavigableMap** [interface](https://www.netjstech.com/2015/05/interface-in-java.html) so it has many navigation methods returning the closest matches for given search targets. Let's see some of them in example code.

- **descendingKeySet()**- Returns a reverse order NavigableSet view of the keys contained in this map.
- **floorEntry(K key)**- Returns a key-value mapping associated with the greatest key less than or equal to the given key, or null if there is no such key.
- **headMap(K toKey)**- Returns a view of the portion of this map whose keys are strictly less than toKey.
- **higherKey(K key)**- Returns the least key strictly greater than the given key, or null if there is no such key.

```
import java.util.Iterator;
import java.util.Map;
import java.util.NavigableSet;
import java.util.Set;
import java.util.concurrent.ConcurrentNavigableMap;
import java.util.concurrent.ConcurrentSkipListMap;

public class CSMDemo {
  public static void main(String[] args) {
    ConcurrentNavigableMap<String, String> cityMap = new ConcurrentSkipListMap<String, String>();
    cityMap.put("ND", "New Delhi");
    cityMap.put("MU", "Mumbai");
    cityMap.put("CH", "Chennai");
    cityMap.put("HD", "Hyderabad");
    System.out.println("---- Traversing the map-----");
    Set<Map.Entry<String, String>> citySet = cityMap.entrySet();
    // using for-each loop in Java 8
    citySet.forEach((m)->System.out.println("key " + m.getKey() + 
            " value " + m.getValue()));
        
    NavigableSet<String> reverseKeys = cityMap.descendingKeySet();
    // using iterator
    Iterator<String> itr = reverseKeys.iterator();
    System.out.println("---- Map keys in reverse order-----");
    while(itr.hasNext()){
      System.out.println("Key " + itr.next());
    }
        
    System.out.println("---- Floor entry-----");
    
    Map.Entry<String, String> tempMap = cityMap.floorEntry("II");
    System.out.println(tempMap);
        
    System.out.println("---- Head Map-----");
    ConcurrentNavigableMap<String, String> map = cityMap.headMap("MU");
    Set<Map.Entry<String, String>> set = map.entrySet();
    // using for-each loop in Java 8
    set.forEach((m)->System.out.println("key " + m.getKey() + 
                " value " + m.getValue()));
    
    System.out.println("---- Higher entry-----");
        
    tempMap = cityMap.higherEntry("II");
    System.out.println(tempMap);
  }
}
```

**Output**

```
---- Traversing the map-----
key CH value Chennai
key HD value Hyderabad
key MU value Mumbai
key ND value New Delhi
---- Map keys in reverse order-----
Key ND
Key MU
Key HD
Key CH
---- Floor entry-----
HD=Hyderabad
---- Head Map-----
key CH value Chennai
key HD value Hyderabad
---- Higher entry-----
MU=Mumbai
```

**DescendingKeySet** returns the keys in reverse order.

**floorEntry()** method returns the greatest key less than or equal to the given key. Here key is provided as "II" if you see greatest key less than or equal to "II" is "HD". Note here that key provided should not be a key already present in the Map ("II" is not a key present in the Map).

That's all for this topic **ConcurrentSkipListMap in JavaConcurrentSkipListMap in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!