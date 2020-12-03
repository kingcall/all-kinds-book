### Print Odd-Even Numbers Using Threads And Semaphore Java Program

This Java program prints odd and even numbers alternatively using two threads. One thread prints odd numbers and another thread even numbers. This program makes use of [inter-thread communication](https://www.netjstech.com/2015/07/inter-thread-communiction-wait-notify-java-multi-thread.html) using [Semaphore](https://www.netjstech.com/2016/02/semaphore-in-java-concurrency.html) class which is present in concurrent util package.

- Refer [Print odd-even numbers using threads and wait-notify](https://www.netjstech.com/2016/04/print-odd-even-numbers-using-threads-wait-notify.html) to see how to print odd-even numbers using wait notify.

The **Semaphore** class present in **java.util.concurrent** package is a counting semaphore in which a semaphore, conceptually, maintains a set of permits. Semaphore class has two methods that make use of permits-

- **acquire()**- Acquires a permit from this semaphore, blocking until one is available, or the [thread](https://www.netjstech.com/2015/06/creating-thread-in-java.html) is interrupted. It has another overloaded version **acquire(int permits)**.
- **release()**- Releases a permit, returning it to the semaphore. It has another [overloaded method](https://www.netjstech.com/2015/04/method-overloading-in-java.html) **release(int permits)**.

### Java Program to print odd-even numbers using threads and semaphore

In the Java program there is class **SharedPrinter** whose object is shared between two threads. In this class there is a method **printEvenNum()** for printing even numbers and method **printOddNum()** for printing odd numbers.

These two methods are called by the respective threads **EvenNumProducer** and **OddNumProducer** and these threads communicate using semaphore. Idea is to have 2 semaphores when first is acquired release second, when second is acquired release first. That way shared resource has controlled access and there is inter-thread communication between the threads.

Note that one of the semaphore semEven is initialized with 0 permits that will make sure that even number generating thread doesn't start first.

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class EvenOddSem {

 public static void main(String[] args) {
    SharedPrinter sp = new SharedPrinter();
    // Starting two threads
    ExecutorService executor = Executors.newFixedThreadPool(2);
    executor.execute(new EvenNumProducer(sp, 10));
    executor.execute(new OddNumProducer(sp, 10));
    executor.shutdown();
 }
}

//Shared class used by both threads
class SharedPrinter{
 boolean evenFlag = false;
 // 2 semaphores 
 Semaphore semEven = new Semaphore(0);
 Semaphore semOdd = new Semaphore(1);
 
 //Method for printing even numbers
 public void printEvenNum(int num){
  try {
   semEven.acquire();
  } catch (InterruptedException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
  System.out.println(num);
  semOdd.release(); 
 }
 
 //Method for printing odd numbers
 public void printOddNum(int num){
  try {
   semOdd.acquire();
  } catch (InterruptedException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
  System.out.println(num);
  semEven.release();
   
 }
}

//Thread Class generating Even numbers
class EvenNumProducer implements Runnable{
  SharedPrinter sp;
  int index;
  EvenNumProducer(SharedPrinter sp, int index){
    this.sp = sp;
    this.index = index;
  }
  @Override
  public void run() {
    for(int i = 2; i <= index; i = i+2){
      sp.printEvenNum(i);
    }   
  }  
}

//Thread Class generating Odd numbers
class OddNumProducer implements Runnable{
  SharedPrinter sp;
  int index;
  OddNumProducer(SharedPrinter sp, int index){
    this.sp = sp;
    this.index = index;
  }
  @Override
  public void run() {
    for(int i = 1; i <= index; i = i+2){
      sp.printOddNum(i);
    }
  }
} 
```

**Output**

```
1
2
3
4
5
6
7
8
9
10
```

That's all for this topic **Print Odd-Even Numbers Using Threads And Semaphore Java Program**. If you have any doubt or any suggestions to make please drop a comment. Thanks!