## 引导
虽然`ReentrantLock`已经实现了重入锁的全部功能，但有没有性能更好的锁呢？
数据处理既然分为读和写操作，那我们能不能把锁拆分为两种呢，因为有时候我们并不需要写和读同时共享同一把锁，获取更高的吞吐量。
`ReenTrantReadWriteLock`就是这么一把这样的锁

## 如何使用？

## 定义
`ReentrantReadWriteLock`从字面自已了解到，这是一把可重入的读写锁，同时拥有`ReentrantReadWriteLock.ReadLock`和`ReentrantReadWriteLock.WriteLock`两把锁。
主要具有以下特点：
1. 读写锁都可以重入
2. 读写锁互斥
3. 同时获取读写锁时，必须先获取`writeLock`再获取`readLock`，否则直接导致死锁
4. 获取锁支持中断
5. 支持公平锁方式

## 如何实现？
```java
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
