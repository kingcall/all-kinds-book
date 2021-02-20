[TOC]

## java 中的Iterator 和 Iterable

首先我们看一下我们常见的遍历方式

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(1);
list.add(3);
for (int i = 0; i < list.size(); i++) {
    System.out.print(list.get(i) + ",");
}
System.out.println();

Iterator it = list.iterator();
while (it.hasNext()) {
    System.out.print(it.next() + ",");
}
System.out.println();

for (Integer i : list) {
    System.out.print(i + ",");
}
System.out.println();

list.forEach(line -> {
    System.out.print(line + ",");
});
```

第一种就是普通的for循环

第二种为迭代器遍历

第三种为增强for 循环

第四种是for each循环

后面两种方式涉及到Java中的iterator和iterable接口，接下来我们来看看这两个接口的区别以及如何在自定义类中实现for each循环。

### Iterator

开始之前我们先看一下Iterator 接口的定义

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
```

iterator通过以next和 hasNext()两个方法定义了对集合迭代访问的方法，而具体的实现方式依赖于不同的实现类，具体的集合类实现Iterator接口中的方法以实现迭代。

#### Iterator 的意义

Java 5 中加入 Java 语言的最大的便利功能之一，增强的 for 循环，消除了使用 Java 集合的最后一道障碍。以前，开发人员必须手动获得一个 `Iterator` ，使用 `next()` 获得 `Iterator` 指向的对象，并通过 `hasNext()` 检查是否还有更多可用对象。从 Java 5 开始，我们可以随意使用 for 循环的变种，它可以在幕后处理上述所有工作。实际上，这个增强适用于实现 `Iterable` 接口的 *任何对象* ，而不仅仅是 `Collections` 。

无疑 Java 开发人员很喜爱 Java 集合 `Iterator` ，但是您最后一次使用 `Iterator` 接口是什么时候的事情了？可以这么说，大部分时间我们只是将 `Iterator` 随意放到 `for()` 循环或加强 `for()` 循环中，然后就继续其他操作了。

但是进行深入研究后，就会发现 `Iterator` 实际上有两个十分有用的功能。

第一，`Iterator` 支持从源集合中安全地删除对象，只需在 `Iterator` 上调用 `remove()` 即可。这样做的好处是可以避免 `ConcurrentModifiedException` ，这个异常顾名思意：当打开 `Iterator` 迭代集合时，同时又在对集合进行修改。有些集合不允许在迭代时删除或添加元素，但是调用 `Iterator` 的 `remove()` 方法是个安全的做法。

第二， `Iterator` 支持派生的（并且可能是更强大的）兄弟成员。`ListIterator`，只存在于 `List` 中，支持在迭代期间向 `List` 中添加或删除元素，并且可以在 `List` 中双向滚动。

双向滚动特别有用，尤其是在无处不在的 “滑动结果集” 操作中，因为结果集中只能显示从数据库或其他集合中获取的众多结果中的 10 个。它还可以用于 “反向遍历” 集合或列表，而无需每次都从前向后遍历。插入 `ListIterator` 比使用向下计数整数参数 `List.get()`“反向” 遍历 `List` 容易得多。



![The Listiterator interface extends the Java Iterator interface.](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/14:07:32-java-iterator-interface-20210127224929491-20210127224938385.png)



所有的Java 集合都包含iterator()方法，这个方法返回一个iterator的实例，用来遍历集合中的元素

####  Iterator的方法

`Iterator` 接口提供了四个可以对集合元素进行操作的方法

- `hasNext()` - 返回 `true` 如果集合里还有元素
- `next()` - 返回集合里的下一个元素
- `remove()` - 删除集合里的下一个元素，并且通过next 方法返回
- `forEachRemaining()` - 对集合中的剩余元素进行操作



####  Iterator的例子



```java
public class IteratorDemo {
    
    @Test
    public void test() {
        List<String> s = Arrays.asList("a", "b","c");
        Iterator<String> iterator = s.iterator();
        iterator.next();
        iterator.forEachRemaining((x)-> System.out.println(x));
    }

    @Test
    public void test2() {
        // Creating an ArrayList
        ArrayList<Integer> numbers = new ArrayList<>();
        numbers.add(1);
        numbers.add(3);
        numbers.add(2);
        System.out.println("ArrayList: " + numbers);

        // Creating an instance of Iterator
        Iterator<Integer> iterate = numbers.iterator();

        // Using the next() method
        int number = iterate.next();
        System.out.println("Accessed Element: " + number);

        // Using the remove() method
        iterate.remove();
        System.out.println("Removed Element: " + number);

        System.out.print("Updated ArrayList: ");

        // Using the hasNext() method
        while(iterate.hasNext()) {
            // Using the forEachRemaining() method
            iterate.forEachRemaining((value) -> System.out.print(value + ", "));
        }
    }
}
```



**Output**

```
ArrayList: [1, 3, 2]
Acessed Element: 1
Removed Element: 1
Updated ArrayList: 3, 2,
```

在上面的例子中注意一下下面这个写法

```
iterate.forEachRemaining((value) -> System.put.print(value + ", "));
```

这里我们传递一个lambda 表达式作为`forEachRemaining`方法的参数



### Iterable

我们注意到我们在使用Iterator的时候，往往会获取一个Iterator，而这个获取的方式是通过调用Iterator()方法实现，而Iterator()其实是定义自在Iterable接口中的

我们先看一下这个接口定义的描述信息

```java
/**
 * Implementing this interface allows an object to be the target of the "for-each loop" statement. See
 * @param <T> the type of elements returned by the iterator
 *
 * @since 1.5
 * @jls 14.14.2 The enhanced for statement
 */
public interface Iterable<T> {
    Iterator<T> iterator();
  
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
  
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

这个接口定义了我们的可以使用`for-each` 这种遍历方式来遍历该对象里面的元素Iterable接口的源码可以发现其只是返回了一个Iterator对象。

```java
@Test
public void test() {
    List<String> s = Arrays.asList("a", "b","c");
    s.forEach(line -> System.out.println(line));
}
```



### Iterable与Iterator关系

这里有一个问题，为什么不直接将hasNext()，next()方法放在Iterable接口中，其他类直接实现就可以了？而是通过分开实现呢

原因是有些集合类可能不止一种遍历方式，实现了Iterable的类可以再实现多个Iterator内部类，例如`LinkedList`中的`ListItr`和`DescendingIterator`两个内部类，就分别实现了双向遍历和逆序遍历。通过返回不同的`Iterator`实现不同的遍历方式，这样更加灵活。如果把两个接口合并，就没法返回不同的`Iterator`实现类了

`LinkedList`的`ListItr`

```java
 private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);
            nextIndex = index;
        }

        public boolean hasNext() {
            return nextIndex < size;
        }

        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }
        
  }
```

## 总结

1. Java 中一共有四种可以遍历集合的方式
2. Iterator 接口使得我们可以通过Iterator 的方式遍历集合
3. Iterable 接口使得我们可以使用for-each 循环和增强for 循环遍历集合，for-each循环和增强for循环也都是依赖于Iterator迭代器，只不过Java提供的语法糖，Java编译器会将其转化为Iterator迭代器方式遍历
4. Iterable与Iterator的分开始实现是为了不同的实现类可以更加灵活的实现不同的Iterator

