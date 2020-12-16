[TOC]



对于很多 Java 开发人员来说，Java Collections API 是标准 Java 数组及其所有缺点的一个非常需要的替代品。将 Collections 主要与 `ArrayList` 联系到一起本身没有错，但是对于那些有探索精神的人来说，这只是 Collections 的冰山一角。

##### 关于本系列

您觉得自己懂 Java 编程？事实上，大多数程序员对于 Java 平台都是浅尝则止，只学习了足以完成手头上任务的知识而已。在[本系列中](https://developer.ibm.com/zh/series/5-things-you-didnt-know-about/)，Ted Neward 深入挖掘 Java 平台的核心功能，揭示一些鲜为人知的事实，帮助您解决最棘手的编程挑战。

虽然 `Map` （以及它的常用实现 `HashMap` ）非常适合名-值对或键-值对，但是没有理由让自己局限于这些熟悉的工具。可以使用适当的 API，甚至适当的 Collection 来修正很多易错的代码。

本文是系列中的第二篇文章，也是专门讨论 Collections 的 7 篇文章中的第一篇文章，之所以花这么大的篇幅讨论 Collections，是因为这些集合在 Java 编程中是如此重要。首先我将讨论做每件事的最快（但也许不是最常见）的方式，例如将 `Array` 中的内容转移到 `List` 。然后我们深入探讨一些较少人知道的东西，例如编写定制的 Collections 类和扩展 Java Collections API。

## 1. Collections 比数组好

刚接触 Java 技术的开发人员可能不知道，Java 语言最初包括数组，是为了应对上世纪 90 年代初期 C++ 开发人员对于性能方面的批评。从那时到现在，我们已经走过一段很长的路，如今，与 Java Collections 库相比，数组不再有性能优势。

例如，若要将数组的内容转储到一个字符串，需要迭代整个数组，然后将内容连接成一个 `String` ；而 Collections 的实现都有一个可用的 `toString()` 实现。

除少数情况外，好的做法是尽快将遇到的任何数组转换成集合。于是问题来了，完成这种转换的最容易的方式是什么？事实证明，Java Collections API 使这种转换变得容易，如清单 1 所示：

##### 清单 1. ArrayToList

```
import java.util.*;

public class ArrayToList
{
    public static void main(String[] args)
    {
        // This gives us nothing good
        System.out.println(args);

        // Convert args to a List of String
        List<String> argList = Arrays.asList(args);

        // Print them out
        System.out.println(argList);
    }
}
```



注意，返回的 `List` 是不可修改的，所以如果尝试向其中添加新元素将抛出一个 `UnsupportedOperationException` 。

而且，由于 `Arrays.asList()` 使用 *varargs* 参数表示添加到 `List` 的元素，所以还可以使用它轻松地用以 `new` 新建的对象创建 `List` 。

## 2. 迭代的效率较低

将一个集合（特别是由数组转化而成的集合）的内容转移到另一个集合，或者从一个较大对象集合中移除一个较小对象集合，这些事情并不鲜见。

您也许很想对集合进行迭代，然后添加元素或移除找到的元素，但是不要这样做。

在此情况下，迭代有很大的缺点：

- 每次添加或移除元素后重新调整集合将非常低效。
- 每次在获取锁、执行操作和释放锁的过程中，都存在潜在的并发困境。
- 当添加或移除元素时，存取集合的其他线程会引起竞争条件。

可以通过使用 `addAll` 或 `removeAll` ，传入包含要对其添加或移除元素的集合作为参数，来避免所有这些问题。

## 3. 用 for 循环遍历任何 Iterable

Java 5 中加入 Java 语言的最大的便利功能之一，增强的 for 循环，消除了使用 Java 集合的最后一道障碍。

以前，开发人员必须手动获得一个 `Iterator` ，使用 `next()` 获得 `Iterator` 指向的对象，并通过 `hasNext()` 检查是否还有更多可用对象。从 Java 5 开始，我们可以随意使用 for 循环的变种，它可以在幕后处理上述所有工作。

实际上，这个增强适用于实现 `Iterable` 接口的 *任何对象* ，而不仅仅是 `Collections` 。

清单 2 显示通过 `Iterator` 提供 `Person` 对象的孩子列表的一种方法。 这里不是提供内部 `List` 的一个引用 （这使 `Person` 外的调用者可以为家庭增加孩子 — 而大多数父母并不希望如此）， `Person` 类型实现 `Iterable` 。这种方法还使得 for 循环可以遍历所有孩子。

##### 清单 2. 增强的 for 循环：显示孩子

```
// Person.java
import java.util.*;

public class Person
    implements Iterable<Person>
{
    public Person(String fn, String ln, int a, Person... kids)
    {
        this.firstName = fn; this.lastName = ln; this.age = a;
        for (Person child : kids)
            children.add(child);
    }
    public String getFirstName() { return this.firstName; }
    public String getLastName() { return this.lastName; }
    public int getAge() { return this.age; }

    public Iterator<Person> iterator() { return children.iterator(); }

    public void setFirstName(String value) { this.firstName = value; }
    public void setLastName(String value) { this.lastName = value; }
    public void setAge(int value) { this.age = value; }

    public String toString() {
        return "[Person: " +
            "firstName=" + firstName + " " +
            "lastName=" + lastName + " " +
            "age=" + age + "]";
    }

    private String firstName;
    private String lastName;
    private int age;
    private List<Person> children = new ArrayList<Person>();
}

// App.java
public class App
{
    public static void main(String[] args)
    {
        Person ted = new Person("Ted", "Neward", 39,
            new Person("Michael", "Neward", 16），
            new Person("Matthew", "Neward", 10));

        // Iterate over the kids
        for (Person kid : ted)
        {
            System.out.println(kid.getFirstName());
        }
    }
}
```

在域建模的时候，使用 `Iterable` 有一些明显的缺陷，因为通过 `iterator()` 方法只能那么 “隐晦” 地支持一个那样的对象集合。但是，如果孩子集合比较明显， `Iterable` 可以使针对域类型的编程更容易，更直观。

## 4. 经典算法和定制算法

您是否曾想过以倒序遍历一个 `Collection` ？对于这种情况，使用经典的 Java Collections 算法非常方便。

在上面的 [清单 2](https://developer.ibm.com/zh/articles/j-5things2/#清单-2-增强的-for-循环：显示孩子) 中，`Person` 的孩子是按照传入的顺序排列的；但是，现在要以相反的顺序列出他们。虽然可以编写另一个 for 循环，按相反顺序将每个对象插入到一个新的 `ArrayList` 中，但是 3、4 次重复这样做之后，就会觉得很麻烦。

在此情况下，清单 3 中的算法就有了用武之地：

##### 清单 3. ReverseIterator

```
public class ReverseIterator
{
    public static void main(String[] args)
    {
        Person ted = new Person("Ted", "Neward", 39,
            new Person("Michael", "Neward", 16），
            new Person("Matthew", "Neward", 10));

        // Make a copy of the List
        List<Person> kids = new ArrayList<Person>(ted.getChildren());
        // Reverse it
        Collections.reverse(kids);
        // Display it
        System.out.println(kids);
    }
}
```

显示更多

`Collections` 类有很多这样的 “算法”，它们被实现为静态方法，以 `Collections` 作为参数，提供独立于实现的针对整个集合的行为。

而且，由于很棒的 API 设计，我们不必完全受限于 `Collections` 类中提供的算法 — 例如，我喜欢不直接修改（传入的 Collection 的）内容的方法。所以，可以编写定制算法是一件很棒的事情，例如清单 4 就是一个这样的例子：

##### 清单 4. ReverseIterator 使事情更简单

```
class MyCollections
{
    public static <T> List<T> reverse(List<T> src)
    {
        List<T> results = new ArrayList<T>(src);
        Collections.reverse(results);
        return results;
    }
}
```

显示更多

## 5. 扩展 Collections API

以上定制算法阐释了关于 Java Collections API 的一个最终观点：它总是适合加以扩展和修改，以满足开发人员的特定目的。

例如，假设您需要 `Person` 类中的孩子总是按年龄排序。虽然可以编写代码一遍又一遍地对孩子排序（也许是使用 `Collections.sort` 方法），但是通过一个 `Collection` 类来自动排序要好得多。

实际上，您甚至可能不关心是否每次按固定的顺序将对象插入到 `Collection` 中（这正是 `List` 的基本原理）。您可能只是想让它们按一定的顺序排列。

`java.util` 中没有 `Collection` 类能满足这些需求，但是编写一个这样的类很简单。只需创建一个接口，用它描述 `Collection` 应该提供的抽象行为。对于 `SortedCollection` ，它的作用完全是行为方面的。

##### 清单 5. SortedCollection

```
public interface SortedCollection<E> extends Collection<E>
{
    public Comparator<E> getComparator();
    public void setComparator(Comparator<E> comp);
}
```



编写这个新接口的实现简直不值一提：

##### 清单 6. ArraySortedCollection

```
import java.util.*;

public class ArraySortedCollection<E>
    implements SortedCollection<E>, Iterable<E>
{
    private Comparator<E> comparator;
    private ArrayList<E> list;

    public ArraySortedCollection(Comparator<E> c)
    {
        this.list = new ArrayList<E>();
        this.comparator = c;
    }
    public ArraySortedCollection(Collection<? extends E> src, Comparator<E> c)
    {
        this.list = new ArrayList<E>(src);
        this.comparator = c;
        sortThis();
    }

    public Comparator<E> getComparator() { return comparator; }
    public void setComparator(Comparator<E> cmp) { comparator = cmp; sortThis(); }

    public boolean add(E e)
    { boolean r = list.add(e); sortThis(); return r; }
    public boolean addAll(Collection<? extends E> ec)
    { boolean r = list.addAll(ec); sortThis(); return r; }
    public boolean remove(Object o)
    { boolean r = list.remove(o); sortThis(); return r; }
    public boolean removeAll(Collection<?> c)
    { boolean r = list.removeAll(c); sortThis(); return r; }
    public boolean retainAll(Collection<?> ec)
    { boolean r = list.retainAll(ec); sortThis(); return r; }

    public void clear() { list.clear(); }
    public boolean contains(Object o) { return list.contains(o); }
    public boolean containsAll(Collection <?> c) { return list.containsAll(c); }
    public boolean isEmpty() { return list.isEmpty(); }
    public Iterator<E> iterator() { return list.iterator(); }
    public int size() { return list.size(); }
    public Object[] toArray() { return list.toArray(); }
    public <T> T[] toArray(T[] a) { return list.toArray(a); }

    public boolean equals(Object o)
    {
        if (o == this)
            return true;

        if (o instanceof ArraySortedCollection)
        {
            ArraySortedCollection<E> rhs = (ArraySortedCollection<E>)o;
            return this.list.equals(rhs.list);
        }

        return false;
    }
    public int hashCode()
    {
        return list.hashCode();
    }
    public String toString()
    {
        return list.toString();
    }

    private void sortThis()
    {
        Collections.sort(list, comparator);
    }
}
```



这个实现非常简陋，编写时并没有考虑优化，显然还需要进行重构。但关键是 Java Collections API 从来无意将与集合相关的任何东西定死。它总是需要扩展，同时也鼓励扩展。

当然，有些扩展比较复杂，例如 `java.util.concurrent` 中引入的扩展。但是另一些则非常简单，只需编写一个定制算法，或者已有 `Collection` 类的简单的扩展。

扩展 Java Collections API 看上去很难，但是一旦开始着手，您会发现远不如想象的那样难。

## 结束语

和 Java Serialization 一样，Java Collections API 还有很多角落等待有人去探索 — 正因为如此，我们还不准备结束这个话题。在 *[5 件事](https://developer.ibm.com/zh/series/5-things-you-didnt-know-about/)* 系列的下一篇文章中，将可以看到用 Java Collections API 做更多事情的 5 种新的方式。



`java.util` 中的 Collections 类旨在通过取代数组提高 Java 性能。如您在[第 1 部分](https://developer.ibm.com/zh/articles/j-5things2/) 中了解到的，它们也是多变的，能够以各种方式定制和扩展，帮助实现优质、简洁的代码。

Collections 非常强大，但是很多变：使用它们要小心，滥用它们会带来风险。

## 1. List 不同于数组

Java 开发人员常常错误地认为 `ArrayList` 就是 Java 数组的替代品。Collections 由数组支持，在集合内随机查找内容时性能较好。与数组一样，集合使用整序数获取特定项。但集合不是数组的简单替代。

要明白数组与集合的区别需要弄清楚 *顺序* 和 *位置* 的不同。例如，`List` 是一个接口，它保存各个项被放入集合中的 *顺序* ，如清单 1 所示：

##### 清单 1. 可变键值

```
import java.util.*;

public class OrderAndPosition
{
    public static <T> void dumpArray(T[] array)
    {
        System.out.println("=============");
        for (int i=0; i<array.length; i++)
            System.out.println("Position " + i + ": " + array[i]);
    }
    public static <T> void dumpList(List<T> list)
    {
        System.out.println("=============");
        for (int i=0; i<list.size(); i++)
            System.out.println("Ordinal " + i + ": " + list.get(i));
    }

    public static void main(String[] args)
    {
        List<String> argList = new ArrayList<String>(Arrays.asList(args));

        dumpArray(args);
        args[1] = null;
        dumpArray(args);

        dumpList(argList);
        argList.remove(1);
        dumpList(argList);
    }
}
```

显示更多

当第三个元素从上面的 `List` 中被移除时，其 “后面” 的各项会上升填补空位。很显然，此集合行为与数组的行为不同（事实上，从数组中移除项与从 `List` 中移除它也不完全是一回事儿 — 从数组中 “移除” 项意味着要用新引用或 null 覆盖其索引槽）。

## 2. 令人惊讶的 `Iterator` ！

无疑 Java 开发人员很喜爱 Java 集合 `Iterator` ，但是您最后一次使用 `Iterator` 接口是什么时候的事情了？可以这么说，大部分时间我们只是将 `Iterator` 随意放到 `for()` 循环或加强 `for()` 循环中，然后就继续其他操作了。

但是进行深入研究后，您会发现 `Iterator` 实际上有两个十分有用的功能。

第一，`Iterator` 支持从源集合中安全地删除对象，只需在 `Iterator` 上调用 `remove()` 即可。这样做的好处是可以避免 `ConcurrentModifiedException` ，这个异常顾名思意：当打开 `Iterator` 迭代集合时，同时又在对集合进行修改。有些集合不允许在迭代时删除或添加元素，但是调用 `Iterator` 的 `remove()` 方法是个安全的做法。

第二， `Iterator` 支持派生的（并且可能是更强大的）兄弟成员。`ListIterator`，只存在于 `List` 中，支持在迭代期间向 `List` 中添加或删除元素，并且可以在 `List` 中双向滚动。

双向滚动特别有用，尤其是在无处不在的 “滑动结果集” 操作中，因为结果集中只能显示从数据库或其他集合中获取的众多结果中的 10 个。它还可以用于 “反向遍历” 集合或列表，而无需每次都从前向后遍历。插入 `ListIterator` 比使用向下计数整数参数 `List.get()`“反向” 遍历 `List` 容易得多。

## 3. 并非所有 `Iterable` 都来自集合

Ruby 和 Groovy 开发人员喜欢炫耀他们如何能迭代整个文本文件并通过一行代码将其内容输出到控制台。通常，他们会说在 Java 编程中完成同样的操作需要很多行代码：打开 `FileReader`，然后打开 `BufferedReader`，接着创建 `while()` 循环来调用 `getLine()`，直到它返回 *null* 。当然，在 `try/catch/finally` 块中必须要完成这些操作，它要处理异常并在结束时关闭文件句柄。

这看起来像是一个没有意义的学术上的争论，但是它也有其自身的价值。

他们（包括相当一部分 Java 开发人员）不知道并不是所有 `Iterable` 都来自集合。`Iterable` 可以创建 `Iterator` ，该迭代器知道如何凭空制造下一个元素，而不是从预先存在的 `Collection` 中盲目地处理：

##### 清单 2. 迭代文件

```
// FileUtils.java
import java.io.*;
import java.util.*;

public class FileUtils
{
    public static Iterable<String> readlines(String filename)
        throws IOException
    {
        final FileReader fr = new FileReader(filename);
        final BufferedReader br = new BufferedReader(fr);

        return new Iterable<String>() {
            public <code>Iterator</code><String> iterator() {
                return new <code>Iterator</code><String>() {
                    public boolean hasNext() {
                        return line != null;
                    }
                    public String next() {
                        String retval = line;
                        line = getLine();
                        return retval;
                    }
                    public void remove() {
                        throw new UnsupportedOperationException();
                    }
                    String getLine() {
                        String line = null;
                        try {
                            line = br.readLine();
                        }
                        catch (IOException ioEx) {
                            line = null;
                        }
                        return line;
                    }
                    String line = getLine();
                };
            }
        };
    }
}

//DumpApp.java
import java.util.*;

public class DumpApp
{
    public static void main(String[] args)
        throws Exception
    {
        for (String line : FileUtils.readlines(args[0]))
            System.out.println(line);
    }
}
```

显示更多

此方法的优势是不会在内存中保留整个内容，但是有一个警告就是，它不能 `close()` 底层文件句柄（每当 `readLine()` 返回 null 时就关闭文件句柄，可以修正这一问题，但是在 `Iterator` 没有结束时不能解决这个问题）。

## 4. 注意可变的 `hashCode()`

`Map` 是很好的集合，为我们带来了在其他语言（比如 Perl）中经常可见的好用的键/值对集合。JDK 以 `HashMap` 的形式为我们提供了方便的 `Map` 实现，它在内部使用哈希表实现了对键的对应值的快速查找。但是这里也有一个小问题：支持哈希码的键依赖于可变字段的内容，这样容易产生 bug，即使最耐心的 Java 开发人员也会被这些 bug 逼疯。

假设清单 3 中的 `Person` 对象有一个常见的 `hashCode()` （它使用 `firstName` 、`lastName` 和 `age` 字段 — 所有字段都不是 final 字段 — 计算 `hashCode()` ），对 `Map` 的 `get()` 调用会失败并返回 `null` ：

##### 清单 3. 可变 `hashCode()` 容易出现 bug

```
// Person.java
import java.util.*;

public class Person
    implements Iterable<Person>
{
    public Person(String fn, String ln, int a, Person... kids)
    {
        this.firstName = fn; this.lastName = ln; this.age = a;
        for (Person kid : kids)
            children.add(kid);
    }

    // ...

    public void setFirstName(String value) { this.firstName = value; }
    public void setLastName(String value) { this.lastName = value; }
    public void setAge(int value) { this.age = value; }

    public int hashCode() {
        return firstName.hashCode() & lastName.hashCode() & age;
    }

    // ...

    private String firstName;
    private String lastName;
    private int age;
    private List<Person> children = new ArrayList<Person>();
}

// MissingHash.java
import java.util.*;

public class MissingHash
{
    public static void main(String[] args)
    {
        Person p1 = new Person("Ted", "Neward", 39);
        Person p2 = new Person("Charlotte", "Neward", 38);
        System.out.println(p1.hashCode());

        Map<Person, Person> map = new HashMap<Person, Person>();
        map.put(p1, p2);

        p1.setLastName("Finkelstein");
        System.out.println(p1.hashCode());

        System.out.println(map.get(p1));
    }
}
```

显示更多

很显然，这种方法很糟糕，但是解决方法也很简单：永远不要将可变对象类型用作 `HashMap` 中的键。

## 5. `equals()` 与 `Comparable`

在浏览 Javadoc 时，Java 开发人员常常会遇到 `SortedSet` 类型（它在 JDK 中唯一的实现是 `TreeSet` ）。因为 `SortedSet` 是 `java.util` 包中唯一提供某种排序行为的 `Collection` ，所以开发人员通常直接使用它而不会仔细地研究它。清单 4 展示了：

##### 清单 4. `SortedSet` ，我很高兴找到了它！

```
import java.util.*;

public class UsingSortedSet
{
    public static void main(String[] args)
    {
        List<Person> persons = Arrays.asList(
            new Person("Ted", "Neward", 39),
            new Person("Ron", "Reynolds", 39),
            new Person("Charlotte", "Neward", 38),
            new Person("Matthew", "McCullough", 18)
        );
        SortedSet ss = new TreeSet(new Comparator<Person>() {
            public int compare(Person lhs, Person rhs) {
                return lhs.getLastName().compareTo(rhs.getLastName());
            }
        });
        ss.addAll(perons);
        System.out.println(ss);
    }
}
```

显示更多

使用上述代码一段时间后，可能会发现这个 `Set` 的核心特性之一：它不允许重复。该特性在 `Set` Javadoc 中进行了介绍。`Set` 是不包含重复元素的集合。更准确地说，set 不包含成对的 e1 和 e2 元素，因此如果 e1.equals(e2)，那么最多包含一个 null 元素。

但实际上似乎并非如此 — 尽管[清单 4](https://developer.ibm.com/zh/articles/j-5things3/#清单-4-code-sortedset-code-，我很高兴找到了它！) 中没有相等的 `Person` 对象（根据 `Person` 的 `equals()` 实现），但在输出时只有三个对象出现在 `TreeSet` 中。

与 set 的有状态本质相反，`TreeSet` 要求对象直接实现 `Comparable` 或者在构造时传入 `Comparator`，它不使用 `equals()` 比较对象；它使用 `Comparator/Comparable` 的 `compare` 或 `compareTo` 方法。

因此存储在 `Set` 中的对象有两种方式确定相等性：大家常用的 `equals()` 方法和 `Comparable/Comparator` 方法，采用哪种方法取决于上下文。

更糟的是，简单的声明两者相等还不够，因为以排序为目的的比较不同于以相等性为目的的比较：可以想象一下按姓排序时两个 `Person` 相等，但是其内容却并不相同。

一定要明白 `equals()` 和 `Comparable.compareTo()` 两者之间的不同 — 实现 `Set` 时会返回 0。甚至在文档中也要明确两者的区别。

## 结束语

Java Collections 库中有很多有用之物，如果您能加以利用，它们可以让您的工作更轻松、更高效。但是发掘这些有用之物可能有点复杂，比如只要您不将可变对象类型作为键，您就可以用自己的方式使用 `HashMap` 。