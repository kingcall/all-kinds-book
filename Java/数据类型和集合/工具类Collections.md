[TOC]

## Collections 工具类

在一些需要对集合进行特殊操作的时候，但是集合本身并么有为我们提供这样的操作的时候,我们可以尝试使用这个工具类

本文主要介绍Collections的常用方法，例如 Collections.sort()、Collections.shuffle()、Collections.reverse()、Collections.addAll()、Collections.copy()、Collections.binarySearch()、Collections.synchronizedXXX()

还有就是针对Collections.copy() 的方式源码进行讲解，从而避免`IndexOutOfBoundsException: Source does not fit in dest`

### 排序 sort

```java
@Test
public void sort(){
    // Creating an array list
    ArrayList<Integer> numbers = new ArrayList<>();
    // Add elements
    numbers.add(4);
    numbers.add(2);
    numbers.add(3);
    System.out.println("Unsorted ArrayList: " + numbers);
    // Using the sort() method
    Collections.sort(numbers);
    System.out.println("Sorted ArrayList: " + numbers);
}
// 输出结果
Unsorted ArrayList: [4, 2, 3]
Sorted ArrayList: [2, 3, 4]
```



### 打散 shuffle

Shuffle 这个操作在大数据组件里，是非常常见的，在其他的场合下还是比较少见的，你可以认为它是打乱排序，有点随机的意思

```java
@Test
public  void shuffle() {
    // Creating an array list
    ArrayList<Integer> numbers = new ArrayList<>();
    // Add elements
    numbers.add(1);
    numbers.add(2);
    numbers.add(3);
    System.out.println("Sorted ArrayList: " + numbers);
    // Using the shuffle() method
    Collections.shuffle(numbers);
    System.out.println("ArrayList using shuffle: " + numbers);
}
// 输出结果(第一次)
Sorted ArrayList: [1, 2, 3]
ArrayList using shuffle: [3, 2, 1]
// 输出结果(第二次，可能需要多次运行)
Sorted ArrayList: [1, 2, 3]
ArrayList using shuffle: [1, 3, 2]
```

可以看出shuffle 的结果一直在变



### 反转 reverse

```java
@Test
public void reverse() {
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);

    // Using reverse()
    Collections.reverse(numbers);
    System.out.println("Reversed ArrayList1: " + numbers);
}

// 输出结果
ArrayList1: [1, 2]
Reversed ArrayList1: [2, 1]
```



### 填充 fill

```java
@Test
public void fill() {
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);
    // Using fill()
    Collections.fill(numbers, 0);
    System.out.println("ArrayList1 using fill(): " + numbers);
}
// 输出结果
ArrayList1: [1, 2]
ArrayList1 using fill(): [0, 0]
```



### 添加集合 addAll

将一个list 中的元素添加到另外一个list 中去，其实你别看这个方法简单，但是还是很好用的，而且不会出什么问题，不像下面介绍的copy 方法，使用不当还会抛异常,当然下面并没有直接使用Collections.addAll 方法，而是直接使用的集合的addAll 方法，因为第二个参数接受的是一个可变参数而不是集合，但是很多时候我们更常用的实集合

但是下面我们看一下Collections.addAll 方法的源代码

```java
public static <T> boolean addAll(Collection<? super T> c, T... elements) {
    boolean result = false;
    for (T element : elements)
        result |= c.add(element);
    return result;
}
```

其实就是遍历，然后不断添加



```java
@Test
public void addAll() {
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);
    ArrayList<Integer> newNumbers = new ArrayList<>();
    // Using addAll
  Collections.
    newNumbers.addAll(numbers);
    System.out.println("ArrayList2 using addAll(): " + newNumbers);
}
// 输出结果
ArrayList1: [1, 2]
ArrayList2 using addAll(): [1, 2]
```

当然这里需要注意一下，addAll()是添加元素到集合的尾部

```java
@Test
public void addAll() {
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);

    ArrayList<Integer> newNumbers = new ArrayList<>();
    newNumbers.add(4);
    newNumbers.addAll(numbers);
    System.out.println("ArrayList2 using addAll(): " + newNumbers);
}
// 输出结果如下 [1,2] 添加到了4的后面
ArrayList1: [1, 2]
ArrayList2 using addAll(): [4, 1, 2]
```

所以到target 集合(也就是调用addAll 方法的集合)是一个空的集合的时候，我们可以认为这就是一个copy 操作

### 拷贝 copy

```java
@Test
public void copy() {
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);
    ArrayList<Integer> newNumbers = new ArrayList<>();
    // Using copy()
    Collections.copy(newNumbers, numbers);
    System.out.println("ArrayList2 using copy(): " + newNumbers);
}
```

运行结果如下，报错了，既然报错了，那就看看源码，解决一下

```
ArrayList1: [1, 2]
java.lang.IndexOutOfBoundsException: Source does not fit in dest
	at java.util.Collections.copy(Collections.java:558)
	at datastructure.java数据类型.hash.CollectionsUtil.copy(CollectionsUtil.java:104)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
```

下面就是copy 方法的源代码，我的习惯是在看代码之前，看一下代码的注释

```java
/**
 * Copies all of the elements from one list into another.  After the operation, the index of each copied element in the destination list
 * will be identical to its index in the source list.  
 * 拷贝list 中的全部元素到另外一个list中去，在拷贝完成之后目标列表中每个元素的索引将与源列表中其索引相同
 * The destination list must be at least as long as the source list.  If it is longer, the remaining elements in the destination list are unaffected
 * 目标列表必须至少与源列表一样长。 如果更长，则目标列表中的其余元素不受影响
 * This method runs in linear time.
 * 该方法以线性时间运行
 * @param  <T> the class of the objects in the lists 泛型
 * @param  dest The destination list. 目标list
 * @param  src The source list.
 * @throws IndexOutOfBoundsException if the destination list is too small to contain the entire source List.
 * @throws UnsupportedOperationException if the destination list's list-iterator does not support the set operation.
 */
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    int srcSize = src.size();
    if (srcSize > dest.size())
      	// source 不匹配 dest(还不如直接抛出 The destination list must be at least as long as the source 外国人就是这么含蓄) 
        throw new IndexOutOfBoundsException("Source does not fit in dest");

    if (srcSize < COPY_THRESHOLD ||
        (src instanceof RandomAccess && dest instanceof RandomAccess)) {
        for (int i=0; i<srcSize; i++)
            dest.set(i, src.get(i));
    } else {
        ListIterator<? super T> di=dest.listIterator();
        ListIterator<? extends T> si=src.listIterator();
        for (int i=0; i<srcSize; i++) {
            di.next();
            di.set(si.next());
        }
    }
}
```

也就是说我们得到了，如果source 的size 如果大于 dest 的size 的话，则会抛出IndexOutOfBoundsException异常的结论,然后你开始修改你的代码了，一顿操作之后，就成下面这样了

```java
@Test
public void copy() {
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);
  	// 新list的大小是源list的大小+1
    ArrayList<Integer> newNumbers = new ArrayList<>(numbers.size()+1);
    // Using copy()
    Collections.copy(newNumbers, numbers);
    System.out.println("ArrayList2 using copy(): " + newNumbers);
}
```

你以为你修改对了吗，其实是错的，你一运行，还是报上面的错误

```
ArrayList1: [1, 2]
java.lang.IndexOutOfBoundsException: Source does not fit in dest
	at java.util.Collections.copy(Collections.java:558)
	at datastructure.java数据类型.hash.CollectionsUtil.copy(CollectionsUtil.java:104)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
```

你知道问题出在了那里吗，哈哈，我们添加一行代码，你就知道了

```java
@Test
public void copy() {
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);
    ArrayList<Integer> newNumbers = new ArrayList<>(numbers.size()+1);
    System.out.println(newNumbers.size()+"\t"+numbers.size());
}
// 运行结果如下
ArrayList1: [1, 2]
0	2
```

`你以为的你以为不一定是你以为的以为` 这句话用在这里很合适啊，其实主要问题出现在了ArrayList的构造函数上

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}
```

其实和很多集合一样，initialCapacity 这是规定了实现的存储容器大小，例如HashMap 背后的Node<K,V>[] 数组的大小，以及这里的Object[]数组的大小，而size 的含义是集合中当前存储了多少元素，你可以这样认为一个是存储能力，一个是当前实际存储，两个是不一样的概念

所以，我们只要让list 的实际存储元素达到copy 的要求即可，Arrays.asList() 就是一个不错的选择

```java
@Test
public void copy() {
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);
    List<Integer> newNumbers = Arrays.asList(new Integer[numbers.size()]);
    System.out.println(newNumbers.size()+"\t"+numbers.size());
    Collections.copy(newNumbers, numbers);
    System.out.println("ArrayList2 using copy(): " + newNumbers);
}
// 运行结果
ArrayList1: [1, 2]
2	2
ArrayList2 using copy(): [1, 2]
```

更多细节可以看[工具类Arrays]() 和[ArrayList]()

当然这里我们再扩展一下，前面我们说过，在target 是一个空的集合的时候，我们调用copy 和 addAll的效果是一样的

```java
@Test
public void copy() {
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);
    List<Integer> newNumbers = new ArrayList<>();
    newNumbers.addAll(numbers);
    System.out.println(newNumbers.size()+"\t"+numbers.size());
    System.out.println("ArrayList2 using copy(): " + newNumbers);
}
```

总结：在target 集合为空的时候，copy 和addAll() 可以相互替换，推荐使用addAll，因为不容易出错，但是在target不为空的时候，copy 是添加在target 的头部，并且会覆盖对应位置已有元素，而addAll是添加在尾部，不会覆盖已有元素

### 交换 swap

交换集合中，两个指定位置的元素，你说没有这样的操作方法，也不影响什么，你可以先get 然后put

```java
@Test
public void swap() {
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);


    // Using swap()
    Collections.swap(numbers, 0, 1);
    System.out.println("ArrayList1 using swap(): " + numbers);
}
// 输出结果
ArrayList1: [1, 2]
ArrayList1 using swap(): [2, 1]
```



### 二分查找 binarySearch

当然这里有一个前提，那就是数组中的元素已经有序的了，这也是二分查找的前提，因为这里二分查找的实现方式的原因，要求元素是从下到大的排列

```java
@Test
public  void binarySearch() {
    // Creating an ArrayList
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    numbers.add(3);

    // Using binarySearch()
    int pos = Collections.binarySearch(numbers, 3);
    System.out.println("The position of 3 is " + pos);
}
```

当然这里我们可以简单看一下，这里的实现,当然按照我的习惯，先看一下注释和方法说明

```java
/**
 * Searches the specified list for the specified object using the binary search algorithm.  
 * 使用二分查找算法在特定的list中查找特定的值
 * The list must be sorted into ascending order according to the {@linkplain Comparable natural ordering} of its elements (as by the {@link #sort(List)} method) prior to making this call. 
 * 在调用这个方法之前，list 中的元素必须按照升序排序
 * If it is not sorted, the results are undefined.  
 * 如果list 不是有序的，结果是不确定的
 * If the list  contains multiple elements equal to the specified object, there is no guarantee which one will be found.
 * 如果list 中包含多个和查找值想的元素，不能保证返回的实第一个
 * This method runs in log(n) time for a "random access" list (which provides near-constant-time positional access).  
 * 对于可以按照下标访问的集合（实现了RandomAccess），这个方法的时间复杂度是 log(n) 
 * If the specified list does not implement the {@link RandomAccess} interface and is large, this method will do an iterator-based binary search that performs O(n) link traversals and O(log n) element comparisons.
 * 如果没有实现RandomAccess 接口，则采用iiterator-based binary 查找，这里包含两个过程，首先是O(n) 的遍历时间复杂度，然后是O(log n)的查找复杂度
 * @param  <T> the class of the objects in the list
 * @param  list the list to be searched.
 * @param  key the key to be searched for.
 * @return the index of the search key, if it is contained in the list; 如果包含则返回下标
 *         otherwise, <tt>(-(<i>insertion point</i>) - 1)</tt>.  The
 *         <i>insertion point</i> is defined as the point at which the
 *         key would be inserted into the list: the index of the first
 *         element greater than the key, or <tt>list.size()</tt> if all
 *         elements in the list are less than the specified key.  Note
 *         that this guarantees that the return value will be &gt;= 0 if
 *         and only if the key is found.
 * @throws ClassCastException if the list contains elements that are not <i>mutually comparable</i> (for example, strings and
 *         integers), or the search key is not mutually comparable
 *         with the elements of the list.
 */
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
  	// BINARYSEARCH_THRESHOLD=5000
    if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
        return Collections.indexedBinarySearch(list, key);
    else
        return Collections.iteratorBinarySearch(list, key);
}
private static <T> int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {
    int low = 0;
    int high = list.size()-1;
    while (low <= high) {
        int mid = (low + high) >>> 1;
        Comparable<? super T> midVal = list.get(mid);
        int cmp = midVal.compareTo(key);
      	// 这里就决定了元素得从小到大
        if (cmp < 0)
            low = mid + 1;
        else if (cmp > 0)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found
}
```

### frequency 统计次数

```java
@Test
public void frequency() {
    // Creating an ArrayList
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    numbers.add(3);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);

    int count = Collections.frequency(numbers, 2);
    System.out.println("Count of 2: " + count);
}
// 输出结果
ArrayList1: [1, 2, 3, 2]
Count of 2: 2
```



### 交集 disjoint

判断两个集合是否有交集

```java
@Test
public void disjoint() {
    // Creating an ArrayList
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    numbers.add(3);
    numbers.add(2);
    System.out.println("ArrayList1: " + numbers);


    ArrayList<Integer> newNumbers = new ArrayList<>();
    newNumbers.add(5);
    newNumbers.add(6);
    System.out.println("ArrayList2: " + newNumbers);

    boolean value = Collections.disjoint(numbers, newNumbers);
    System.out.println("Two lists are disjoint: " + value);
}
// 输出结果
ArrayList1: [1, 2, 3, 2]
ArrayList2: [5, 6]
Two lists are disjoint: true
```

### 最大最小值

```java
@Test
public void extremeValues() {
    // Creating an ArrayList
    ArrayList<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    numbers.add(3);

    // Using min()
    int min = Collections.min(numbers);
    System.out.println("Minimum Element: " + min);

    // Using max()
    int max = Collections.max(numbers);
    System.out.println("Maximum Element: " + max);
}
// 输出结果
Minimum Element: 1
Maximum Element: 3
```



### 线程安全操作

也就是说，下面的操作可以将原本不是线程安全的集合，转变为线程安全的集合，这里我也给出一个例子,网上有很多Map 的例子，这里我给出一个关于List 的例子

![image-20201203124428196](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/12:44:29-image-20201203124428196.png)



#### 例一 插入操作

```java
// 定义了一个线程类
class SynchroProblem implements Runnable {
    private List<Integer> numList;

    //Constructor
    public SynchroProblem(List<Integer> numList) {
        this.numList = numList;
    }

    @Override
    public void run() {
        System.out.println("in run method");
        for (int i = 0; i < 10; i++) {
            numList.add(i);
            try {
                // introducing some delay
                Thread.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

@Test
public void test() throws Exception {
    List<Integer> numList = new ArrayList<Integer>();
    // Creating three threads
    Thread t1 = new Thread(new SynchroProblem(numList));
    Thread t2 = new Thread(new SynchroProblem(numList));
    Thread t3 = new Thread(new SynchroProblem(numList));
    t1.start();
    t2.start();
    t3.start();
    try {
        t1.join();
        t2.join();
        t3.join();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    // 在这里我们的预期输出是30
    System.out.println("Size of list is " + numList.size());
    //这里是0-9 每个数字输出3次
    for(Integer i : numList){
        System.out.println("num - " + i);
    }
}
```

接下来我们看一下运行结果,size 大小并不是30，而且6 只输出了两次，当你多次运行之后，你会发现每次运行结果可能还不一样，典型的线程安全问题

```
in run method
in run method
in run method
Size of list is 29
num - 0
num - 0
num - 0
num - 1
num - 1
num - 1
num - 2
num - 2
num - 2
num - 3
num - 3
num - 3
num - 4
num - 4
num - 4
num - 5
num - 5
num - 6
num - 6
num - 6
num - 7
num - 7
num - 7
num - 8
num - 8
num - 8
num - 9
num - 9
num - 9
```

如何修正呢，使用我们今天介绍的方法即可

```java
@Test
public void test() throws Exception {
  	// 当然你也可以使用 Vector   
    //List<Integer> numList = new Vector<Integer>();
    List<Integer> numList = Collections.synchronizedList(new ArrayList<Integer>());
    // Creating three threads
    Thread t1 = new Thread(new SynchroProblem(numList));
    Thread t2 = new Thread(new SynchroProblem(numList));
    Thread t3 = new Thread(new SynchroProblem(numList));
    t1.start();
    t2.start();
    t3.start();
    try {
        t1.join();
        t2.join();
        t3.join();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Size of list is " + numList.size());
    for(Integer i : numList){
        System.out.println("num - " + i);
    }
}
// 输出结果
Size of list is 30
```





```java
@Test
public void safe() {
    Random random = new Random();
    class Mt extends Thread {
        int xAnInt;
        List<Integer> list;

        public Mt(int i, List<Integer> list) {
            xAnInt = i;
            this.list = list;
        }

        @Override
        public void run() {
            try {
                TimeUnit.MILLISECONDS.sleep(random.nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            list.add(xAnInt);
        }
    }
    ArrayList<Integer> list = new ArrayList();
    for (int i = 0; i < 1000; i++) {
        new Mt(i, list).start();
    }
    
    System.out.println(list.size());
}
// 输出结果(每次运行都不一样，证明了已经发生了数据错误的情况)
29
```

现在我们对这个代码稍做修改,然后就可以正确运行了

```java
@Test
public void safe() {
    Random random = new Random();
    class Mt extends Thread {
        int xAnInt;
        List<Integer> list;

        public Mt(int i, ArrayList<Integer> list) {
            xAnInt = i;
            this.list = list;
        }

        @Override
        public void run() {
            try {
                TimeUnit.MILLISECONDS.sleep(random.nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            list.add(xAnInt);
        }
    }
    ArrayList<Integer> list = new ArrayList();
    // 其实就是创建了一个新的线程安全的list
    List<Integer> asfeList = Collections.synchronizedList(list);
    for (int i = 0; i < 1000; i++) {
        new Mt(i, list).start();
    }

    System.out.println(list.size());
}
```



#### 例三 遍历操作

因为我们知道，在集合遍历过程中，如果集合被修改了，遍历则将失败，这个Fail-Fast 导致的，但是在多线程的环境中，会导致其他其他线程数据问题，所以我们希望是Fai-Safe,也就是说在遍历过程中，集合被修改了，也可以继续遍历，而不是打断遍历，抛出异常

```java
@Test
public void traverseSafe() {
    List<Integer> numList = new ArrayList<>();
    numList.add(1);
    numList.add(2);
    numList.add(3);
    numList.forEach(ele -> {
        if (ele==2){
            new Thread(()->numList.remove(1)).start();
        }
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(ele);
    });
}
```

运行结果,我么可以看到运行失败了

```java
1
2
java.util.ConcurrentModificationException
	at java.util.ArrayList.forEach(ArrayList.java:1260)
	at datastructure.java数据类型.hash.CollectionsUtil.traverseSafe(CollectionsUtil.java:260)
```

接下来，我们看一下我们今天介绍的解决方案

```java
@Test
public void traverseSafe() {
  	// 当然这里你乙二胺可以使用Vector
    List<Integer> numList = Collections.synchronizedList(new ArrayList<Integer>());
    numList.add(1);
    numList.add(2);
    numList.add(3);
    numList.forEach(ele -> {
        if (ele==2){
            new Thread(()->numList.remove(1)).start();
        }
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(ele);
    });
}
// 运行结果
1
2
3
```

