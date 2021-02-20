[TOC]

## Arrays工具类

和Collections 一样，主要是提供了对集合的一些特殊操作或者说是扩展了集合的操作方法，Arrays类位于 java.util 包中，主要包含了**操作数组**的各种方法，其实我们知道其他的一些集合例如List、Set  这些集合类还是提供了很多方法的，但是数组它是不属于集合体系下的，它是通过`[]` 进行标识的，它没有提供给用户可操作的任何方法，只提供了一个`length` 属性

如果你直接去看去看这个类的话，你会发现这个类提供的方法特别多就像下面这样，但是你要是仔细看的话，就会发现其实很多方法都是重载方法，针对不同类型的数组提供了不同的方法

![image-20210116103549974](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210116103549974.png)

下面我们就看一下到底有哪些不同的方法，下面就是主要的方法，后面我们针对这些方法进行讲解

```java
asList(T... a) 
binarySearch( Object[] a, Object key)
copyOf(Object[] original, int newLength) 
copyOfRange(Object[] original, int from, int to) 
deepEquals(Object[] a1, Object[] a2)
deepHashCode(Object a[]) 
deepToString(Object[] a) 
deepToString(Object[] a, StringBuilder buf, Set<Object[]> dejaVu) {
equals(Object[] a, Object[] a2)
fill(Object[] a, Object val)
hashCode(char a[])
legacyMergeSort(Object[] a) 
parallelPrefix(Object[] array, Object op)
parallelSetAll(Object[] array, IntToObjectFunction generator) 
parallelSort(Object[] a) 
setAll(Object[] array, IntToObjectFunction generator)
sort(Object[] a)
spliterator(Object[] array) 
spliterator(Object[] array, int startInclusive, int endExclusive)
stream(Object[] array)
stream(Object[] array, int startInclusive, int endExclusive) 
swap(Object[] x, int a, int b)
toString(Object[] a)
```



### asList(T ... a)

将数组或者是可变参数转化为List

刚接触 Java 技术的开发人员可能不知道，Java 语言最初包括数组，是为了应对上世纪 90 年代初期 C++ 开发人员对于性能方面的批评。从那时到现在，我们已经走过一段很长的路，如今，与 Java Collections 库相比，数组不再有性能优势。

例如，若要将数组的内容转储到一个字符串，需要迭代整个数组，然后将内容连接成一个 `String` ；而 Collections 的实现都有一个可用的 `toString()` 实现，可以直接使用。除少数情况外，好的做法是尽快将遇到的任何数组转换成集合。

```java
@Test
public void asList(){
    List<String> list = Arrays.asList(new String[]{"a", "b", "c"});
    List<String> list2 = Arrays.asList("a","b","c");
    System.out.println(list);
    System.out.println(list2);
}
// 输出结果
[a, b, c]
[a, b, c]
```

注意，返回的 `List` 是不可修改的，所以如果尝试向其中添加新元素将抛出一个 `UnsupportedOperationException` 。

而且，由于 `Arrays.asList()` 使用 **可变** 参数表示添加到 `List` 的元素，所以还可以使用它轻松地用以 `new` 新建的对象创建 `List` 。

### sort()

对数组中的元素进行排序

**sort(Object[])** 按照升序，对全部元素进行排序

```java
@Test
public void sort() {
    int[] nums = {2, 5, 0, 4, 6, -10};
    Arrays.sort(nums);
    for (int i : nums) {
        System.out.print(i + " ");
    }
}
// 输出结果
-10 0 2 4 5 6 
```

**sort(Object[] array, int from, int to)** 按照升序，对指定范围内的元素进行排序

```java
@Test
public void sort() {
    int[] nums = {2, 5, 0, 4, 6, -10};
    Arrays.sort(nums,0,4);
    for (int i : nums) {
        System.out.print(i + " ");
    }
}
// 输出结果
0 2 4 5 6 -10 
```



**Arrays.sort(nums, new Comparator<Integer>() )** 使用Comparator 指定排序方向

升序

```java
@Test
public void sort() {
    Integer[] nums = {2, 5, 0, 4, 6, -10};
    Arrays.sort(nums, new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            return o1.compareTo(o2);
        }
    });
    for (int i : nums) {
        System.out.print(i + " ");
    }
}
// 输出结果
-10 0 2 4 5 6 
```

降序

```java
@Test
public void sort() {
    Integer[] nums = {2, 5, 0, 4, 6, -10};
    Arrays.sort(nums, new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            return o2.compareTo(o1);
        }
    });
    for (int i : nums) {
        System.out.print(i + " ");
    }
}
// 输出结果
6 5 4 2 0 -10 
```

### fill(Object[] a, Object val)

可以为数组元素填充相同的值

```java
@Test
public void fill() {
    int[] nums = {2, 5, 0, 4, 6, -10};
    Arrays.fill(nums, 1);
    for (int i : nums) {
        System.out.print(i + " ");
    }
}
// 输出结果
1 1 1 1 1 1
```

**Arrays.fill(Object[] array,int from,int to,Object object)** 可以为指定范围内的元素填充相同的值

```java
@Test
public void fill() {
    int[] nums = {2, 5, 0, 4, 6, -10};
    Arrays.fill(nums, 1,3,100);
    for (int i : nums) {
        System.out.print(i + " ");
    }
}
// 输出结果
2 100 100 4 6 -10 
```



### toString(Object[] array)

返回数组的字符串形式，因为默认输出的时候是内存地址

```java
@Test
public void toStr() {
    int[] nums = {2, 5, 0, 4, 6, -10};
    System.out.println(nums);
    System.out.println(Arrays.toString(nums));
}
// 输出结果
[I@587c290d
[2, 5, 0, 4, 6, -10]
```



### deepToString(Object[][] arrays)

返回多维数组的字符串形式

```java
@Test
public void deepToString() {
    int[][] nums = {{2, 5, 0, 4, 6, -10},{2, 5, 0, 4, 6, -10}};
    System.out.println(nums);
    System.out.println(Arrays.deepToString(nums));
}
// 输出结果
[[I@587c290d
[[2, 5, 0, 4, 6, -10], [2, 5, 0, 4, 6, -10]]
```



### binarySearch(Object[] a, Object key)

采用二分查找方式，查找某个特定的key，如果找到返回下标，找不到则返回-1，需要注意的是二分查找**要求输入是有序的**

**无序的输入**

```java
@Test
public void binarySearch() {
    int bytes = Arrays.binarySearch(new byte[]{6,3,7,9,1,4,5,10}, (byte) 1);
    System.out.println(bytes);
}
// 输出结果
-1
```

可以看出当我们的输入是无序的时候，查找是不准确的，例如 上面的1 是存在的，但是却没有找到

**有序的输入**

```java
@Test
public void binarySearch() {
    int bytes = Arrays.binarySearch(new byte[]{1,2,3,4,5,6,7}, (byte) 3);
    System.out.println(bytes);
}
// 输出结果
2
```



### copyOf(object[] original, int newLength)

把数组复制到另一个数组中并返回结果,我们先看一下这个方法的实现,其实我们看到底层是调用的 System.arraycopy 来实现的

```java
public static int[] copyOf(int[] original, int newLength) {
    int[] copy = new int[newLength];
  	// Math.min(original.length, newLength) 是要拷贝的长度
    System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
    return copy;
}
```

newLength 指的是新数组的长度，如果新数组的长度大于要复制的数组的长度，空的位置默认值来填充

```java
@Test
public void copyOf() {
    int[] ints = Arrays.copyOf(new int[]{1, 2, 3}, 10);
    System.out.println(Arrays.toString(ints)); 
}
// 输出结果
[1, 2, 3, 0, 0, 0, 0, 0, 0, 0]
```

新数组的长度小于老数组的长度，则拷贝对应位置的数据

```java
@Test
public void copyOf() {
    int[] ints = Arrays.copyOf(new int[]{1, 2, 3}, 10);
    System.out.println(Arrays.toString(ints));
    ints = Arrays.copyOf(new int[]{1, 2, 3}, 2);
    System.out.println(Arrays.toString(ints));
}
// 输出结果
[1, 2]
```



### copyOfRange

我们还是先看一下这个方法 的实现，同样还是借助System.arraycopy 实现的

```java
public static int[] copyOfRange(int[] original, int from, int to) {
    int newLength = to - from;
    if (newLength < 0)
        throw new IllegalArgumentException(from + " > " + to);
    int[] copy = new int[newLength];
  	// 特定范围是通过Math.min(original.length - from, newLength)来实现的
    System.arraycopy(original, from, copy, 0,Math.min(original.length - from, newLength));
    return copy;
}
```



```java
@Test
public void copyOfRange() {
    int[] ints = Arrays.copyOfRange(new int[]{9, 8, 7,6,5,4,3,2,1}, 0, 5);
    System.out.println(Arrays.toString(ints));
}
// 输出结果
[9, 8, 7, 6, 5]
```



### parallelPrefix

使用并行计算，并对数组做计算的功能，由于是一个函数式接口，可以作lambda表达式

```java
@Test
public void parallelPrefix() {
    int[] ints = {1, 2, 3, 4, 5};
    //K表示数组第一个值，也就是0号索引，V代表K的下一个索引值，两个索引的值相加
    Arrays.parallelPrefix(ints,(K,V)-> K+V);
    //输出为[1, 3, 6, 10, 15]：流程是1和2相加为3，3和3相加为6，6和4相加为10...以此往后类推
    System.out.println(Arrays.toString(ints));

    int[] Ints = {5,4,3,2,1,0};
    //从1号索引到5号索引之间开始相加数值
    Arrays.parallelPrefix(Ints,1,5,(K,V) -> K + V);
    System.out.println(Arrays.toString(Ints)); //输出为[5, 4, 7, 9, 10, 0]

}
```

### parallelSort

使用并行排序+合并排序算法对数组进行排序* 

```java
@Test
public void parallelSort() {
    int[] ints = {2, 3, 4, 5, 1};
    Arrays.parallelSort(ints);
    //输出为：[1, 2, 3, 4, 5]
    System.out.println(Arrays.toString(ints));
    
    int[] Ints = {2, 3, 4, 5, 1,6,8,7};
    //从1到7号索引之间进行排序
    Arrays.parallelSort(Ints,1,7);
    //输出为：[2, 1, 3, 4, 5, 6, 8, 7]
    System.out.println(Arrays.toString(Ints));
}
```



### stream

Java 中的集合提供了stream 操作，但是数组并没有，所以在这个工具类中提供了相应的方法，我们看到这个方法是1.8 才加进来的

```java
/**
 * Returns a sequential {@link IntStream} with the specified array as its
 * source.
 *
 * @param array the array, assumed to be unmodified during use
 * @return an {@code IntStream} for the array
 * @since 1.8
 */
public static IntStream stream(int[] array) {
    return stream(array, 0, array.length);
}
```



```java
@Test
public void stream() {
    int[] ints = {1, 2, 3,4,5,6,7};
    IntStream stream = Arrays.stream(ints);
    //调用流count方法计算数据总数
    System.out.println(stream.count());

    //从0-3号索引区间开始计算
    IntStream stre = Arrays.stream(ints, 0, 3);
    System.out.println(stre.count());
}
```



## 总结

1. 关于数组的拷贝底层调用的是 System.arraycopy，所以我们除了使用Arrays工具类提供的两个拷贝方法，也可以使用System.arraycopy 方法
2. List、Set  等这些集合类在Java中提供了很多可以操作方法，但是数组它是不属于集合体系下的，它是通过`[]` 进行标识的，它没有提供给用户可操作的任何方法，只提供了一个`length` 属性，所以Arrays工具类提供了很多的方法可以方便的让我们操作数组

























