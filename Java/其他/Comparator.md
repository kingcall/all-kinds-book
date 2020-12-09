[TOC]

## Comparator

### 为什么有的时候可以直接调用 Collections.sort 方法

我们知道，当你不提供Comparator 作为参数的时候，这个时候我们要求被排序的对象要实现Comparable 接口，这个我们已经学习了很多了，例如

```java
class People  {
    public String name;
    public int age;

    public People(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

	@Test
    public void comparatorUsage() {
        List<People> peopleList = new ArrayList<>();
        peopleList.add(new People("b", 2));
        peopleList.add(new People("a", 1));
        Collections.sort(peopleList);

    }
```

例如当上面的代码中，People类没有实现Comparable 接口的时候，你就会看到下面的错误，这个时候你只要让People实现Comparable 接口即可，如果People不是你写的代码或者你不想实现Comparable ，这个时候你就可以使用Comparator，这个前面也讲过

![image-20201209182548868](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201209182548868.png)

但是有时候你会看到类似下面的代码缺可以执行，这是为什么呢,这是因为, String 类本身已经实现了 Comparable接口，所以我们在这里就可以得到一个结论，那就是如果不提供Comparator ，那么被排序的对象必须实现Comparable接口

```java
    static List<String> testList = null;

    @BeforeAll
    public static void setUp() {
        testList = new ArrayList<>();
        testList.add("d");
        testList.add("a");
        testList.add("c");
        testList.add("b");
        System.out.println("====================Original List========================");
        testList.forEach(System.out::println);
    }

    @Test
    @Order(1)
    public void defaultSort() {
        Collections.sort(testList);
        System.out.println("====================Default List========================");
        testList.forEach(System.out::println);
    }
```



### Comparator.naturalOrder 和 Comparator.reverseOrder

很多时候我们会面临这样的场景，那就是排序逻辑不变，一会儿根据升序排序，一会根据降序排序，这个时候如果我们的Comparable 中的排序逻辑可以满足上面的排序，就是排序类型(升序还是降序)是不满足的，这个时候我们就可以配合Comparator，来改变原来默认的排序类型(其实就是升序)



### nullsFirst 和 nullsLast

首先我们看一下引入这两个方法的原因是什么

```java
public class JavaComparators {
    static List<String> testList = null;

    @BeforeAll
    public static void setUp() {
        testList = new ArrayList<>();
        testList.add("d");
        testList.add("a");
        testList.add("c");
        testList.add("b");
        System.out.println("====================Original List========================");
        testList.forEach(System.out::println);
    }
    @Test
    public void defaultSort() {
        testList.add(null);
        Collections.sort(testList);
        System.out.println("====================Default List========================");
        testList.forEach(System.out::println);
    }
}
```

接下来，你就会收到如下错误，那就是`NullPointerException`,这是应为我们对在排序的过程中，调用对象的方法或者属性。这个时候就会抛出空指针异常

```
====================Original List========================
d
a
c
b
java.lang.NullPointerException
	at java.util.ComparableTimSort.binarySort(ComparableTimSort.java:262)
	at java.util.ComparableTimSort.sort(ComparableTimSort.java:189)
	at java.util.Arrays.sort(Arrays.java:1312)
```

这个时候我们可以根据业务的特点，选择nullsFirst 还是nullsLast

```java
    @Test
    public void nullsFirst(){
        testList.add(null);
        System.out.println("============== nullsFirst ==============");
        Collections.sort(testList,Comparator.nullsFirst(Comparator.naturalOrder()));
        testList.forEach(System.out::println);
        System.out.println("============== nullsLast ==============");
        Collections.sort(testList,Comparator.nullsLast(Comparator.naturalOrder()));
        testList.forEach(System.out::println);
    }
```

输出结果如下，因为这里我们选择则是自然排序，可以看出在原来排序的基础上，将null 值放到了合适的地方，这里需要注意的是Comparator.nullsLast和Comparator.nullsFirst方法依然需要Comparator作为参数，这个时候我们就可以将我们在前面学习的两种Comparator中的一个作为参数传递给它

```
====================Original List========================
d
a
c
b

============== nullsFirst ==============
null
a
b
c
d
============== nullsLast ==============
a
b
c
d
null
```



### comparing

Comparator.comparing(keyExtractor,keyComparator)接受两个参数

第一个是keyExtractor，你可以把它认为是两个需要比较对象的比较部分，说白了就是提取出需要比较的部分，在这里我需要比较的是名字和年龄拼接起来的一个字段

第二个是Comparator，则是你需要定义比较的逻辑，本来Comparator中compare的参数直接是需要比较的两个对象，这里则是keyExtractor的输出

```java
    @Test
    public void comparing() {
        List<People> peopleList = new ArrayList<>();
        peopleList.add(new People("b", 2));
        peopleList.add(new People("a", 1));

        Collections.sort(peopleList, Comparator.comparing(
                (people) -> people.getName() + people.getAge(),
                (o1, o2) -> o1.compareTo(o2)));
        System.out.println("====================Reverse List========================");
        peopleList.forEach(System.out::println);
    }
```

输出结果

```
====================Reverse List========================
Student{name='a', age=1}
Student{name='b', age=2}
```



当然comparing 方法还有另外一种形式，那就是只提供keyExtractor,因为这个时候它会给你提供一个默认的Comparator

就是keyExtractor提取出来的key，然后对key 调用进行compareTo 方法作为Comparator

```java
    public static <T, U extends Comparable<? super U>> Comparator<T> comparing(Function<? super T, ? extends U> keyExtractor)
    {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
    }
```

**但是这里有个注意的地方时，你提取出来的key必须是实现Comparable 接口的，因为它接下来要调用key 的compareTo 方法**

````java
    @Test
    public void comparing2() {
        List<People> peopleList = new ArrayList<>();
        peopleList.add(new People("b", 2));
        peopleList.add(new People("a", 1));

        Collections.sort(peopleList, Comparator.comparing(
                (people) -> people.getName() + people.getAge()));
        System.out.println("====================Reverse List========================");
        peopleList.forEach(System.out::println);

    }
````



输出结果

```
====================Reverse List========================
Student{name='a', age=1}
Student{name='b', age=2}
```

### Comparator的正常实现

```java
class MyComparator implements Comparator<People> {
    @Override
    public int compare(People o1, People o2) {
        return (this.name + this.age).compareTo(o.name + age);
    }
}

```

然后在需要使用的地方，直接 `new MyComparator` 即可 

### Comparator的匿名内部类的实现

```java
    @Test
    public void comparatorUsageByAnonymous() {
        List<People> peopleList = new ArrayList<>();
        peopleList.add(new People("b", 2));
        peopleList.add(new People("a", 1));

        Collections.sort(peopleList, new Comparator<People>() {
            @Override
            public int compare(People o1, People o2) {
                return (o1.name + o1.age).compareTo(o2.name + o2.age);
            }
        });
        System.out.println("====================Reverse List========================");
        peopleList.forEach(System.out::println);
    }
```

上面的这个实现和People实现Comparable的下面这种方式是等价的

```java
    @Override
    public int compareTo(Student o) {
        return (this.name + this.age).compareTo(o.name + age);
    }
```

需要注意的是，匿名内部类往往是在这个Comparator只使用一次的时候才使用，所以如果你要多次使用还是要以类的方式将其提取出来

### Comparator的Lambda 表达式的实现

```java
    public void comparatorUsageByLambda() {
        List<People> peopleList = new ArrayList<>();
        peopleList.add(new People("b", 2));
        peopleList.add(new People("a", 1));
        
        Collections.sort(peopleList, (o1, o2) -> (o1.name + o1.age).compareTo(o2.name + o2.age));
        System.out.println("====================Reverse List========================");
        peopleList.forEach(System.out::println);
    }
```



