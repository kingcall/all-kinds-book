### Printing Numbers in Sequence Using Threads Java Program

This post shows how you can print numbers in sequence using three threads in Java. If there are three threads thread1, thread2 and thread3 then numbers should be printed alternatively by these threads like this.

```
thread1 - 1
thread2 - 2
thread3 – 3
thread1 - 4
thread2 - 5
thread3 – 6
...
...
...
```

### Print numbers in sequence using three threads in Java

While printing numbers in sequence using threads trick is to use modulo division to check which thread can print the number and which threads are to be blocked waiting.

Each thread is assigned one of the numbers 0, 1 and 2. Each number is divided by 3 (number of threads), remainder will be any one of these numbers 0, 1 or 2. That is what is checked; **if (remainder = number assigned to thread)** only then thread can work otherwise it goes into waiting state.

```
class SharedPrinter{
  int number = 1;
  int numOfThreads;
  int numInSequence;
  SharedPrinter(int numInSequence, int numOfThreads){
    this.numInSequence = numInSequence;
    this.numOfThreads = numOfThreads;
  }
  public void printNum(int result){
    synchronized(this) {
      while (number < numInSequence - 1) {
        while(number % numOfThreads != result){
          try {
            this.wait();
          } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
          }
        }
        System.out.println(Thread.currentThread().getName() + " - " + number++);
        this.notifyAll();
      }
    }
  }
}
class SeqRunnable implements Runnable{
  SharedPrinter sp;
  int result;
  static Object sharedObj = new Object();
  SeqRunnable(SharedPrinter sp, int result){
    this.sp = sp;
    this.result = result;
  }
  @Override
  public void run() {
    sp.printNum(result);
  }
}
public class SeqNumber {
  final static int NUMBERS_IN_SEQUENCE = 10;
  final static int NUMBER_OF_THREADS = 3;
  public static void main(String[] args) {
    // Shared object
    SharedPrinter sp = new SharedPrinter(NUMBERS_IN_SEQUENCE, NUMBER_OF_THREADS);
    // Creating 3 threads
    Thread t1 = new Thread(new SeqRunnable(sp, 1), "Thread1");
    Thread t2 = new Thread(new SeqRunnable(sp, 2), "Thread2");
    Thread t3 = new Thread(new SeqRunnable(sp, 0), "Thread3");

    t1.start();
    t2.start();
    t3.start();
  }
}
```

**Output**

```
Thread1 - 1
Thread2 - 2
Thread3 - 3
Thread1 - 4
Thread2 - 5
Thread3 - 6
Thread1 - 7
Thread2 - 8
Thread3 - 9
Thread1 - 10
```

That's all for this topic **Printing Numbers in Sequence Using Threads Java Program**. If you have any doubt or any suggestions to make please drop a comment. Thanks!