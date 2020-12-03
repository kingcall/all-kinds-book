### Java ThreadLocal Class With Examples

Usually when there are multiple threads sharing an object we need to synchronize the critical section of the code in order to make it thread safe. ThreadLocal class in Java provides another way of thread-safety apart from [synchronization](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html).



### How ThreadLocal class provides thread safety

ThreadLocal class in Java provides **thread-local variables** where each thread that accesses one (via its get or set method) has its own, independently initialized copy of the variable. Since each and every [thread](https://www.netjstech.com/2015/06/creating-thread-in-java.html) has its own copy of the object so explicit synchronization is not needed to provide thread safety.

### How to create a ThreadLocal variable

Let's see how threadlocal variable is created and what all methods are provided by ThreadLocal class in Java to get and set value of a ThreadLocal variable.

**Creating a ThreadLocal variable**

```
private static final ThreadLocal<String> threadLocalVar = new ThreadLocal<String>();
```

Here I have created a ThreadLocal variable called threadLocalVar which will store a String value.

**Setting and accessing the value**

Once an instance of ThreadLocal class is created, its set method can be used to set a value-

```
threadLocalVar.set("This is a thread local variable");
```



get method is used to read the value, note that get method returns the value in the current thread's copy of this thread-local variable.

```
threadLocalVar.get();
```



### ThreadLocal variable can be accessed globally by a thread

One interesting point about Java ThreadLocal variable is the **global access**. Any thread local variable is global to a thread. It can be accessed anywhere from the [thread](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html). If, from a thread several methods residing in different classes are called, thread local variable will be visible to all those methods. There is no need to pass the thread local variable as a parameter.

### ThreadLocal variable is local to a thread

At the same time any threadlocal variable is **local to a thread**. If there are 10 threads spawned all the 10 threads will have **their own thread local variable**. One thread can not access/modify other thread's Thread Local variables.

### Initial Value of a ThreadLocal variable

ThreadLocal class in Java provides a method called **initialValue()** which can be used to provide initial value to a created ThreadLocal variable.

```
// Atomic integer containing the next thread ID to be assigned
private static final AtomicInteger nextId = new AtomicInteger(0);
// Thread local variable containing each thread's ID
private static final ThreadLocal<Integer> threadId = new ThreadLocal<Integer>() {
  @Override 
  protected Integer initialValue() {
    return nextId.getAndIncrement();
  }
};
```

Note that ThreadLocal class is modified in Java 8 and a new method **withInitial** is added to it with a general form -

```
public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier)
```

Here Supplier is a [functional interface](https://www.netjstech.com/2015/06/functional-interfaces-and-lambda-expression-in-java-8.html) with one method get() and using [lambda expression](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) implementation of this method can be provided. If we use **withInitial** method, what we did for **initialValue()** method -

```
private static final ThreadLocal<Integer> threadId = new ThreadLocal<Integer>() {
  @Override 
  protected Integer initialValue() {
    return nextId.getAndIncrement();
  }
};
```

Can be done like this-

```
 threadId = ThreadLocal.withInitial(()-> {return nextId.getAndIncrement();});
```

### Java ThreadLocal class usage examples

Now when we know what is a ThreadLocal class and how it can be used to create variables which are thread safe, let's see two use cases where it can be used.

1. When we have a requirement to

    

   associate state

    

   with a thread (e.g., a user ID or Transaction ID). That usually happens with a web application that every request going to a servlet has a unique transactionID associated with it.

   **Example code for this requirement**

   ```
   // This class will provide a thread local variable which
   // will provide a unique ID for each thread
   class ThreadId {
     // Atomic integer containing the next thread ID to be assigned
     private static final AtomicInteger nextId = new AtomicInteger(0);
   
     // Thread local variable containing each thread's ID
     private static final ThreadLocal<Integer> threadId =
       ThreadLocal.<Integer>withInitial(()-> {return nextId.getAndIncrement();});
   
     // Returns the current thread's unique ID, assigning it if necessary
     public static int get() {
       return threadId.get();
     }
   }
   
   // In this class thread's run method is executed
   class MyClass implements Runnable{
     @Override
     public void run() {
       System.out.println("Thread " + Thread.currentThread().getName() 
       + " Value - " +  ThreadId.get());
     }
   }
   
   public class ThreadLocalDemo {  
     public static void main(String[] args) {
       MyClass mc1 = new MyClass();
       Thread thread1 = new Thread(mc1, "Thread-1");
       Thread thread2 = new Thread(mc1, "Thread-2");
       Thread thread3 = new Thread(mc1, "Thread-3");
       thread1.start();
       thread2.start();
       thread3.start();
     }
   }
   ```

   **Output**

   ```
   Thread Thread-1 Value - 0
   Thread Thread-2 Value - 1
   Thread Thread-3 Value - 2
   ```

   It can be seen how each thread has a unique ID.

2. Another use case where

    

   ThreadLocal

    

   is useful is when we want to have a thread safe instance and we don't want to use

    

   synchronization

    

   as the performance cost with synchronization is more. One such case is when SimpleDateFormat is used. Since SimpleDateFormat is not thread safe so we have to provide mechanism to make it thread safe.

   First let's see how we can use synchronized to achieve that.

   ```
   public class DateFormatDemo {
     private DateFormat df = new SimpleDateFormat("dd/MM/yyyy");
     public Date convertStringToDate(String dateString) throws ParseException {
     Date result;
     synchronized(df) {
       result = df.parse(dateString);
     }
     return result;
     }  
   }
   ```

   Now let's see an example how ThreadLocal can be used by storing separate instance of SimpleDateFormat for each thread.

   ```
   public class ThreadLocalDemo1 implements Runnable {
     // threadlocal variable is created
     private static final ThreadLocal<SimpleDateFormat> dateFormat = 
                   new ThreadLocal<SimpleDateFormat>(){
       @Override
       protected SimpleDateFormat initialValue(){
         System.out.println("Initializing SimpleDateFormat for - " 
                           + Thread.currentThread().getName() );
         return new SimpleDateFormat("dd/MM/yyyy");
       }
     };
               
     public static void main(String[] args) {
       ThreadLocalDemo1 td = new ThreadLocalDemo1();
       // Two threads are created
       Thread t1 = new Thread(td, "Thread-1");
       Thread t2 = new Thread(td, "Thread-2");
       t1.start();
       t2.start();
     }
   
     @Override
     public void run() {
       System.out.println("Thread run execution started for " 
                         + Thread.currentThread().getName());
       System.out.println("Date formatter pattern is  " 
                        + dateFormat.get().toPattern());
       System.out.println("Formatted date is " 
                        + dateFormat.get().format(new Date()));
     } 
   }
   ```

   **Output**

   ```
   Thread run execution started for Thread-2
   Thread run execution started for Thread-1
   Initializing SimpleDateFormat for - Thread-2
   Initializing SimpleDateFormat for - Thread-1
   Date formatter pattern is  dd/MM/yyyy
   Date formatter pattern is  dd/MM/yyyy
   Formatted date is 26/07/2015
   Formatted date is 26/07/2015
   ```

   If you notice here SimpleDateFormat instance is created and initialized for each thread.

**Points to note**

1. ThreadLocal class in Java provides another alternative to thread safety, apart from synchronization.
2. With ThreadLocal each and every thread has its own thread local variable. One thread cannot access/modify other thread's ThreadLocal variables.
3. ThreadLocal instances are typically private static fields in classes that wish to associate state with a thread.

That's all for this topic **Java ThreadLocal Class With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!