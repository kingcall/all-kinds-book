### Print Odd-Even Numbers Using Threads And wait-notify Java Program

In this post we'll see a Java program to print odd and even numbers sequentially using two threads. One thread generates odd numbers and another even numbers. This Java program makes use of [inter-thread communication using wait, notify, notifyAll](https://www.netjstech.com/2015/07/inter-thread-communiction-wait-notify-java-multi-thread.html) to print odd-even numbers.

- Refer[ Print odd-even numbers using threads and semaphore](https://www.netjstech.com/2016/04/print-odd-even-numbers-using-threads-semaphore.html) to see how to print odd-even numbers using threads and semaphore.

### Java program to print odd-even numbers using threads

There is a class **SharedPrinter** whose object is shared between two threads. In this class there is a method **printEvenNum()** for printing even numbers and method **printOddNum()** for printing odd numbers.

These two methods are called by the respective threads **EvenNumProducer** and **OddNumProducer** and these threads communicate using wait and notify, of course from inside a [synchronized block](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html).

- Refer [Why wait(), notify() and notifyAll() must be called inside a synchronized method or block](https://www.netjstech.com/2015/07/why-wait-notify-and-notifyall-called-from-synchronized-java.html) to know more about the topic.

```
public class EvenOddThreadDemo {
  public static void main(String[] args) {
    // shared class object
    SharedPrinter sp = new SharedPrinter();
    // creating two threads
    Thread t1 = new Thread(new EvenNumProducer(sp, 10));
    Thread t2 = new Thread(new OddNumProducer(sp, 10));
    // starting threads
    t1.start();
    t2.start();
  }
}
// Shared class used by both threads
class SharedPrinter{
  boolean evenFlag = false;
 
  //Method for printing even numbers
  public void printEvenNum(int num){
    synchronized (this) {
      // While condition as mandated to avoid spurious wakeup
      while(!evenFlag){
        try {
          //asking current thread to give up lock
          wait();
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
      }
      System.out.println(num);
      evenFlag = false;
      // Wake up thread waiting on this monitor(lock)
      notify();
    }
  }
 
  //Method for printing odd numbers
  public void printOddNum(int num){
    synchronized (this) {
      // While condition as mandated to avoid spurious wakeup
      while(evenFlag){
        try {
         //asking current thread to give up lock
         wait();
        } catch (InterruptedException e) {
         // TODO Auto-generated catch block
         e.printStackTrace();
        }
      }
      System.out.println(num);
      evenFlag = true;
      // Wake up thread waiting on this monitor(lock)
      notify();   
    }
  }
}

// Thread Class generating Even numbers
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

