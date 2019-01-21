## Semaphore 信号量
### 能做什么呢？
>Semaphore本质上是个计数信号量，实现了对线程的管理。
>该信号量维护了一个**许可集**，线程在运行时首先从许可集中获取许可，线程运行完毕归还许可，来达到**限制并发执行的数量**的目的。

```java
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```
构造方法中需要指定许可集的数量，默认使用的是非公平锁模式，也可以指定为公平锁模式。
>**permits**变量将赋值给AQS中的**state**变量

### 如何使用？

假如有一个停车场，但是只有一定数量的停车位，当停车位满了之后，就不允许其他车辆停车了。
当停车场的车还未腾出空位时，可能还有其他车辆进入需要停车，此时需要告诉他们此时车位已满需要等候（排队或抢占或自行到其他地方寻找车位），只有当停车场的车腾出位置时，才允许其他车辆停车。

```java
ExecutorService commonPool = Executors.newFixedThreadPool(10);
int semaphoreCount = 3; // 允许三个线程同时执行
Semaphore semaphore = new Semaphore(semaphoreCount); 
int threadCount = 5;
while (threadCount-- > 0) {
    commonPool.execute(()->{
        try {
            semaphore.acquire();
            // do something
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            semaphore.release(); // 返回许可 
        }
    });
}
// do other something
```
通过的控制线程执行的数量，保证合理的使用公共资源，以保障系统的快速、稳定的运行。

### 如何实现？

Semaphore提供了两种资源获取方式：**响应中断**&**不响应中断**

响应中断模式
```java
// 阻塞获取许可且响应中断模式
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
// 尝试在指定时间内得到一个许可，否则中断
public boolean tryAcquire(long timeout, TimeUnit unit)
    throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
// 尝试在指定时间内得到permites个许可，否则中断
public boolean tryAcquire(int permits, long timeout, TimeUnit unit)
    throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    return sync.tryAcquireSharedNanos(permits, unit.toNanos(timeout));
    }
```

不响应中断模式
```java
// 阻塞获取许可不相应中断模式
public void acquireUninterruptibly() {
    sync.acquireShared(1);
}
// 立即返回是否有可用许可。如果有，将许可数减少一个
public boolean tryAcquire() {
    return sync.nonfairTryAcquireShared(1) >= 0;
}
// 立即返回是否有可用许可。如果有，将许可数减少permits个
public boolean tryAcquire(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    return sync.nonfairTryAcquireShared(permits) >= 0;
}
```
尝试获取信号量，不会阻塞当前线程，获取成功返回true，减少许可数，否则马上返回false

```java
// 释放信号量，返还许可
public void release() {
    sync.releaseShared(1);
}

public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        int next = current + releases;
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
        if (compareAndSetState(current, next))
            return true;
    }
}
```
>释放许可实际上是将AQS中state值加1，并且通过**doReleaseShared**唤醒等待队列中的第一个节点。
