### Java Stream API Examples

In the post [Java Stream API Tutorial](https://www.netjstech.com/2016/11/stream-api-in-java-8.html) we have already got an introduction of Stream API. A Stream can be defined as a sequence of elements supporting sequential and parallel **aggregate operations**. Using these aggregation operations we can create a **pipeline**. Some of the aggregation operations provided are collect, concat, count, distinct, filter, forEach, limit, map, max, min, reduce, sorted. In this post we’ll see some **Java stream API examples** using these operations and also create pipeline consisting sequence of aggregate operations.



### Java Stream API count() method example

count method returns the count of elements in the given stream.

Note that this is a special case of a [reduction](https://www.netjstech.com/2017/01/reduction-operations-in-java-stream-api.html) and it is a **terminal operation**.

```
List<Integer> myList = Arrays.asList(7, 18, 10, 24, 17, 5);  
long count = myList.stream().count();
System.out.println("Total elements in the list " + count);
```

This code snippet will give the count of the elements in the List.

Now if you want to get the count of the elements greater than 10 you can create a pipeline where you first **filter** on the predicate that you want those elements of the list whose value is greater than 10 and then count those elements.

```
List<Integer> myList = Arrays.asList(7, 18, 10, 24, 17, 5); 
long count = myList.stream().filter(i -> i > 10).count();
System.out.println("Total elements in the list with value greater than 10 " + count);
```

### Java Stream API concat() method example

concat() method in Java Stream creates a lazily concatenated stream whose elements are all the elements of the first stream followed by all the elements of the second stream.

```
List<String> myList = Arrays.asList("1", "2", "3", "4", "5");
  
String[] arr1 = { "a", "b", "c", "d" };
// concatenating two streams
Stream<String> stream = Stream.concat(myList.stream(), Arrays.stream(arr1));
stream.forEach(System.out::print);
```

**Output**

```
12345abcd
```

Here you can see the concatenated stream is returned. If you are wondering what is this **System.out::print** refer [Method reference in Java 8](https://www.netjstech.com/2015/06/method-reference-in-java-8.html). You may also want to read about [forEach statement in Java 8](https://www.netjstech.com/2016/09/foreach-statement-in-java-8.html).

Since parameters of the concat operations are streams so all the aggregation operations can be applied to them too. As example if there are two lists having name and you want a merged list with all the names that start with “A” that can be done as follows–

```
List<String> nameList1 = Arrays.asList("Ram", "Amit", "Ashok", "Manish", "Rajat");
  
List<String> nameList2 = Arrays.asList("Anthony", "Samir", "Akash", "Uttam");
  
String[] arr1 = { "a", "b", "c", "d" };
// concatenating two streams
Stream<String> stream = Stream.concat(nameList1.stream().filter(n -> n.startsWith("A")), nameList2.stream().filter(n -> n.startsWith("A")));

stream.forEach(System.out::println);
```

### Java Stream API distinct() method example

Returns a stream consisting of the distinct elements (according to Object.equals(Object)) of this stream.

Using **distinct method** of the Java Stream API, duplicate elements from a collection like list can be removed very easily by creating a pipeline where distinct method will return a stream having distinct elements only which can later be collected in a list using collect method.

```
List<Integer> myList = Arrays.asList(7, 18, 10, 7, 10, 24, 17, 5);
  
System.out.println("Original list: " + myList);
List<Integer> newList = myList.stream().distinct().collect(Collectors.toList());

System.out.println("new List : " + newList);
```

### Java Stream API filter() method example

filter method returns a stream consisting of the elements of this stream that match the given predicate.

Here note that Predicate is a functional interface and can be implemented as a lambda expression. In the above examples we have already used filter method.

As an example let’s say we have a list of names and we want to print names which doesn’t start with “A”.

```
List<String> nameList = Arrays.asList("Ram", "Amit", "Ashok", "Manish", "Rajat");
  
nameList.stream().filter(n -> !n.startsWith("A")).collect(Collectors.toList()).forEach(System.out::println);
```

**Output**

```
Ram
Manish
Rajat
```

### Java Stream API limit() method example

Returns a stream consisting of the elements of this stream, truncated to be no longer than maxSize in length.

If you want 10 random numbers, then you can use limit method with the int stream.

```
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
```

### Java Stream API map() method example

Returns a stream consisting of the results of applying the given function to the elements of this stream. So, whatever function is provided is applied on all the elements of the stream. Note that this is an **intermediate operation**.

**As Example** – If you have a list of salaries and you want to increase it by 10%.

```
List<Integer> myList = Arrays.asList(7000, 5000, 4000, 24000, 17000, 6000);
  
myList.stream().map(i -> (i+ i * 10/100)).forEach(System.out::println);
```

- Refer [Map operation in Java Stream API](https://www.netjstech.com/2017/01/map-operation-in-java-stream-api.html) to read about mapping operations in Java stream API.

### findFirst() and findAny() methods in Java Stream API

- **findFirst()**- Returns an [Optional](https://www.netjstech.com/2016/01/optional-class-in-java-8.html) describing the first element of this stream, or an empty Optional if the stream is empty. If the stream has no encounter order (List or Array wil be ordered, where as set or map won’t), then any element may be returned.
- **findAny()**- Returns an Optional describing some element of the stream, or an empty Optional if the stream is empty. The behavior of this operation is explicitly nondeterministic; it is free to select any element in the stream. This is to allow for maximal performance in parallel operations; the cost is that multiple invocations on the same source may not return the same result. (If a stable result is desired, use findFirst() instead.)

```
List<String> nameList = Stream.of("amy", "nick", "margo", "desi");
Optional<String> name = nameList.stream().findFirst();
System.out.println("First Name " + name);

name = nameList.parallelStream().findAny();
System.out.println("First Name " + name);
```

**Output**

```
First Name Optional[amy]
First Name Optional[margo]
```

You can see in case of **findFirst()** method, first element of the list is displayed. Even with [parallelStream](https://www.netjstech.com/2017/01/parallel-stream-in-java-stream-api.html), findFirst() will give the first element.

Whereas in case of **findAny()** method any random element is picked. You can see that findAny() method is used with parallelStream here.

### max and min methods in Java Stream API

- **max**- Returns the maximum element of this stream according to the provided Comparator.
- **min**- Returns the minimum element of this stream according to the provided Comparator.

max and min are also **reduction operations**. Both of them are **terminal operations**.

```
List<Integer> myList = Arrays.asList(7000, 5000, 4000, 24000, 17000, 6000);
// Obtain a Stream to the array list.
Stream<Integer> myStream = myList.stream();
Optional<Integer> val = myStream.min(Integer::compare);
if(val.isPresent()){
 System.out.println("minimum value in the list " + val.get());
}  
Optional<Integer> val1 = myList.stream().max(Integer::compare);
if(val1.isPresent()){
 System.out.println("maximum value in the list " + val1.get());
}
```

Note that here Optional class is used. To know more about Optional class refer [Optional class in Java 8](https://www.netjstech.com/2016/01/optional-class-in-java-8.html).

### Java Stream API sorted() method example

sorted method returns a stream consisting of the elements of this stream, sorted according to natural order or there is another variant where custom [comparator](https://www.netjstech.com/2015/10/difference-between-comparable-and-comparator-java.html) can be provided.

```
List<Integer> myList = Arrays.asList(7000, 5000, 4000, 24000, 17000, 6000);
myList.stream().sorted().forEach(System.out::println);
```

### Summary Statistics classes

A state object for collecting statistics such as count, min, max, sum, and average. There are different SummaryStatistics classes in Java Stream API like **IntSummaryStatistics**, **DoubleSummaryStatistics**, **LongSummaryStatistics**.

**As example –**

```
List<Integer> myList = Arrays.asList(7, 5, 4, 24, 17, 6);
IntSummaryStatistics stats = myList.stream().collect(Collectors.summarizingInt(i-> i));

System.out.println("Sum - " + stats.getSum());
System.out.println("Count " + stats.getCount());
System.out.println("Average " + stats.getAverage());
System.out.println("Max " + stats.getMax());
System.out.println("Min " + stats.getMin());
```

Here **Collectors.summarizingInt** method is used which applies an int-producing mapping function to each input element, and returns summary statistics for the resulting values.

In place of

```
IntSummaryStatistics stats = myList.stream().collect(Collectors.summarizingInt(i-> i));
```

Using **mapToInt** method it can also be written as -

```
IntSummaryStatistics stats = myList.stream().mapToInt(i -> i).summaryStatistics();
```

That's all for this topic **Java Stream API Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!