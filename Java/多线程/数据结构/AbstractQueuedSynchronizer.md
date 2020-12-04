- 队列同步器AbstractQueuedSynchronizer（以下简称同步器或AQS），是用来构建锁或者其他同步组件的基础框架，它使用了一个内置的int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。
- AQS内部维护一个state状态位，尝试加锁的时候通过CAS(CompareAndSwap)修改值，如果成功设置为1，并且把当前线程ID赋值，则代表加锁成功，一旦获取到锁，其他的线程将会被阻塞进入阻塞队列自旋，获得锁的线程释放锁的时候将会唤醒阻塞队列中的线程，释放锁的时候则会把state重新置为0，同时当前线程ID置为空。

![image-20201130220321615](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/30/22:03:22-image-20201130220321615.png)

