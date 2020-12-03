### Java Stream API Interview Questions And Answers

In this post Java Stream API Interview Questions and answers are listed. This compilation will help the Java developers in preparing for their interviews.

1. **What is Stream API in Java?**

   Stream API is added in Java 8 and works very well in conjunction with lambda expressions. You can create a pipeline of stream operations to manipulate data by performing operations like search, filter, sort, count, map etc.

   Read more about Stream API in Java

    

   here

   .

2. ------

3. **What is stream in Stream API?**

   A stream can be visualized as a pipeline. A stream pipeline consists of a source (which might be an array, a collection, a generator function, an I/O channel, etc.), zero or more intermediate operations (which transform a stream into another stream, such as filter(Predicate)), and a terminal operation (which produces a result or side-effect, such as count() or forEach(Consumer)).

   Read more about Stream API in Java

    

   here

   .

4. ------

5. **Explain stream operations with an example?**

   In this example let's take an ArrayList as an input. There are two operations - take only those elements of the list which are greater than 5 and then sort the result. After that print the elements of the list.

   ```
   // Creating the list
   List<Integer> numList = Arrays.asList(34, 6, 3, 12, 65, 1, 8);
   numList.stream().filter((n) -> n > 5).sorted().forEach(System.out::println); 
   ```

   Here ArrayList is the data source for the stream and there are two intermediate operations–

   - **filter**- Filter condition here is; take only those elements of the list which are greater than 5.
   - **sorted**- sort that filtered output of the last stream.

   Terminal operation here is forEach statement (provided in Java 8) which iterates the sorted result and displays them. Read more about forEach statement in Java 8

    

   here

   .

6. ------

7. **How many types of Stream operations are there?**

   Stream operations are divided into intermediate and terminal operations, and are combined to form stream pipelines.

   - **Intermediate operations** return a new stream. They are always lazy; executing an intermediate operation does not actually perform any filtering, but instead creates a new stream that, when traversed, contains the elements of the initial stream that match the given predicate.
   - **Terminal operations** such as Stream.forEach or IntStream.sum, may traverse the stream to produce a result or a side-effect. After the terminal operation is performed, the stream pipeline is considered consumed, and can no longer be used.

   See some Stream API examples

    

   here

   .

8. ------

9. **What are Stateless and Stateful operations in Java stream?**

   Intermediate operations are further divided into stateless and stateful operations.

   - **Stateless operations**, such as filter and map, retain no state from previously seen element when processing a new element, each element can be processed independently of operations on other elements.
   - **Stateful operations**, such as distinct and sorted, may incorporate state from previously seen elements when processing new elements. Stateful operations may need to process the entire input before producing a result. For example, one cannot produce any results from sorting a stream until one has seen all elements of the stream.

   See some Stream API examples

    

   here

   .

10. ------

11. **What is Parallel Stream in Java Stream API?**

    You can execute streams in serial or in parallel. When a stream executes in parallel, the Java runtime partitions the stream into multiple sub-streams.

    **As example** - Collection has methods Collection.stream() and Collection.parallelStream(), which produce sequential and parallel streams respectively.

    Read more about parallel stream

     

    here

    .

12. ------

13. **What is the benefit of using parallel stream?**

    When parallel stream is used the Java runtime partitions the stream into multiple sub-streams. This parallel execution of data, with each sub-stream running in a separate thread, will result in increase in performance.

    Read more about parallel stream

     

    here

    .

14. ------

15. **Can you use streams with primitives?**

    Streams work only on object references. They can’t work on primitive types so you have two options to use primitives.

    - You can wrap primitive types into a wrapper object. As example Stream<Integer>, Stream<Long> or Stream<Double>.
    - Second and better option is to use primitive specializations of Stream like **IntStream**, **LongStream**, and **DoubleStream** that can store primitive values.
    - **As example** - IntStream is = IntStream.of(3, 4, 5, 6);

    Read more about Primitive type streams in Java

     

    here

    .

16. ------

17. **How can you transform Stream to primitive type Stream?**

    Stream interface provides methods **mapToInt**, **mapToDouble** and **mapToLong** that can be used to transform stream of objects to a stream of primitive types.

    **As example**- If you have a list of employee objects and you want to get the maximum salary. In that case you can take the salary field and use mapToInt method to get a stream of primitive types. Then you can use max method on that primmitive type stream.

    ```
    OptionalInt maxSalary = empList.parallelStream().mapToInt(e -> e.getSalary()).max();
    ```

    Read more about Primitive type streams in Java Stream API

     

    here

    .

18. ------

19. **What are Reduction Operations in Java Stream API?**

    Stream API contains many terminal operations (such as average, sum, min, max, and count) that return one value by combining the contents of a stream. These operations are called reduction operations because these operations reduce the stream to a single non-stream value.

    Read more about Reduction Operations in Java Stream API

     

    here

    .

20. ------

21. **What are Map operation in Java Stream API?**

    Map operations are used to do the element mapping from one stream to another. Map operation will return a stream consisting of the results of applying the given function to the elements of this stream. So, whatever function is provided is applied on all the elements of the stream.

    Since new stream is returned map operation is an intermediate operation.

    Read more about Map operation in Java Stream API

     

    here

    .

22. ------

23. **What is a mutable reduction operation?**

    A mutable reduction operation can be defined as an operation that accumulates input elements into a mutable result container, such as a Collection or StringBuilder.

    Read more about Reduction operation in Java Stream API

     

    here

    .

24. ------

25. **What is a collect method in Java stream?**

    Using collect method you can store the result of the stream operation into a collection. This is a terminal operation.

    **As example** - If you have employee objects and you want a list having names of all the employees you can use the toList method of the Collectors class.

    ```
    List<String> nameList = empList.stream().map(Employee::getName).collect(Collectors.toList());
    ```

    Read more about Collecting in Java Stream API

     

    here

    .

26. ------

27. **What is flatMap() method in Java?**

    In mapping operation the given function is applied to all the elements of the stream. Where as flattening a structure, means bringing all the nested structures at the same level.

    **As example** if you have a list of Strings, list<String> like - [[“a”, “b”, “c”], [“c”, “d”], [“c”, “e”, “f”]] then flattening it will bring everything to the same level and the structure you will have be like this-

    ```
    [“a”, “b”, “c”, “c”, “d”, “c”, “e”, “f”]
    ```

    **flatMap()** method means you are bringing both of them together, function will be applied to all the elements of the stream and then it will be flatten to have a single level structure.

    Read more about FlatMap in Java

     

    here

    .

28. ------

29. **What is a Spliterator in Java?**

    Spliterators, like iterators, are for traversing the elements of a source. Spliterator can split the source and iterate the splitted parts in parallel. That way a huge data source can be divided into small sized units that can be traversed and processed in parallel.

    You can also use spliterator even if you are not using parallel execution.

    Read more about Spliterator in Java

     

    here

    .