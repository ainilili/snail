## CountDownLatch 计数器

### 能做什么？

>能够允许一个或多个线程阻塞等待，直到所有的线程执行完毕后再继续执行剩余操作。

### 如何使用？

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

### 如何实现？

```java
public class CountDownLatch {
    private static final class Sync extends AbstractQueuedSynchronizer {

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }

        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int releases) {
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }
    private final Sync sync;
}
```

>CountDownLatch只需要实现少量代码即可实现相应的功能。

```java
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }
```

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
