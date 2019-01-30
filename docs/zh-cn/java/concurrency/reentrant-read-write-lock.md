## 引导
虽然`ReentrantLock`已经实现了重入锁的功能，但有没有性能更好的锁呢？
数据处理既然分为读和写操作，那我们能不能把锁拆分为两种呢，因为有时候我们并不需要写和读同时共享同一把锁，获取更高的并发和吞吐量。
读写锁`ReenTrantReadWriteLock`就是这么一把这样的锁。

## 定义
`ReentrantReadWriteLock`从字面自已了解到，这是一把可重入的读写锁，同时拥有`ReentrantReadWriteLock.ReadLock`和`ReentrantReadWriteLock.WriteLock`两把锁。
主要具有以下特点：
1. 读写锁都可以重入。最多可支持 65535 个递归写锁和 65535 个递归读锁
2. 读写锁互斥
3. 同时获取读写锁时，必须先获取`writeLock`再获取`readLock`，否则直接导致死锁
4. 获取锁支持中断
5. 支持公平和非公平锁方式
6. 锁降级。先获取写锁，再获取读锁，最后释放写锁，可以将写锁降级为读锁


## ReentrantReadWriteLock
```java
// ReentrantReadWriteLock
private final ReentrantReadWriteLock.ReadLock readerLock;
private final ReentrantReadWriteLock.WriteLock writerLock;
final Sync sync;

public ReentrantReadWriteLock() {
    this(false);
}

public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
```

```java
// ReadLock
public void lock() {
  sync.acquireShared(1);
}

public void unlock() {
  sync.releaseShared(1);
}

public boolean tryLock() {
    return sync.tryReadLock();
}

public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}

// WriteLock
public void lock() {
    sync.acquire(1);
}

public void unlock() {
    sync.release(1);
}

public boolean tryLock( ) {
    return sync.tryWriteLock();
}

public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```
> 虽然拥有两把锁，但是其锁的主体还是`Sync`来实现的。所以实际上*只有一把锁*，只是在获取锁和写入锁的方式上不一样。
`ReentrantReadWriteLock`依然使用 AQS 中 *int* 类型的 *state* 来表示同步状态，表示锁被重复获取的次数。
由于内部维护一对读写锁，要使用一个变量维护多种状态，其采用了`按位切割`的方式维护这个变量，高 16 位表示读，低 16 位表示写
```java
// Sync.java

static final int SHARED_SHIFT   = 16; // 移位
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1; // 65535，最大可重入次数
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```
* `#sharedCount(int c)` 获得写状态的锁的次数
* `#exclusiveCount(int c)` 获得持有读状态的锁的*线程数*。由于读锁可以同时被多个线程持有，并且每个线程持有的读锁都支持重入，所以需要对每个线程持有的读锁的数量单独计数，这里使用到了`HoldCounter`计数器

#### 【读锁】tryAcquireShared
尝试获取读同步状态，成功返回`>= 0`，否则返回`< 0`
```java
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    // 有写线程且非本线程直接返回-1
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    int r = sharedCount(c); // readLock 获取个数
    if (!readerShouldBlock() && // 读锁是否需要阻塞
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) { // 修改高16位状态
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current); // 重试
}
```

#### fullTryAcquireShared
```java
final int fullTryAcquireShared(Thread current) {
    HoldCounter rh = null;
    for (;;) {
        int c = getState();
        // 锁降级，有写线程且非本线程直接返回-1
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
        } else if (readerShouldBlock()) { // 需要阻塞读锁。判断当前线程是否获得读锁
            if (firstReader == current) {
            } else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    return -1;
            }
        }
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) { // 修改高16位读锁的状态
            // 第一次获取读锁
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```
> 本质上，`#fullTryAcquireShared(Thread current)`是`#tryAcquireShared(int unused)`方法的自旋锁逻辑
#### tryReadLock
尝试获取读锁，将立即返回结果
* 获取成功，返回 true
* 获取失败，返回 false，不等待排队
```java
final boolean tryReadLock() {
    Thread current = Thread.currentThread();
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0 &&
            getExclusiveOwnerThread() != current)
            return false;
        int r = sharedCount(c);
        if (r == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (r == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return true;
        }
    }
}
```
#### 读锁 unlock
```java
// ReentrantReadWriteLock.java
public void unlock() {
    sync.releaseShared(1);
}

// AbstractQueuedSynchronizer.java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

#### tryReleaseShared
```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) { // 释放锁的线程为第一个获取锁的线程
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    // 更新同步状态
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

#### 【写锁】tryRelease
```java
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        // 存在读写锁且有读锁或者持有写锁线程不是当前线程
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() || // 是否需要阻塞读锁，非公平锁时总为 false
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current); // 设置当前线程为持有读锁线程
    return true;
}
```
> 获取写锁时，只有在没有读锁或者重入写锁时才会成功。因为需要保证写锁的操作对读锁可见，否则获取读锁的线程就不能感知写线程的操作。
所以只有在读锁完全释放时，写锁才可以被当前线程获取。当写锁被获取，其他所有读写线程都会被阻塞。

#### tryWriteLock
尝试获取写锁，将立即返回结果
* 获取成功，返回 true
* 获取失败，返回 false，不等待排队
```java
final boolean tryWriteLock() {
    Thread current = Thread.currentThread();
    int c = getState();
    if (c != 0) {
        int w = exclusiveCount(c);
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
    }
    if (!compareAndSetState(c, c + 1))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

#### 写锁 unlock
```java
// ReentrantReadWriteLock.java
public void unlock() {
    sync.release(1);
}

// AbstractQueuedSynchronizer.java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

#### tryRelease
```java
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively()) // 判断当前是否为写锁持有者
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}
```
> 释放写锁时，每次释放均减少写状态，当状态为 0 时，释放写锁持有者线程，表示锁已经完全释放，可以由其他线程访问读写锁获取同步状态。此次写状态的修改对后续线程可见。

#### getThreadId
获取线程编号
```java
static final long getThreadId(Thread thread) {
    return UNSAFE.getLongVolatile(thread, TID_OFFSET);
}

private static final sun.misc.Unsafe UNSAFE;
private static final long TID_OFFSET;
static {
    try {
        UNSAFE = sun.misc.Unsafe.getUnsafe();
        Class<?> tk = Thread.class;
        TID_OFFSET = UNSAFE.objectFieldOffset
            (tk.getDeclaredField("tid"));
    } catch (Exception e) {
        throw new Error(e);
    }
}
```
理论上，可以直接使用`Thread#getId`方法获取线程编号。
```java
// Thread.java
private long tid;

public long getId() {
    return tid;
}
```
但实际上，`Thread`的这个方法是非`final`修饰的，实现`Thread`的子类，覆写该方法，可能导致无法获取到正确的`tid`属性。因此`ReentrantReadWriteLock`使用`Unsafe`直接获取`tid`属性。
