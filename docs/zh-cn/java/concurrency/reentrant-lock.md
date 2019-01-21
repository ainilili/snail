## ReentrantLock

ReentrantLock从字面理解叫可重入锁，它支持对**同一线程的重复加锁**，与JVM传统的关键字**synchronized**监视器锁提供相同的独占锁、可重入互斥和内存语义，同时又支持一些高级特性，比如**尝试加锁、定时等待、中断、公平锁等等**一些关键字没有的特性，使用可以更灵活。

### 为什么要用锁？

### 能做什么？
```java
final Lock lock = new ReentrantLock(); // 1
int thread = 5;
CountDownLatch cdl = new CountDownLatch(thread);
while (thread-- > 0) {
    new Thread(() -> {
        lock.lock(); // 2
        try {
            TimeUnit.SECONDS.sleep(1);
            System.out.println("sleep release at:" + new SimpleDateFormat("ss").format(new Date()));
        }  finally {
            cdl.countDown(); 
            lock.unlock(); // 3
        }
    }).start();
}
cdl.await(); // 4
System.out.println("complete");
```

1. 实例化 ReentrantLock 对象
2. 获取 lock
3. 释放 lock
4. 等待全部线程执行完毕

> 执行结果
```java
sleep release at:45
sleep release at:46
sleep release at:47
sleep release at:48
sleep release at:49
complete
```

从代码输出可以看出，开启的多个线程并没有同时输出。Lock 使多线程队列运行，未获取 Lock 的线程必须等待获得 Lock 的线程的 unlock 操作（解锁）才能获取锁继续运行，合理的使用来保证程序的正确性。
同时需要注意与 synchronized 关键字不同的是 **synchronized 抛出异常会自动释放监视器锁，而ReentrantLock 必须手动显示释放 lock** ，推荐使用try ... finally ... 将 unlock 包裹在 finally 中保证锁可以正确的释放，以免造成死锁。

```java
public ReentrantLock() {    sync = new NonfairSync();}

public ReentrantLock(boolean fair) {    sync = fair ? new FairSync() : new NonfairSync();}
```
>ReentrantLock 拥有两种运行模式：**NonfairSync** 和 **FairSync**， 默认使用 NonfairSync 非公平锁模式提高吞吐量，但也有可能会出现线程饿死。

ReentrantLock 的 lock 的获取释放都是通过继承 AQS 内部类 Sync 的子类 NonfairSync、FairSync 来实现。

```java
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires; // 增加重入标志位
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc); // 由于独占锁特性，此操作线程安全
                return true;
            }
            return false;
        }

static final class NonfairSync extends Sync {

    final void lock() {
        if (compareAndSetState(0, 1)) //尝试直接获取锁
            setExclusiveOwnerThread(Thread.currentThread()); //设置独占锁线程标志
        else
            acquire(1); //尝试获取重入锁,，是否加入AQS Queue
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}

static final class FairSync extends Sync {

    final void lock() {
        acquire(1);  //获取锁，判断AQS Queue是否有等待获取的线程
    }

    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            //判断AQS Queue中是否有线程等待获取锁，没有则直接获取锁
            if (!hasQueuedPredecessors() && 
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current); //设置独占锁线程标志
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) { //已有线程获取锁，尝试获取重入锁
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc); 
            return true;
        }
        return false;
    }
}
```
由此，公平锁和非公平锁的主要区别：
1.  非公平锁直接尝试获取锁，否则加入到 AQS Queue 队列，`以获得最大吞吐量`。
2. 非公平锁会首先判断 AQS Queue 中是否有等待锁的线程，没有才尝试获取锁。

>下面以非公平锁作详细介绍

```java
    // tryAcquire 尝试获取锁与重入锁，成功则直接返回
    // addWaiter 封装当前线程成 Node，加入 AQS Queue ，等待唤醒
    // acquireQueued 自旋方式获取锁
    // selfInterrupt 中断线程
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

```java
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode); // 包装成 Node
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) { // 尝试 CAS 操作 Node 到队尾
                pred.next = node; // 链表连接队列的最后一个节点到 Node
                return node;
            }
        }
        enq(node); // 将 Node 入队
        return node;
    }
    
    // 初始化 Queue 和通过 CAS 的方式将 Node 添加到队尾
    private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) {
            if (compareAndSetHead(new Node())) // 初始化 Queue
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```
释放锁时也比较简单，通过减少 state 标志位，由于可重入，直到0时才会释放整个独占锁

```java
    public void unlock() {
        sync.release(1);
    }
    
    public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h); // 唤醒后继节点
        return true;
        }
        return false;
    }
    
    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null); // 占用锁线程置空，表示独占锁已释放
        }
        setState(c);
        return free;
    }
```
