In this post Java Collections interview questions and answers are listed. This compilation will help the Java developers in preparing for their interviews.

1. **What is Java Collections Framework? What are the benefits of using Collections framework?**

   Java Collections Framework provides a standard way to handle a group of objects. Benefits of the Collections Framework are-

   - **High performance**, implementation of the Collection classes (ArrayList, LinkedList, HashSet etc.) are very efficient and used as-is in most of the cases.
   - **Reduced effort** as framework already provides several implementations for most of the scenarios that can be used as-is.
   - Provides **interoperability**, as exp. if you are using List interface any implementation that implements List interface can be swapped with the existing one.
   - If you want to extend any collection that can be easily done using the standard interfaces provided with in the Collections frameworks.

2. ------

3. **Describe the Collection framework hierarchy.**

   At the **root** of the Collections framework is **Collection interface**, it must be implemented by any class that defines a collection. This interface declares the core methods that every collection will have, if any class doesn't implement any of the method then it can throw **UnsupportedOperationException**.
   Then there are **List** and **Set** interfaces that extend Collection interface and provided some of its own behaviour that will be further implemented by the classes that implement List and Set interfaces respectively.
   There is also a **Queue interface** that extends collection to provide behaviour of a queue. On the other hand there is Map interface which provides core methods for the Map implementations.

   [![Java collections hierarchy](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/16:51:05-collection%252Bhierarchy.png)](https://4.bp.blogspot.com/-XgvVWyptI5Y/XBDUGGAvQgI/AAAAAAAABDE/n089LFzkSBkoGGCqSmEFSQy5Me9Bf5mkwCPcBGAYYCw/s1600/collection%2Bhierarchy.png)

   

   **Collections Framework Hierarchy**

4. ------

5. **Name the core Collection classes.**

   - **AbstractCollection**- Implements most of the Collection interface.
   - **AbstractList**- Extends AbstractCollection and implements List interface.
   - **AbstractSequentialList** - Extends AbstractList to provide sequential access.
   - **[ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html)**- Extends AbstractList to provide dynamic array implementation.
   - **[LinkedList](https://www.netjstech.com/2015/08/how-linked-list-class-works-internally-java.html)**- Extends AbstractSequentialList to implement a linked list.
   - **AbstractQueue** - Extends AbstractCollection and implements Queue interface.
   - **ArrayDeque**- Resizable-array implementation of the Deque interface.
   - **PriorityQueue** - Extends AbstractQueue to provide a priority-based queue.
   - **AbstractSet**- Extends AbstractCollection and implements Set interface.
   - **[HashSet](https://www.netjstech.com/2015/09/how-hashset-works-internally-in-java.html)**- Extends AbstractSet backed by a HashTable (actually a HashMap instance).
   - **[LinkedHashSet](https://www.netjstech.com/2015/09/linkedhashset-in-java.html)**- Extends HashSet to provide insertion order interation.
   - **[TreeSet](https://www.netjstech.com/2015/10/treeset-in-java.html)**- Extends AbstractSet (and implements NavigableSet interface) to provide sorted set.
   - **[EnumSet](https://www.netjstech.com/2015/10/enumset-in-java.html)**- Extends AbstractSet for use with Enum elements.

6. ------

7. **How generics changed the collection framework?**

   With **JDK5** the Collection framework was reengineered to add **generics**. That was done to add "**Type Safety**" to the collections. Prior to JDK5 (and generics) elements were stored in collections as Object references, which brought the danger of accidentally storing incompatible types in Collection because for Collections everything was Object reference. Which would have resulted in run time error when trying to retrieve elements and casting them to the desired type.
   With the introduction of generics now we can explicitly tell which data type is to be stored in the collections, which helps in avoiding the run time error. As Collection will throw error at compile time itself for incompatible data type. As exp.

   ```
   Map<String, String> cityTemperatureMap = new LinkedHashMap<String, String>();
   ```

   Here we are explicitly stating that this LinkedHashMap can only store string as both

    

   key and value

   .

   

8. ------

9. **What change does autoboxing bring to Collection?**

   Since we know that collection **cannot store primitive types**, only **references**. Before the introduction of **autoboxing/unboxing** in JDK5, if you wanted to strore primitive value like int in a collection, it had to be wrapped in an Integer class manually. While retrieving the value it had to be cast manually into its primitive type.
   Collection still store only references but autoboxing/unboxing facilitate it to **happen automatically**, with no need to wrap it manually.

   ```
   List<Integer> tempList = new ArrayList<Integer>();
   // Storing in collection using autoboxing
   tempList.add(1);
   // wrapping primitive int 2 to Integer manually, with out autoboxing
   tempList.add(new Integer(2));
   ```

   

10. ------

11. **What is for-each style loop**

    Another feature which was added to Collection with **JDK 5** is **for-each style loop**. Any collection which wants to be the target of the "for-each loop" statement has to implement **iterable** interface.
    Using for-each loop is easier than constructing the iterator to iterate over the collection and can be used in most of the cases rather than using the iterator loop.
    If you have a list of Strings, that can be iterated using for-each loop like this.

    ```
    for(String city : cityList){
        System.out.println("Name " + city);
    }
    ```

    

12. ------

13. **What is a diamond operator?**

    Diamond operator let the compiler infer the type arguments for the generic classes. *It is added in Java 7*.
    **As Example** - Before **JDK7** if we had to define a Map using String as both Key and Value we had to write it like this -

    ```
    Map<String, String> cityMap = new HashMap<String, String>();
    ```

    In Java SE 7, you can substitute the parameterized type of the constructor with an empty set of type parameters (

    <>

    ) known as

     

    diamond operator

    .

    ```
    Map<String, String> cityMap = new HashMap<>();
    ```

    

14. ------

15. **What are the changes in Collections framework in Java 8?**

    There are several changes in Collections Framework in Java 8 mostly influenced by the inclusion of [Lambda expression in Java 8](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) -

    - Stream API

      \- Stream can be obtained for Collection via the stream() and parallelStream() methods;

      As exp

       

      if you have a list of cities in a List and you want to remove the duplicate cities from that list it can be done as

      ```
      cityList = cityList.stream().distinct().collect(Collectors.toList());
      ```

    - **ForEach loop** which can be used with Collection. **Spliterators** which are helpful in parallel processing where several threads can itearate/process part of the collection.

    - New methods are added to Collections like replaceAll, getOrDefault, putIfAbsent in Map.

    - HashMap, [LinkedHashMap](https://www.netjstech.com/2015/11/linkedhashmap-in-java.html) and ConcurrentHashMap.implementation is changed to reduce hash collisions. Instead of linked list a balanced tree is used to store the entries after a certain threshold is reached.
      Refer [How HashMap internally works in Java](https://www.netjstech.com/2015/05/how-hashmap-internally-works-in-java.html) to know more about the changes done in Java 8 to HashMap internal implementation.

    

16. ------

17. **What is ForEach statement added in Java 8?**

    Java 8 has added a functional style lopping to the Collections. Real advantage of this loop is when it is used on a stream with the chain of functional methods. If you have a Map<String, String> then it can be looped using ForEach statement like this -

    ```
    Set<Map.Entry<String, String>> valueSet = cityMap.entrySet();
    valueSet.forEach((a)->System.out.println("Key is " + a.getKey() + " Value is " + a.getValue()));
    ```

    

18. ------

19. **What is RandomAccess interface?**

    RandomAccess interface is a [marker interface](https://www.netjstech.com/2015/05/marker-interface-in-java.html) used by List implementations to indicate that they support fast (generally constant time) random access.
    Note that using it with a sequential access list like LinkedList will result in taking more time to access a particular element.

20. ------

21. **Which collection classes implement random access interface?**

    In Java.util package the classes that implement random access interface are- ArrayList, CopyOnWriteArrayList, Stack, Vector.

22. ------

23. **What is the difference between Collection and Collections?**

    **Collection** is an interface which is the root of the Collections framework, as you can see in question#2, List, Set and Queue are the interfaces that extend the Collection interface.
    **Collections** is a utility class with static methods that operate on or return Collections. It provides several methods like sort, reverse, synchronizedCollection, binarySearch etc.

24. ------

25. **What all Collection classes are inherently thread-safe?**

    In the initial Collection classes like Vector, HashTable and Stack all the methods were synchronized to make these classes thread safe.
    Later implementations starting from Java 1.2 were not synchronized.
    Classes in java.util.concurrent package like ConcurrentHashMap, CopyOnWriteArrayList also provides thread safety but with a different implementation that doesn't require making all the methods synchronized.

26. ------

27. **How can we synchronize a collection OR how to make a collection thread-safe?**

    Collections class has several static methods that return synchronized collections. Each of the six core collection interfaces -Collection, Set, List, Map, SortedSet, and SortedMap - has one static factory method.

    ```
    public static <T> Collection<T> synchronizedCollection(Collection<T> c);
    public static <T> Set<T> synchronizedSet(Set<T> s);
    public static <T> List<T> synchronizedList(List<T> list);
    public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m);
    public static <T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s);
    public static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m); 
    ```

    Each of these methods returns a synchronized (thread-safe) Collection backed up by the specified collection.

    

28. Refer

29.  

30. How and why to synchronize ArrayList in Java

31.  

32. to know how to synhronize ArrayList.

33. ------

34. **How to make a Collection Class immutable?**

    Collections class provides static method to make a Collection unmodifiable. Each of the six core collection interfaces -Collection, Set, List, Map, SortedSet, and SortedMap - has one static factory method.

    ```
    public static <T> Collection<T> unmodifiableCollection(Collection<? extends T> c);
    public static <T> Set<T> unmodifiableSet(Set<? extends T> s);
    public static <T> List<T> unmodifiableList(List<? extends T> list);
    public static <K,V> Map<K, V> unmodifiableMap(Map<? extends K, ? extends V> m);
    public static <T> SortedSet<T> unmodifiableSortedSet(SortedSet<? extends T> s);
    public static <K,V> SortedMap<K, V> unmodifiableSortedMap(SortedMap<K, ? extends V> m);
    ```

    

35. ------

36. **What is List interface?**

    List interface extends the root interface Collection and adds behavior for the collection that stores a sequence of elements. These elements can be inserted or accessed by their position in the list using a zero-based index.

37. ------

38. **Which Collection classes implement List interface?**

    Some of the Collection classes that implement List interface are [ArrayList](https://www.netjstech.com/2015/08/how-arraylist-works-internally-in-java.html), CopyOnWriteArrayList, [LinkedList](https://www.netjstech.com/2015/08/how-linked-list-class-works-internally-java.html), Stack, Vector

39. ------

40. **Why ArrayList is called dynamically growing array? How that dynamic behavior is achieved?**

    In the case of ArrayList you don't have to anticipate in advance, like in the case of array, how many elements you are going to store in the arraylist. As and when elements are added list keeps growing. That is why ArrayList is called **dynamically growing array**.
    For ArrayList, data structure used for storing elements is **array** itself. When ArrayList is created it initializes an array with an **initial capacity** (default is array of length 10). When that limit is crossed another array is created which is 1.5 times the original array and the elements from the old array are copied to the new array.

    Refer

     

    ArrayList in Java

     

    to know more about ArrayList in Java.

    Refer

     

    How ArrayList works internally in Java

     

    to know how ArrayList works internally in Java.

41. ------

42. **How ArrayList works internally in Java?**

    Basic data structure used by ArrayList to store objects is an array of Object class, which is defined like -
    **transient Object[] elementData;**
    When we add an element to an ArrayList it first verifies whether it has that much capacity in the array to store new element or not, in case there is not then the new capacity is calculated which is **50%** more than the old capacity and the array is increased by that much capacity (Actually uses **Arrays.copyOf** which returns the original array increased to the new length)

    Refer

     

    How ArrayList works internally in Java

     

    to know how ArrayList works internally in Java.

43. ------

44. **How to add elements to an ArrayList?**

    

    - List provides a method **add(E e)** which appends specified element to the end of the list. Using add(E e) method will mean keep adding elements sequentially to the list.
    - Another add method - **add(int index, E element)** inserts the specified element at the specified position in this list.
    - Third method **addAll(int index, Collection<? extends E> c)** inserts all of the elements in the specified collection into this list, starting at the specified position.

    

45. ------

46. **Does ArrayList allow duplicate elements?**

    Yes. List allows duplicate elements to be added.

47. ------

48. **Does ArrayList allow null?**

    In ArrayList any number of nulls can be added.

49. ------

50. **How to join two or more ArrayLists?**

    List provides a method addAll to join two or more lists in Java.
    If you have one list cityList and another List secondCityList then you can join them using **addAll** like this -
    **cityList.addAll(secondCityList);**

    Refer

     

    How to join lists in Java

     

    to know more ways to join Lists in Java.s

51. ------

52. **How to remove elements from an ArrayList?**

    ArrayList provides several methods to remove elements from the List. Since ArrayList **internally uses array** to store elements, one point to note is that when an element is removed from the List the remaining elements **have to be shifted** to fill the gap created in the underlying array.

    - **clear()**- Removes all of the elements from this list.
    - **remove(int index)**- Removes the element at the specified position in this list.
    - **remove(Object o)**- Removes the first occurrence of the specified element from this list, if it is present.
    - **removeAll(Collection<?> c)**- Removes from this list all of its elements that are contained in the specified collection.
    - **removeIf(Predicate<? super E> filter)**- Removes all of the elements of this collection that satisfy the given predicate.

    Note that

     

    removeIf

     

    is added in Java 8

    

    Refer

     

    How to remove elements from an ArrayList in Java

     

    to know more ways to remove elements from an ArrayList in Java.

53. ------

54. **How to remove duplicate elements from an ArrayList?**

    We can use a HashSet to do the job of removing the duplicate elements. HashSet only stores unique elements and we'll use that feature of HashSet to remove duplicates.
    If you have a List called cityList you can create a HashSet using this list -
    **Set<String> citySet = new HashSet<String>(cityList);**
    then add this set back to the cityList
    **cityList.clear();
    cityList.addAll(citySet);
    **That will remove all the duplicate elements from the given list. Note that insertion order won't be retained if a HashSet is used. In case insertion order is to be retained use [LinkedHashSet](https://www.netjstech.com/2015/09/linkedhashset-in-java.html).

    Refer

     

    How to remove duplicate elements from an ArrayList in Java

     

    to know more ways to remove duplicate elements from an ArrayList in Java.

55. ------

56. **How to loop/iterate an ArrayList in Java?**

    There are many ways to loop/iterate an arrayList in Java. Options are -

    - for loop
    - for-each loop
    - iterator
    - list iterator
    - Java 8 forEach loop

    for-each loop is the best way to iterate a list if you just need to traverse sequentially through a list.

    

    Refer

     

    How to loop/iterate an arraylist in Java

     

    to know more ways to loop ArrayList in Java.

57. ------

58. **What is ListIterator in Java?**

    ListIterator provides the functionality to iterate a list in **both directions**. The interesting point about list iterator is that it has no current element. Its current cursor position always lies between the element that would be returned by a call to **previous()** and the element that would be returned by a call to **next()**.

    Refer

     

    List iterator in Java

     

    to know more about ListIterators in Java.

59. ------

60. **How to sort ArrayList in Java?**

    Collections class has a static method sort which can be used for sorting an arraylist.
    There are two overloaded versions of sort method -

    - **public static <T extends Comparable<? super T>> void sort(List<T> list)**- Sorts the specified list into ascending order, according to the natural ordering of its elements.
    - **public static <T> void sort(List<T> list, Comparator<? super T> c)**- Sorts the specified list according to the order induced by the specified comparator.

    If you have a List called cityList it can be sorted like this-

    Collections.sort(cityList);

    

    Refer

     

    How to sort arraylist in Java

     

    to know more about How to sort ArrayList in Java.

61. ------

62. **How to sort ArrayList of Strings in descending order?**

    Collections.sort() method will sort the ArrayList of String in ascending order.
    To have a **descending order** either comparator has to be provided or **reverseOrder** method of the Collections class can be used.
    **Collections.sort(cityList, Collections.reverseOrder());**

    Refer

     

    How to sort arraylist in Java

     

    to know more about How to sort ArrayList in Java.

63. ------

64. **How to sort ArrayList of custom objects in Java?**

    Refer [How to sort arraylist of custom objects in Java](https://www.netjstech.com/2015/08/how-to-sort-arraylist-of-custom-objects-java.html) to know more about How to sort ArrayList of custom objects in Java.

65. ------

66. **How and why to synchronize ArrayList in Java?**

    In order to have better performance most of the Collections are not synchronized. Which means sharing an instance of arrayList among many threads where those threads are modifying *(by adding or removing the values)* the collection may result in unpredictable behavior.
    To synchronize a List **Collections.synchronizedList()** method can be used.

    Refer

     

    How and why to synchronize ArrayList in Java

     

    to know more about How to synchronize ArrayList in Java.

67. ------

68. **How to convert ArrayList to Array in Java?**

    Within Collection interface there are two versions of toArray() which can be used to convert ArrayList to array.

    - Object[] toArray()
    - <T> T[] toArray(T array[])

    The first version returns an array of Object.

    The second version returns an array containing the elements of the list. Normally we go with the second version because it returns the array of the same type as List.

    If you have a List called cityList, it can be converted to an array like this -

    ```
    String cityArray[] = new String[cityList.size()];   
    cityArray = cityList.toArray(cityArray);
    ```

    

    Refer

     

    How to convert ArrayList to Array in Java

     

    to know more about How to convert ArrayList to Array in Java.

69. ------

70. **How to convert Array to ArrayList in Java?**

    Arrays class contains a static factory method asList() that allows arrays to be viewed as lists.
    **public static <T> List<T> asList(T... a)**
    **As Exp.**

    ```
    String cityArray[] = {"Delhi", "Mumbai", "Bangalore", "Hyderabad", "Chennai"};
    //Converting array to List
    List<String> cityList = Arrays.asList(cityArray);
    ```

    

    Refer

     

    How to convert Array to ArrayList in Java

     

    to know more about How to convert Array to ArrayList in Java.

71. ------

72. **How is the performance of an ArrayList?**

    

    - **Adding an element**- If you are adding at the end using **add(E e)** method it is **O(1)**. In the worst case it may go to **O(n)**. That will happen if you add more elements than the capacity of the underlying array.
    - **Retrieving an element**- Since ArrayList internally uses an array to store elements so **get(int index)** means going to that index directly in the array. So, for ArrayList get(int index) is **O(1)**.
    - **Removing an element**- If you are removing using the **remove(int index)** method then, in case of ArrayList getting to that index is fast but removing will mean shuffling the remaining elements to fill the gap created by the removed element with in the underlying array. Thus it can be said remove(int index) operation is **O(n - index)** for the arraylist.

    ```
    String cityArray[] = {"Delhi", "Mumbai", "Bangalore", "Hyderabad", "Chennai"};
    //Converting array to List
    List<String> cityList = Arrays.asList(cityArray);
    ```

    

    ------

73. **What is LinkedList class in Java? How is it implemented?**

    LinkedList class in Java implements **List** and **Deque** interfaces and LinkedList implements it using **doubly linkedlist**.
    Within the LinkedList implementation there is a private class Node which provides the structure for a node in a doubly linked list. It has item variable for holding the value and two reference to Node class itself for connecting to next and previous nodes.

    Refer

     

    How Linked List class works internally in Java

     

    to know more about LinkedList in Java.

74. ------

75. **What is Deque interface in Java?**

    **Java.util.Deque** is an interface in Java which extends Queue interface and provides support for element insertion and removal at both ends. The name deque is short for "**double ended queue**" and is usually pronounced "deck".
    Some of the implementing classes of the Deque interface are [LinkedList](https://www.netjstech.com/2015/08/how-linked-list-class-works-internally-java.html), ConcurrentLinkedDeque, LinkedBlockingDeque. Note that addFirst() and addLast() methods provided in the LinkedList implementation are specified by the Deque interface.

76. ------

77. **Difference between ArrayList and LinkedList in Java?**

    In Java collections framework [ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html) and [LinkedList](https://www.netjstech.com/2015/08/how-linked-list-class-works-internally-java.html) are two different implementations of List interface

    - LinkedList is implemented using a **doubly linked list** concept where as ArrayList internally uses an **array of Objects** which can be resized dynamically
    - For LinkedList **add(e Element)** is always **O(1)** where as for ArrayList **add(e Element)** operation runs in amortized constant time, that is, adding n elements requires **O(n)** time.
    - For LinkedList **get(int index)** is O(n) where as for ArrayList **get(int index)** is O(1).
    - If you are removing using the remove(int index) method then for LinkedList class it will be O(n). In case of ArrayList getting to that index is fast but removing will mean shuffling the remaining elements to fill the gap created by the removed element with in the underlying array.

    

    Refer

     

    Difference between ArrayList and LinkedList in Java

     

    to see differences in detail.

78. ------

79. **Difference between ArrayList and Vector in Java?**

    

    - **ArrayList** is not [synchronized](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) whereas **Vector** is synchronized.
    - **Performace** wise ArrayList is fast in comparison to Vector as ArrayList is not synchronized.
    - ArrayList, by default, grows by 50% in case the initial capacity is exhausted. In case of Vector the backing array's size is doubled by default.
    - For [traversing an ArrayList](https://www.netjstech.com/2015/08/how-to-loop-iterate-arraylist-in-java.html) iterator is used. For traversing Vecor **Iterator/Enumerator** can be used. Note that Iterator is [fail-fast](https://www.netjstech.com/2015/05/fail-fast-vs-fail-safe-iterator-in-java.html) for both Vector and ArrayList. Enumerator which can be used with Vector is **not fail-fast**.

    

    Refer

     

    Difference between ArrayList and Vector in Java

     

    to see differences in detail.

80. ------

81. **Difference between Array and ArrayList in Java?**

    

    - **Array** is fixed in size which is provided at the time of creating an Array. ArrayList grows dynamically and also known as **dynamically growing array**.
    - Array can store primitive types as well as objects. In ArrayList only Objects can be stored.
    - Arrays can be multi-dimensional whereas ArrayList is always unidimensional.
      As Exp. a two dimensional array can be created as -
      **Integer myArray[][] = new Integer[4][3];**
    - **Performance wise** both Array and ArrayList are almost same as *internally ArrayList also uses an Array*. But there is **overhead of resizing** the array and copying the elements to the new Array in case of ArrayList.

    

    Refer

     

    Difference between Array and ArrayList in Java

     

    to see differences in detail.

82. ------

83. **What is an Iterator?**

    Iterator is an interface which is part of the Java Collections framework. An Iterator enables you to iterate a collection. Iterators also allow the caller to remove elements from the underlying collection during the iteration.
    Collection classes provide iterator method which returns an iterator. This iterator can be used to traverse a collection.
    **As exp.** if there is a set called citySet, an iterator on this set can be obtained as -

    ```
    Iterator<String> itr = citySet.iterator();
    while (itr.hasNext()) {
        String city = (String) itr.next();
        System.out.println("city " + city);        
    }
    ```

    

84. ------

85. **Difference between iterator and ListIterator?**

    

    - **Iterator** can be obtained on any Collection class like List or Set. But **ListIterator** can only be used to traverse a List.

    - **Iterator** only moves in one direction using **next()** method. **ListIterator** can iterate in **both directions** using next() and previous() methods.

    - Iterator

       

      always start at the beginning of the collection.

      ListIterator

       

      can be obtained at any point.

      As Exp.

       

      If you have a List of integers numberList, you can obtain a ListIterator from the third index of this List.

      ```
      ListIterator<Integer> ltr = numberList.listIterator(3);
      ```

    - ListIterator provides an **add(E e)** method which is not there in Iterator. add(E e) inserts the specified element into the list.

    - ListItearator also provides set method. void **set(E e)** replaces the last element returned by next() or previous() with the specified element

    

    Refer

     

    List iterator in Java

     

    to know more about ListIterator in Java.

86. ------

87. **Why Iterator doesn't have add method whereas ListIterator has add method?**

    **Iterator** can be obtained on any Collection class like List or Set so contract for Iterator makes no guarantees about the order of iteration.
    But ListIterator can only be used to **traverse a List** so it does guarantee the order of the iteration. That is why ListIterator provides an add operation.

88. ------

89. **What is fail fast iterator?**

    An iterator is considered fail-fast if it throws a **ConcurrentModificationException** under either of the following two conditions:

    - In multi-threaded environment, if one thread is trying to modify a Collection while another thread is iterating over it.
    - Even with single thread, if a thread modifies a collection directly while it is iterating over the collection with a fail-fast iterator, the iterator will throw this exception.

    fail-fast iterator will throw a ConcurrentModificationException if the underlying collection is structurally modified in any way except through the iterator's own remove or add (if applicable as in list-iterator) methods.

    Note that structural modification is any operation that adds or deletes one or more elements; merely setting the value of an element (in case of list) or changing the value associated with an existing key (in case of map) is not a structural modification.

    

90. ------

91. **What is a fail-safe iterator?**

    An iterator is considered fail-safe if it does not throw **ConcurrentModificationException**. ConcurrentModificationException is not thrown as the fail-safe iterator makes a copy of the underlying structure and iteration is done over that snapshot.
    Since iteration is done *over a copy of the collection so interference is impossible* and the iterator is guaranteed not to throw ConcurrentModificationException.

92. ------

93. **Difference between fail-fast iterator and fail-safe iterator.**

    

    - **fail-fast iterator** throws a ConcurrentModificationException if the underlying collection is structurally modified whereas **fail-safe iterator** doesn't throw ConcurrentModificationException.
    - **fail-fast** iterator doesn't create a copy of the collection whereas **fail-safe** iterator makes a copy of the underlying structure and iteration is done over that snapshot.
    - **fail-fast** iterator provides operations like remove, add and set (in case of ListIterator) whereas in case of **fail-safe** iterators element-changing operations on iterators themselves (**remove, set and add**) are not supported. These methods throw **UnsupportedOperationException**.

    

    Refer

     

    fail-fast Vs fail-safe iterator in Java

     

    to know more about the differences between fail-fast and fail-safe iterator.

94. ------

95. **Can we iterate through a Map?**

    Though we can't iterate a Map as such but Map interface has methods which provide a **set view of the Map**. That set can be iterated. Those two methods are-

    - **Set<Map.Entry<K, V>>entrySet()**- This method returns a set that contains the entries in the map. The entries in the set are actually object of type Map.Entry.
    - **Set<K>keySet()**- This method returns a set that contains the keys of the map.

    There is also a

     

    values()

     

    method in the Map that returns a Collection view of the values contained in this map.

    

    Refer

     

    How to loop through a map in Java

     

    to see code examples of iterating a Map.

96. ------

97. **Are Maps actually collections OR Why doesn't Map extend Collection?**

    Maps themselves are not collections because they don't implement Collection interface. One of the reason as mentioned here https://docs.oracle.com/javase/8/docs/technotes/guides/collections/designfaq.html#a14 is because elements are stored in a Map as a Key-Value pair which provides a very limited abstraction.
    Maps can be viewed as Collections (of keys, values, or pairs), and this fact is reflected in the three "**Collection view operations**" on Maps **(keySet, entrySet, and values)**.

98. ------

99. **What is Map.Entry in Map?**

    Map.Entry is an interface in java.util, in fact Entry is a [nested interface](https://www.netjstech.com/2015/05/interface-in-java.html) in Map interface. Nested interface must be qualified by the name of the class or interface of which it is a member, that's why it is qualified as Map.Entry.
    **Map.Entry** represents a key/value pair that forms one element of a Map.
    The **Map.entrySet** method returns a collection-view of the map, whose elements are of this class.
    **As exp.** If you have a Map called cityMap then using entrySet method you can get the set view of the map whose elements are of type Map.Entry, and then using getKey and getValue methods of the Map.Entry you can get the key and value pair of the Map.

    ```
    for(Map.Entry<String, String> entry:  cityMap.entrySet()){
       System.out.println("Key is " + entry.getKey() + " Value is " + entry.getValue());
    }
    ```

    

100. ------

101. **How Does HashMap internally works in Java?**

     Please Refer [How HashMap internally works in Java](https://www.netjstech.com/2015/05/how-hashmap-internally-works-in-java.html) to know about the internal implementation of the HashMap.

102. ------

103. **Does HashMap allow null keys and null values?**

     HashMap allows *one null key and any number of null values*. If another null key is added it **won't cause** any error. But the *value with the new null key will override the value with old null key*.
     **As exp.** If you have a Map, cityMap and you add two values with null keys in the cityMap, Kolkata will override the New Delhi as only one null key is allowed.

     ```
     cityMap.put(null, "New Delhi");
     cityMap.put(null, "Kolkata");
     ```

     

104. ------

105. **What happens if HashMap has duplicate keys?**

     If an attempt is made to add the same key twice, it won't cause any error but the value which is added later will override the previous value.
     **As Exp.** If you have a Map, cityMap and you add two values with same key in the cityMap, Kolkata will override the New Delhi.

     ```
     cityMap.put("5", "New Delhi");
     cityMap.put("5", "Kolkata");
     ```

     

106. ------

107. **What is hash-collision in hash based collections?**

     In HashMap, using the key, a Hash is calculated and that hash value decides in which bucket the particular **Map.Entry** object will reside.
     Hash collision means *more than one key having the same calculated hash value* thus stored in the same bucket. In HashMap, in that case Entry objects are **stored as a linked-list** with in the same bucket.

     Refer

      

     How HashMap internally works in Java

      

     to know about the internal implementation of the HashMap.

108. ------

109. **What is the hashCode() method?**

     hashCode() method is present in the java.lang.Object class. *This method is used to get a unique integer value for a given object*. We can see it's use with **hash based collections** like HashTable or HashMap where hashCode() is used to find the **correct bucket location** where the particular (key, value) pair is stored.

     Refer

      

     Overriding hashCode() and equals() method in Java

      

     to know more about the hashCode() method.

110. ------

111. **What is equals() method?**

     equals() method is present in the java.lang.Object class. It is used to **determine the equality of two objects**.

     Refer

      

     Overriding hashCode() and equals() method in Java

      

     to know more about the equals() method.

112. ------

113. **When do we need to override hashCode() and equals() methods?**

     The default implementation of **equals()** method in the Object class is a simple reference equality check.

     ```
     public boolean equals(Object obj){
          return (this == obj);
     }
     ```

     The default implementation of

      

     hashCode()

      

     in the Object class just returns integer value of the memory address of the object.

     It becomes very important to override these two methods in case we are using a

      

     custom object as key

      

     in a hash based collection.

     In that case we can't rely on the default implementation provided by the Object class and need to provide custom implementation of hashCode() and equals() method.

     If two objects Obj1 and Obj2 are equal according to their equals() method

      

     then they must have the same hash code too

     . Though the vice-versa is not true that is if two objects have the same hash code then they do not have to be equal too.

     

     Refer

      

     Overriding hashCode() and equals() method in Java

      

     to know more about Overriding hashCode() and equals() method.

114. ------

115. **Which Map implementation should be used if you want to retain the insertion order?**

     LinkedHashMap should be used in this case.

     Refer

      

     LinkedHashMap in Java

      

     to know more about LinkedHashMap in Java.

116. ------

117. **What is LinkedHashMap?**

     LinkedHashMap is also one of the implementation of the Map interface, apart from implementing Map interface LinkedHashMap also extends the HashMap class. So just like HashMap, LinkedHashMap also allows one null key and multiple null values.
     How it differs from other implementations of the Map interface like HashMap and TreeMap is that **LinkedHashMap maintains the insertion order of the elements** which means if we iterate a LinkedHashMap we'll get the keys in the order in which they were inserted in the Map.
     LinkedHashMap maintains a doubly-linked list running through all of its entries and that's how it maintains the iteration order.

     Refer

      

     LinkedHashMap in Java

      

     to know more about LinkedHashMap in Java.

118. ------

119. **Which Map implementation should be used if you want Map values to be sorted by keys?**

     TreeMap should be used in this case.

     Refer

      

     Treemap in Java

      

     to know more about TreeMap in Java.

120. ------

121. **What is TreeMap in Java?**

     TreeMap is also one of the implementation of the Map interface like HashMap and LinkedHashMap. TreeMap class implements the NavigableMap interface and extends the AbstractMap class.
     How it differs from other implementations of the Map interface is that **objects in TreeMap are stored in sorted order**. The elements are ordered *using their natural ordering or a comparator can be provided at map creation time to provide custom ordering*.
     One important point about TreeMap is, though HashMap and LinkedHashMap allow one null as key, **TreeMap doesn't allow null as key**. Any attempt to add null in a TreeMap will result in a NullPointerException.

     Refer

      

     Treemap in Java

      

     to know more about TreeMap in Java.

122. ------

123. **What is a WeakHashMap?**

     WeakHashMap is a Hash table based implementation of the Map interface, with **weak keys**. An entry in a WeakHashMap will automatically be removed when its key is no longer in ordinary use. Which means storing only weak references allows garbage collector to remove the entry (key-value pair) from the map when its key is not referenced outside of the WeakHashMap.
     In WeakHashMap both null values and the null key are supported. A WeakHashMap is created as-
     **Map weakHashMap = new WeakHashMap();**

124. ------

125. **What is a IdentityHashMap?**

     IdentityHashMap class implements the Map interface with a hash table, using **reference-equality** in place of **object-equality** when comparing keys (and values). In other words, in an IdentityHashMap, two keys k1 and k2 are considered equal if and only if **(k1==k2)**. Where as In normal Map implementations (like HashMap) two keys k1 and k2 are considered equal if and only if **(k1==null ? k2==null : k1.equals(k2))**.
     **Note that** This class is not a general-purpose Map implementation. While this class implements the Map interface, it intentionally violates Map's general contract, which mandates the use of the equals method when comparing objects. This class is designed for use only in the rare cases wherein reference-equality semantics are required.

126. ------

127. **Difference between HashMap and HashTable in Java.**

     Though both HashTable and HashMap store elements as a (key, value) pair and use hashing technique to store elements, moreover from Java v1.2, HashTable class was retrofitted to implement the Map interface, making it a member of the Java Collections Framework. But there are certain difference between the two -

     - **HashMap** is not [synchronized](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) where as **HashTable** is synchronized.
     - **HashMap** allows one null value as a key and any number of null values where as **HashTable** does not allow null values either as key or as value.
     - For [traversing a HashMap](https://www.netjstech.com/2015/05/how-to-loop-iterate-hash-map-in-java.html) an iterator can be used. For **traversing a HashTable** either an iterator or Enumerator can be used. The iterator used for both HashMap and HashTable is [fail-fast](https://www.netjstech.com/2015/05/fail-fast-vs-fail-safe-iterator-in-java.html) but the enumerator used with HashTable is fail-safe.
     - Performance wise **HashMap** is faster than the **HashTable** reason being HashMap is not synchronized.

     

     Refer

      

     Difference between HashMap and HashTable in Java

      

     to see in detail.

128. ------

129. **HashMap Vs LinkedHashMap Vs TreeMap in Java.**

     

     - **HashMap** makes no guarantees as to the order of the map.
       **LinkedHashMap** maintains the insertion order of the elements which means if we iterate a LinkedHashMap we'll get the keys in the order in which they were inserted in the Map.
       **TreeMap** stores objects in sorted order.
     - **HashMap** as well as **LinkedHashMap** allows one null as key, multiple values may be null though.
       **TreeMap** does not allow null as key.
     - **HashMap** stores elements in a bucket which actually is an index of the array.
       **LinkedHashMap** also uses the same internal implementation, it also maintains a doubly-linked list running through all of its entries.
       **TreeMap** is a Red-Black tree based NavigableMap implementation.
     - Performance wise **HashMap** provides constant time performance O(1) for get() and put() method.
       **LinkedHashMap** also provides constant time performance O(1) for get() and put() method but in general a little slower than the HashMap as it has to maintain a doubly linked list.
       **TreeMap** provides guaranteed log(n) time cost for the containsKey, get, put and remove operations.

     

     Refer

      

     HashMap Vs LinkedHashMap Vs TreeMap in Java

      

     to see in detail.

130. ------

131. **Difference between Comparable and Comparator.**

     

     | Comparable                                                   | Comparator                                                   |
     | ------------------------------------------------------------ | ------------------------------------------------------------ |
     | **Comparable** interface is in **java.lang** package.        | **Comparator** interface is in **java.util** package.        |
     | **Comparable** interface provides **public int compareTo(T o);** method which needs to be implemented for sorting the elements. This method compares this object with object o and returns an integer, if that integer is- **Positive**- this object is greater than o.**Zero**- this object is equal to o.**Negative**- this object is less than o. | **Comparator** interface provides **int compare(T o1, T o2);** method which needs to be implemented for sorting the elements. Here o1 and o2 objects are compared and an integer value is returned, if that integer is- **Positive**- o1 is greater than o2.**Zero**- o1 is equal to o2.**Negative**- o1 is less than o2. |
     | The class which has to be sorted should implement the **comparable** interface (sorting logic is in the class which has to be sorted), and that implementation becomes the natural ordering of the class. | Some other class can implement the **Comparator** interface not the actual class whose objects are to be sorted. That way there may be many comparators and depending on the ordering needed specific comparator can be used. As example for Employee class if different orderings are needed based on first name, last name, age etc. then we can have different comparators with the implementations for those orderings. |
     | To sort the list when **Comparable** is used we can use **Collections.sort(List)**. | To sort the list when **Comparator** is used, Comparator class has to be explicitly specified as a param in the sort method. **Collections.sort(List, Comparator)** |

     

     Refer

      

     Difference between Comparable and Comparator

      

     to see in detail.

132. ------

133. **How Does HashSet internally works in Java?**

     HashSet internally uses HashMap to store it's elements. But it differs from HashMap on two points.

     - HashSet only stores unique values i.e. no duplicates are allowed.
     - In HashSet we have **add(E e)** method which takes just the element to be added as parameter not the (key, value) pair.

     

     Refer

      

     How HashSet works internally in Java

      

     to know more about the internal implementation of the HashSet.

134. ------

135. **Which Set implementation should be used if you want the insertion order to be maintained?**

     **LinkedHashSet** should be used in this case.

     Refer

      

     LinkedHashSet in Java

      

     to know more about LinkedHashSet in Java.

136. ------

137. **What is LinkedHashSet?**

     LinkedHashSet is also one of the implementation of the Set interface. Actually LinkedHashSet class extends the HashSet and has no other methods of its own.
     LinkedHashSet also stores unique elements just like other implemetations of the Set interface. How LinkedHashSet differs is that **it maintains the insertion-order**; that is *elements in the LinkedHashSet are stored in the sequence in which they are inserted*. Note that insertion order is not affected if an element is re-inserted into the set.

     Refer

      

     LinkedHashSet in Java

      

     to know more about LinkedHashSet in Java.

138. ------

139. **Which Set implementation should be used if you want the elements to be stored in sorted order?**

     **TreeSet** should be used in this case.

     Refer

      

     TreeSet in Java

      

     to know more about TreeSet in Java.

140. ------

141. **What is TreeSet?**

     TreeSet is also one of the implementation of the Set interface like HashSet and LinkedHashSet. TreeSet class implements the **NavigableSet** interface and extends the **AbstractSet** class.
     Just like other implementations of the Set interface HashSet and LinkedHashSet, TreeSet also stores unique elements. How TreeSet differs from other implementations is that **elements in TreeSet are stored in sorted order**. The elements are ordered using their natural ordering or a [comparator](https://www.netjstech.com/2015/10/difference-between-comparable-and-comparator-java.html) can be provided at set creation time to provide custom ordering.
     One more point about TreeSet is, though HashSet and LinkedHashSet allow one null, TreeSet **doesn't allow null as element**.

     Refer

      

     TreeSet in Java

      

     to know more about TreeSet in Java.

142. ------

143. **How to sort elements in different order (other than natural ordering) in TreeSet?**

     By default elements are stored in TreeSet using natural ordering. If you want to sort using different order then you need to provide your own comparator at set creation time. If you have a TreeSet that stores Strings, then to sort the Strings in descending order rather than the natural ordering (which is ascending in case of String) you can write a comparator like this -

     ```
     class CityComparator implements Comparator<String>{
       @Override
       public int compare(String str1, String str2) {
         return str2.compareTo(str1);
       }    
     }
     ```

     This custom comparator has to be provided at the set creation time.

     ```
     Set<String> citySet = new TreeSet<String>(new CityComparator());
     ```

     

     Refer

      

     How to Sort elements in different order in TreeSet

      

     to see the detailed explanation of how to sort elements in different order in TreeSet.

144. ------

145. **How can we sort HashSet in Java?**

     Best way to sort a HashSet is to create a TreeSet using the given HashSet. That will automatically sort the elements as TreeSet is sorted.
     **As Exp.** - If there is a HashSet, citySet a TreeSet can be created using it like this -

     ```
     Set<String> sortedSet = new TreeSet<String>(citySet);
     ```

     

     Refer

      

     How to sort HashSet in Java

      

     to see the detailed explanation of how to sort HashSet in Java.

146. ------

147. **What is EnumSet?**

     EnumSet in Java is a specialized set implementation for use with enum types. All of the elements stored in an enum set must, explicitly or implicitly, come from a single enum type that is specified while creating the set. EnumSet was introduced in Java 5 along with the Enum.
     One of the things to note about EnumSet is that it is an [abstract class](https://www.netjstech.com/2015/04/abstract-class-in-java.html) and uses factory methods to create objects. There are two concrete implementations of EnumSet-

     - RegularEnumSet
     - JumboEnumSet

     

     Refer

      

     EnumSet in Java

      

     to read more about EnumSet in Java.

148. ------

149. **HashSet Vs LinkedHashSet Vs TreeSet in Java**

     

     - **HashSet** is unordered, it will store elements on the basis of the calculated hash that's it.
       **LinkedHashSet** maintains the insertion order of the elements.
       **TreeSet** keeps the element in sorted order. The sorting order,s TreeSet follows by default is known as the natural ordering of the elements.
     - **HashSet as well as LinkedHashSe**t allow null value to be stored. Mind you only one null value is allowed in both HashSet and LinkedHashSet.
       TreeSet does not allow null value, trying to add null to a TreeSet will result in a null pointer exception.
     - For **HashSet and LinkedHashSet** comparison of the elements is done using equals() method. Note that set allows only unique elements, and that uniqueness is maintained by using the equals() method to compare elements.
       **TreeSet** does the comparison of the element using the compareTo (or compare) method, depending on whether sorting is done using [Comparable or Comparator](https://www.netjstech.com/2015/10/difference-between-comparable-and-comparator-java.html).
     - Performance wise **HashSet** is at the top as it has no added baggage of insertion ordering or sorting. If hashing is done correctly HashSet offers constant time performance **O(1)** for the basic operations (add, remove, contains and size).
       For **LinkedHashSet** performance is likely to be just slightly below that of HashSet, due to the added expense of maintaining the linked list.
       **TreeSet** has to sort elements with every insertion so that way performance is slow, but TreeSet provides guaranteed **log(n)** time cost for the basic operations (add, remove and contains) irrespective of the number of elements stored.