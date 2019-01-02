**目录**
- [1 AQS 简介](#1-AQS-简介)
- [2 原理](#2-原理)
    - [2.1 核心状态方法](#2.1-核心状态方法)
    - [2.2 资源共享方式](#2.2-资源共享方式)
- [3 AQS组件](#3-AQS组件)
    - [3.1 CountDownLatch 计时器](#3.1-CountDownLatch-计时器)
    - [3.2 Semaphore 信号量](#-Semaphore-信号量)

### 1 AQS 简介
AQS全称`AbstractQueuedSynchronizer`，即**队列同步器**，位于`java.util.concurrent.locks`包下，由著名并发编程大师`Doug Lea`于JDK1.5实现**J.U.C包中的核心基础组件**
>AQS与此同时也是面试中并发编程的重要知识点

![AQS实现的方法](https://user-gold-cdn.xitu.io/2018/12/27/167eb5eb024d9bcd?w=655&h=371&f=png&s=90429)

### 2 原理
**核心思想：通过STATE同步状态位、FIFO的CLH队列锁，只能在一个时刻发生阻塞，降低上下文切换开销，从而提高吞吐量。**

AQS对象内部通过一个**volatile**修饰的int类型的核心变量**state**表示加锁状态。初始状态下，这个state的值为0。

另外AQS内部还有一个**关键变量**，来记录**当前加锁的线程**，初始状态下，这个变量为null

```java
volatile int state;                 // 共享状态变量
volatile Thread thread;     // 当前加锁线程
volatile Node prev;         //前驱节点
volatile Node next;         //后继节点

```

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的FIFO双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配，同时会挂起当前线程吗，当同步状态释放时，会把首节点唤醒（公平锁），使其再次尝试获取同步状态。
> 在CLH同步队列中，一个节点表示一个线程，保存 着线程的引用（thread）、状态、（waitStatus）、前驱节点（prev）、后继节点（next）

通过内置的FIFO同步队列完成资源获取线程的排队工作，如果获取锁（同步状态）失败，则会把当前线程和状态等信息构造成一个结点（Node）并加入同步等待队列，当同步状态释放时，则会把结点中线程唤醒以再次尝试获取锁

TODO Node节点长截图

#### 2.1 核心状态方法

**由于良好的设计，AQS具有充分的可伸缩性，并隐藏了大量实现细节，从而可以极大的方便扩展**

![AQS](https://user-gold-cdn.xitu.io/2018/12/29/167f7e584754b919?w=875&h=894&f=png&s=90391)

状态信息更新方法
```java
protected final int getState() {  return state; } //获取同步状态值
protected final void setState(int newState) { state = newState; }//设置同步状态
//通过unsafe的CAS方法原子更改状态，expect期望值，update新值
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update); }
```

#### 2.2 资源共享方式
>同步状态，以下简称锁
- **Exclusive**（独占）：同一时刻只有一个线程能够获取锁执行，如`ReentrantLock`。又可以分为公平锁和非公平锁。
    - **FairSync**（公平锁）：严格按照线程在队列中的FIFO顺序，即先到先得（锁）
    - **NonfairSync**（非公平锁）：新加入的线程会尝试通过CAS抢占锁，无视队列顺序，失败则加入同步等待队列。此锁大多数情况下会带来更好的IO性能，但有可能会造成线程饿死（线程一直处于等待状态无法获取锁）
- **Share**（共享）：多个线程可以同时执行，如Semaphore/CountDownLatch、ReadWriteLock。

**不同的同步器争用共享资源的方式也不同，自定义同步器在实现时只需要实现共享资源state的获取与释放即可，对于线程等待队列的维护已经由AQS在上层实现。**

>默认情况下，每个方法都会抛出`UnsupportedOperationException`异常，需要继承AQS抽象类并实现相应的方法。

```java
isHeldExclusively(); //该线程正在是否独占资源
```

```java
//独占锁实现
tryAcquire(int); //独占式获取锁，其他线程必须等待该线程释放锁才能获取锁
tryRelease(int); //独占式释放锁
```

```java
//共享锁实现
tryAcquireShared(int); //共享式可重入锁，返回值大于等于0表示获取成功，否则失败
tryReleaseShared(int); //共享式释放锁
```
一般来说，自定义同步器要么是**独占方法**，要么是**共享方法**，只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一组即可。但AQS也可以实现**独占和共享**两种方式，如`
ReentrantReadWriteLock`

>AQS其他方法均为`final`，所以无法被其他自定义同步器重写使用

### 3 AQS组件

#### 3.1 CountDownLatch 计时器

##### 能做什么？

>能够允许一个或多个线程阻塞等待，直到所有的线程执行完毕后再继续执行剩余操作。

##### 如何使用？

```java
        int count = 10;
        CountDownLatch cdl = new CountDownLatch(count); //实例化对象，并初始化STATE为count
        for (int i = 0; i < count; i++) {
            new Thread(new Runable() {
                public void run() {
                    try {
                        // do something
                    } finally {
                        cdl.countDown(); // STATE - 1
                    }
                }
            }).start();
        }
        cdl.await(); //挂起线程，等待STATE = 0，继续剩余操作
        // do something
```

##### 如何实现？

![CountDownLatch继承AQS](https://user-gold-cdn.xitu.io/2018/12/27/167ef96070c28d7b)

>CountDownLatch只需要实现少量代码即可实现相应的功能。

![CountDownLatch构造函数](https://user-gold-cdn.xitu.io/2018/12/27/167efb28587a671c?w=605&h=79&f=png&s=9275)

实例化对象同时将继承了AQS的内部类Sync初始化state为count。Sync由于可共享，只需要重写`tryAcquireShared(int)`与`tryReleaseShared(int)`方法。

`await()`调用`sync.acquireSharedInterruptibly`中的`tryAcquireShared(arg)`判断当前是否还有正在执行的线程，如果还有线程已经完成，直接返回，否则通过`doAcquireSharedInterruptibly(arg)`通过不断自旋的方式不断获取同步状态，但在自旋中需要调用`shouldParkAfterFailedAcquire()`判断当前线程是否需要挂起，通过方法`
acquireQueued()`，并向队尾新增一个节点并将前节点设为`SIGNAL`等待标志位，通过`LockSupport.park()`方法调用`park()`进入挂起线程，停止消耗资源。

```java
   private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        //前驱节点
        int ws = pred.waitStatus;
        //状态为signal，表示当前线程处于等待状态，直接放回true
        if (ws == Node.SIGNAL)
            return true;
        //前驱节点状态 > 0 ，则为Cancelled,表明该节点已经超时或者被中断了，需要从同步队列中取消
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        }
        //前驱节点状态为Condition、propagate
        else {
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```

```java
//挂起当前线程并检查是否中断
private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```

`countDown()`通过`tryReleaseShared(int)`操作state使用CAS原子减1。等待所有子线程执行完成(state=0)，通过`unpark()`唤醒挂起在`park()`中的线程状态，继续剩余操作。
```java
    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
```
```java
    protected boolean tryReleaseShared(int releases) {
        for (;;) {
            // 获取当前state值
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            // CAS原子减1
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
```
```java
    private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```
```java
    private void unparkSuccessor(Node node) {
        // 当前节点状态
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        Node s = node.next;
        // 没有后继节点或者超时、被中断
        if (s == null || s.waitStatus > 0) {
            s = null;
            // 从tail尾节点开始查找可用节点
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        // 唤醒后继节点
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```
>节点可能存在没有后继节点或者取消（超时、中断）的情况，则需要跳过此节点。
>继续寻找node.next可用节点时，仍然可能存在null或者取消的情况，所以采用tail回溯方法寻找第一个可用线程，并唤醒该线程

#### 3.2 Semaphore 信号量

#### 3.3 CyclicBarrier 同步屏障

#### 3.4 ReentrantLock

#### 3.5 ReentrantReadWriteLock