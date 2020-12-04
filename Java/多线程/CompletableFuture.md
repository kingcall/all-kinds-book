### CompletableFuture in Java With Examples

CompletableFuture in Java was added along with other notable features like [lambda expression](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) and [Stream API in Java 8](https://www.netjstech.com/2016/11/stream-api-in-java-8.html).

CompletableFuture is used for **asynchronous computation**, where the code is executed as a non-blocking call in a separate thread and the result is made available when it is ready.

**Table of contents**

1. [Java CompletableFuture a step ahead from Future Interface](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#completablefuture)
2. [Java CompletableFuture API – Async Variants](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#completablefutureasync)
3. [Java CompletableFuture constructor](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#completablefutureconstructor)
4. [CompletableFuture Java examples](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#completablefutureexamples)
5. [Difference between thenApply() and thenCompose() methods](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#thenApplyvsthenCompose)
6. [Combining two independent CompletableFutures](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#completablefuturecombine)
7. [CompletableFuture Java example](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#completablefuturejava)
8. [Exception handling with CompletableFuture in Java](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#completablefutureexceptionhandling)
9. [CompletableFuture exception handling with exceptionally example](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#completablefutureexceptionally)
10. [CompletableFuture exception handling with handle example](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#completablefuturehandle)
11. [CompletableFuture exception handling with whenComplete example](https://www.netjstech.com/2018/11/completablefuture-in-java-with-examples.html#completablefuturewhenComplete)



### Java CompletableFuture a step ahead from Future Interface

Future which was added in Java 5 also represents the result of an asynchronous computation. Problem with [Future in Java](https://www.netjstech.com/2016/04/callable-and-future-in-java-concurrency.html) is that the API is not that extensive you can just check whether the task is completed or cancelled using **isDone()** and **isCancelled()** method. For getting the result there is get() method which is blocking or you have an option for timed wait. There is also no provision for a callback method which can be called once the task completes.

CompletableFuture class in Java which implements **Future** interface and **CompletionStage** interface tries to address these issues. This class provides methods like **runAsync()** and **supplyAsync()** that run a task asynchronously. But the biggest advantage of CompletableFuture class in Java is *its ability to run a task as a series of stages* (behavior this class gets from implementing CompletionStage) where each stage runs as a possible asynchronous computation, that performs an action or computes a value when another CompletionStage completes.

Using CompletionStages you can create a single CompletableFuture as a chain of stages of CompletionStage where each stage runs when another CompletionStage completes.

For example suppose you are writing a word count program as a non-blocking asynchronous computation.

```
CompletableFuture<String> fileData = readFile(file);
CompletableFuture<Map<String, Integer>> count = fileData.thenApply(WordCount::getCount);
```

### Java CompletableFuture API – Async Variants

In CompletableFuture API most of the methods have three variants where one of them is blocking and two are asynchronous (methods suffixed with Async). Choose the method as per your requirement.

- **thenApply(Function<? super T,? extends U> fn)**- Returns a new CompletionStage that, when this stage completes normally, is executed with this stage's result as the argument to the supplied function.
- **thenApplyAsync(Function<? super T,? extends U> fn)**- Returns a new CompletionStage that, when this stage completes normally, is executed using this stage's default asynchronous execution facility, with this stage's result as the argument to the supplied function. Default asynchronous execution generally is a task running in the ForkJoinPool.commonPool()
- **thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)**- Returns a new CompletionStage that, when this stage completes normally, is executed using the supplied [Executor](https://www.netjstech.com/2016/04/executor-and-executorservice-in-java-concurrency.html), with this stage's result as the argument to the supplied function.

### Java CompletableFuture constructor

In CompletableFuture class there is one constructor.

- **CompletableFuture()**- Creates a new incomplete CompletableFuture.

As you can see the description says incomplete CompletableFuture, so creating a CompletableFuture using this constructor and trying to get its value using get() method will block forever as the get() method waits for this future to complete and then returns its result.

```
CompletableFuture<String> cf = new CompletableFuture<>();
String value = cf.get();
```

You will have to transition this CompletableFuture to a completed state using **complete()** method.

```
CompletableFuture<String> cf = new CompletableFuture<>();
cf.complete("Hello");
String value = cf.get();
System.out.println("Value- " + value);
```

### CompletableFuture Java examples

1- Let’s start with a simple example where a new CompletableFuture is returned that is already completed with the given value.

```
String str = "Hello";
CompletableFuture<String> cf = CompletableFuture.completedFuture(str);
if(cf.isDone()) {
  System.out.println("Value- " + cf.get());
}
```

**Output**

```
Value- Hello
```

2- Running an asynchronous task using runAsync(Runnable runnable) method. This method returns a CompletableFuture<Void>.

```
CompletableFuture<Void> cf = CompletableFuture.runAsync(()->{
  System.out.println("Task executing asynchronously");
});

System.out.println("Value- " + cf.get());
```

**Output**

```
Task executing asynchronously
Value- null
```

3- runAsync() is fine for running asynchronous computations but it doesn't return value. If you want to return a new CompletableFuture with a value then you can use **supplyAsync(Supplier<U> supplier)** method. Here U is the type of value obtained by calling the given Supplier.

```
CompletableFuture cf = CompletableFuture.supplyAsync(()->{
 return "Hello";
});
System.out.println("Value- " + cf.get());
```

**Output**

```
Value- Hello
```

4- Let’s add a new stage to create a chain.

```
CompletableFuture<String> cf = CompletableFuture.supplyAsync(()->{
    return "Hello";
}).thenApply(value-> value.toUpperCase());

System.out.println("Value- " + cf.get());
```

**Output**

```
Value- HELLO
```

Here **thenApply(Function<? super T,? extends U> fn)** method is used. The current stage (thenApply() method) is executed with previous stage's result as the argument to the supplied function and it returns a new CompletionStage.

5- Using the Async variant of the method where an Executor is passed. Note that with the Async variant, method is asynchronously executed in a separate thread obtained from the Executor or from the ForkJoinPool.commonPool() based on the Async variant used.

```
ExecutorService executor = Executors.newFixedThreadPool(2);
CompletableFuture<String> cf = CompletableFuture.supplyAsync(()->{
    return "Hello";
}).thenApplyAsync(value-> value.toUpperCase(), executor);

System.out.println("Value- " + cf.get());
executor.shutdown();
```

6- Using **thenAccept()** method if there is no value to return from the stage. There is also **thenRun()** method which doesn’t return value and takes Runnable as argument.

```
CompletableFuture.supplyAsync(()->{
  return "Hello";
}).thenAccept(value-> {
  System.out.println("Value- " + value);
});
```

### Difference between thenApply() and thenCompose() methods in CompletableFuture

In the Java CompletableFuture class there are two methods thenApply() and thenCompose() with a very little difference and it often confuses people.

**thenApply()**- Returns a new CompletionStage where the type of the result is based on the argument to the supplied function of thenApply() method.

**thenCompose()**- Returns a new CompletionStage where the type of the result is same as the type of the previous stage.

For getting the difference between thenApply() and thenCompose() methods consider the following code.

```
CompletableFuture<CompletableFuture<String>> cf = CompletableFuture.supplyAsync(()->{
  return "Hello";
}).thenApply(value-> {
  String str = value.toUpperCase();
  return CompletableFuture.completedFuture(str);
});
System.out.println("Value- " + cf.get().get());
```

If you see here value returned by the CompletableFuture.supplyAsync method is of type CompletableFuture<String> and taking that as argument in thenApply there is another CompletableFuture returned which makes the return value as the nested layer of CompletableFuture (CompletableFuture<CompletableFuture<String>>). The structure is not flattened.

Now consider the same code with thenCompose() method.

```
CompletableFuture<String> cf = CompletableFuture.supplyAsync(()->{
  return "Hello";
}).thenCompose(value-> {
  String str = value.toUpperCase();
  return CompletableFuture.completedFuture(str);
});
System.out.println("Value- " + cf.get());
```

As you can see here the structure is flattened because thenCompose() returns a result having the type same as previous stage.

### Combining two independent CompletableFutures

There is a **thenCombine()** method that can be used if you want to combine two independent CompletableFutures in a way that when both of the CompletableFutures finish, you want to execute some logic with the results of both.

- **thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)**- Returns a new CompletionStage that, when this and the other given stage both complete normally, is executed with the two results as arguments to the supplied function.

```
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
  return "Combining two CompletableFutures";
});

CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
  return "and getting a new CompletableFuture";
});

CompletableFuture<String> result = future1.thenCombine(future2, (str1, str2) -> str1 + " " + str2);
System.out.println("Value- " + result.get());
```

**Output**

```
Value- Combining two CompletableFutures and getting a new CompletableFuture
```

### CompletableFuture Java example

Here is another example with method calls and a bit more complex than the simple examples we have seen till now. In the example first method fetches the list of users, in the second method user names are changed to upper case. modified list is then returned.

```
public class CFDemo {
  public static void main(String[] args) {    
    CFDemo cfDemo = new CFDemo();    
    try {
      // blocking call
      cfDemo.getUsers();
    } catch (ExecutionException | InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }
    
  public void getUsers() throws ExecutionException, InterruptedException{
    CompletableFuture<List<User>> userList = CompletableFuture.supplyAsync(() -> {
      return getListOfUsers();
    }).thenCompose(users-> {            
      List<User> upperCaseList = null;
      try {
        upperCaseList = users.get().stream().map(
                      user->{
                          user.setFirstName(user.getFirstName().toUpperCase());
                          user.setLastName(user.getLastName().toUpperCase());
                          return user;
                      }).collect(Collectors.toList());
      } catch (InterruptedException | ExecutionException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
      }
      return CompletableFuture.completedFuture(upperCaseList);
    });
        
    userList.get().forEach(System.out::println);
  }
    
  // Dummy method for adding List of Users
  private CompletableFuture<List<User>> getListOfUsers() {
    List<User> users = new ArrayList<User>();
    users.add(new User("Jack", "Reacher", "abc@xyz.com"));    
    users.add(new User("Remington", "Steele", "rs@cbd.com"));
    users.add(new User("Laura", "Holt", "lh@cbd.com"));
    users.add(new User("Jonathan", "Raven", "jr@sn.com"));
    users.add(new User("Tom", "Hanson", "th@jd.com"));
    users.add(new User("Alexander", "Scott", "as@is.com"));
    users.add(new User("Jim", "Phelps", "jp@mi.com"));
    return CompletableFuture.completedFuture(users);
  }
}
```

**Output**

```
JACK REACHER abc@xyz.com
REMINGTON STEELE rs@cbd.com
LAURA HOLT lh@cbd.com
JONATHAN RAVEN jr@sn.com
TOM HANSON th@jd.com
ALEXANDER SCOTT as@is.com
JIM PHELPS jp@mi.com
```

### Exception handling with CompletableFuture in Java

If an exception is thrown at any of the stage with in the chain of CompletionStages the execution stops with in that stage and exception is thrown. For exception handling with CompletableFuture there are three methods handle, whenComplete and exceptionally.

Out of these three, two methods **handle** and **whenComplete** are executed regardless of exception thrown or not. Exception is passed as an argument is these methods which will not be null in case exception is thrown. Using that null check you can write your exception handling code.

**Exceptionally** supports computation only when the triggering stage throws an exception. This method also gives a chance to return a replacement result in case of exception.

### CompletableFuture exception handling with exceptionally example

```
String str = null;
CompletableFuture<String> value = CompletableFuture.supplyAsync(() -> {
  if (str == null)
    throw new IllegalArgumentException("Invalid String value passed " + str);
  return str;
}).exceptionally(exp -> {
  System.out.println("Exception thrown with message - " + exp.getMessage());
  return "";
});
```

**Output**

```
Exception thrown with message - java.lang.IllegalArgumentException: Invalid String value passed null
Value-
```

When string is not null, exception is not thrown so exceptionally() won’t be called.

```
String str = "Hello";
CompletableFuture<String> value = CompletableFuture.supplyAsync(() -> {
  if (str == null)
    throw new IllegalArgumentException("Invalid String value passed " + str);
  return str;
}).exceptionally(exp -> {
  System.out.println("Exception thrown with message - " + exp.getMessage());
  return "";
});
```

**Output**

```
Value- Hello
```

### CompletableFuture exception handling with handle example

```
String str = null;
CompletableFuture<String> value = CompletableFuture.supplyAsync(() -> {
  if (str == null)
    throw new IllegalArgumentException("Invalid String value passed " + str);
  return str;
}).handle((s, exp) -> {
  if(exp != null) {
    System.out.println("Exception thrown with message - " + exp.getMessage());
    s = "";
  }
  return s;
});
```

**Output**

```
Exception thrown with message - java.lang.IllegalArgumentException: Invalid String value passed null
Value- 
```

When string is not null exception is not thrown but handle method still gets called.

```
String str = "Hello";
CompletableFuture<String> value = CompletableFuture.supplyAsync(() -> {
  if (str == null)
    throw new IllegalArgumentException("Invalid String value passed " + str);
  return str;
}).handle((s, exp) -> {
  System.out.println("In handle method..");
  if(exp != null) {
    System.out.println("Exception thrown with message - " + exp.getMessage());
    s = "";
  }
  return s;
});
System.out.println("Value- " + value.get());
```

**Output**

```
In handle method..
Value- Hello
```

### CompletableFuture exception handling with whenComplete example

Method whenComplete preserves the result of the triggering stage instead of computing a new one.

```
String str = null;
CompletableFuture<String> value = CompletableFuture.supplyAsync(() -> {
  if (str == null)
    throw new IllegalArgumentException("Invalid String value passed " + str);
  return str;
}).whenComplete((s, exp) -> {
  System.out.println("in whenComplete method");
  if(exp != null) {
    System.out.println("Exception thrown with message - " + exp.getMessage());
    //s = "";
  }
});
```

**Output**

```
in whenComplete methodException in thread "main" 
Exception thrown with message - java.lang.IllegalArgumentException: Invalid String value passed null
java.util.concurrent.ExecutionException: java.lang.IllegalArgumentException: Invalid String value passed null
 at java.base/java.util.concurrent.CompletableFuture.reportGet(CompletableFuture.java:395)
 at java.base/java.util.concurrent.CompletableFuture.get(CompletableFuture.java:1999)
 at org.nets.program.CFDemo.main(CFDemo.java:27)
Caused by: java.lang.IllegalArgumentException: Invalid String value passed null
 at org.nets.program.CFDemo.lambda$0(CFDemo.java:18)
 at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1700)
 at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.exec(CompletableFuture.java:1692)
 at java.base/java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:290)
 at java.base/java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1603)
 at java.base/java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:177)
```

**Reference:** https://docs.oracle.com/javase/10/docs/api/java/util/concurrent/CompletableFuture.html
https://docs.oracle.com/javase/10/docs/api/java/util/concurrent/CompletionStage.html

That's all for this topic **CompletableFuture in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!