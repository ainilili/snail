**目录**
- [1 AQS 简介](#1-AQS-简介)
- [2 原理](#2-原理)
    - [2.1 核心状态方法](#2.1-核心状态方法)
    - [2.2 资源共享方式](#2.2-资源共享方式)

## 1 AQS 简介
AQS全称`AbstractQueuedSynchronizer`，即**队列同步器**，位于`java.util.concurrent.locks`包下，由著名并发编程大师`Doug Lea`于JDK1.5实现**J.U.C包中的核心基础组件**
>AQS与此同时也是面试中并发编程的重要知识点

![AQS实现的方法](https://user-gold-cdn.xitu.io/2018/12/27/167eb5eb024d9bcd?w=655&h=371&f=png&s=90429)

## 公平锁 / 非公平锁
**公平锁** 多个线程完全按照申请锁的顺序来获取锁
**非公平锁** 多个线程获取锁的线程并不一定按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。但可能会造成优先级反转或者线程饿死现象。

> 对于**ReentrantLock**而言，通过构造函数指定该锁是否为公平锁，默认非公平锁，非公平锁的有点在于吞吐量比公平锁要大。
> 对于**synchronized**而言，也是一种非公平锁。由于其不像**ReentrantLock**是通过**AQS**的来实现线程调度，所以并没有任何办法使其变成公平锁。

## 可重入锁 / 不可重入锁
Wiki：可重入互斥锁（英语：reentrant mutex）是互斥锁的一种，同一线程对其多次加锁不会产生死锁。可重入互斥锁也称递归互斥锁（英语：recursive mutex）或递归锁（英语：recursive lock）。
如果对已经上锁的普通互斥锁进行“加锁”操作，其结果要么失败，要么会阻塞至解锁。而如果换作可重入互斥锁，当且仅当尝试加锁的线程就是持有该锁的线程时，类似的加锁操作就会成功。可重入互斥锁一般都会记录被加锁的次数，只有执行相同次数的解锁操作才会真正解锁。递归互斥锁解决了普通互斥锁不可重入的问题：如果函数先持有锁，然后执行回调，但回调的内容是调用它自己，就会产生死锁。
### 可重入锁
广义上的可重入锁指的是**可重复可递归调用**的锁，在外层使用锁之后，在内层仍然可以使用，并且会不发生死锁（前提是同一个对象或者class），这样的锁叫做可重入锁。
**ReentrantLock**和**synchronized**都是可重入锁
```java
public synchronized void setA() throws InterruptedException {
    TimeUnit.SECONDS.sleep(1);
    setB();
}
public synchronized void setB() throws InterruptedException {
    TimeUnit.SECONDS.sleep(1);
}
```
如上的代码就是一个可重入锁的一个特点，如果不是可重入锁的话，setB可能不会被执行，等待setA锁的释放，造成死锁。
### 不可重入锁
不可重入锁与可重入锁相反，不可递归调用，递归调用就会发生死锁。
自旋锁就是一个很精单的不可重入锁。
```java
import java.util.concurrent.atomic.AtomicReference;

public class UnreentrantLock {
    private AtomicReference<Thread> owner = new AtomicReference<Thread>();

    public void lock() {
        Thread current = Thread.currentThread();
        for (;;) {
            if (!owner.compareAndSet(null, current)) {
                return;
            }
        }
    }

    public void unlock() {
        Thread current = Thread.currentThread();
        owner.compareAndSet(current, null);
    }
}
```
使用原子引用存放线程，同一线程两次调用 lock() 方法，不调用 unlock() 释放锁的话，第二次调用自旋锁的时候就会产生死锁，这个锁就是不可重入，而实际上同一个线程不必每次都去释放锁再来获取锁，避免频繁调度切换耗费资源。

## 互斥锁 / 读写锁
### 互斥锁
在访问共享资源时进行加锁操作，访问结束之后进行解锁操作。加锁后其他任何试图加锁的线程都会被阻塞，直到持有锁的线程解锁。

### 读写锁
读写锁即是互斥锁，又是共享锁，read模式是共享，write是互斥的。

## 2 原理
**核心思想：通过state同步状态位、FIFO的CLH队列锁，只能在一个时刻发生阻塞，降低上下文切换开销，从而提高吞吐量。**

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

```java
static final class Node {
    static final Node SHARED = new Node();
    static final Node EXCLUSIVE = null;
    //表示当前的线程被取消；
    static final int CANCELLED =  1;
    //表示当前节点的后继节点包含的线程需要运行，也就是unpark；
    static final int SIGNAL    = -1;
    //表示当前节点在等待condition，也就是在condition队列中；
    static final int CONDITION = -2;
    //表示当前场景下后续的acquireShared能够得以执行；
    static final int PROPAGATE = -3;
    //表示节点的状态。默认为0，表示当前节点在sync队列中，等待着获取锁。
    //其它几个状态为：CANCELLED、SIGNAL、CONDITION、PROPAGATE
    volatile int waitStatus;
    //前驱节点
    volatile Node prev;
    //后继节点
    volatile Node next;
    //获取锁的线程
    volatile Thread thread;
    //存储condition队列中的后继节点。
    Node nextWaiter;
}
```

## 2.1 核心状态方法

**由于良好的设计，AQS具有充分的可伸缩性，实现了大量实现通用细节，从而可以极大的方便扩展**

![AQS](https://user-gold-cdn.xitu.io/2018/12/29/167f7e584754b919?w=875&h=894&f=png&s=90391)

状态信息更新方法
```java
protected final int getState() {  return state; } //获取同步状态值
protected final void setState(int newState) { state = newState; }//设置同步状态
//通过unsafe的CAS方法原子更改状态，expect期望值，update新值
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update); }
```

## 2.2 资源共享方式
同步状态，以下简称锁
- **Exclusive**（独占）：同一时刻只有一个线程持有，如`ReentrantLock`，`synchronized`。又可以分为公平锁和非公平锁。
    - **FairSync**（公平锁）：严格按照线程在队列中的FIFO顺序，即先到先得（锁）
    - **NonfairSync**（非公平锁）：新加入的线程会尝试通过CAS抢占锁，无视队列顺序，失败则加入同步等待队列。此锁大多数情况下会带来更好的IO性能，但有可能会造成线程饿死（线程一直处于等待状态无法获取锁）
- **Share**（共享）：多个线程可以同时执行，如Semaphore/CountDownLatch、ReentrantReadWriteLock里的读锁是共享的，但是写锁每次只能被独占。
读锁的共享可以保证并发读非常高效，但是读写、写写，写读都是互斥

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
