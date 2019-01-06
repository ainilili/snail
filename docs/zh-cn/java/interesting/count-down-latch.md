## 原版对比
CountDownLatch的作用是作为一个同步锁，挂起当前线程，在多个线程状态达到一致的时候再继续执行当前线程，在实际应用中相当广泛，利用这个特性我们可以对一些场景业务做并发处理，如：
 - Zookeeper多结点连接状态同步
 - 大量数据分批处理后合并

CountDownLatch是通过AQS这个并发框架实现的，核心是通过``Unsafe``的``park``和``unpark``实现线程的挂起和启动当前线程来实现线程的阻塞和执行。

这里简单介绍一下AQS，它是``Doug Lea``大佬写的一款``FIFO``等待队列同步器框架，内部使用``CLH``自旋锁实现。AQS作为一个并发框架，我们并不能直接使用它，通常需要继承它并重写以下几个方法：
 - **isHeldExclusively()**：该线程是否正在独占资源。只有用到condition才需要去实现它。
 - **tryAcquire(int arg)**：独占方式。尝试获取资源，成功则返回true，失败则返回false。
 - **tryRelease(int arg)**：独占方式。尝试释放资源，成功则返回true，失败则返回false。
 - **tryAcquireShared(int arg)**：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
 - **tryReleaseShared(int arg)**：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false

可见，AQS有两种锁模式：``独占``和``共享``，其中CountDownLatch就是后者。这篇文章中，我将会讲述如何通过其他方式去实现一个简单的CountDownLatch！
## 新的实现
对于CountDownLatch的构造需要传入一个int数值作为计数器的初始值，并拥有两个核心方法：
 - **countDown**：减少计数。
 - **await**：对当前线程阻塞，知道计数器减为0。

我们知道，Object自带``wait()``、``notify``和``notifyAll``这三个方法，其中wait方法可以导致线程进入等待状态，直到它被其他线程通过notify()或者notifyAll唤醒，因此我们可以使用这个特性去代替``Unsafe``对线程进行挂起和运行！

对于计数操作，我们使用AtomicInteger来作为计数载体，它内部使用CAS自选的方式保证数值操作的原子性，接下来的代码就很简单了！
```java
import java.util.concurrent.atomic.AtomicInteger;

/**
 *
 * @author nico
 * @email ainililia@163.com
 */
public class AtomicDownLatch {

	private volatile int state;

	private AtomicInteger count;

	private Object lock = new Object();

	public AtomicDownLatch(int count){
		this.count = new AtomicInteger(count);
	}

	public void countDown(){
		count.decrementAndGet();
		release();
	}

	public boolean release(){
		int s = count.get();
		if(s == 0){
			synchronized (lock) {
				lock.notifyAll();
			}
			return true;
		}else{
			return false;
		}
	}

	public void await() throws InterruptedException{
		synchronized (lock) {
			if(state == 0){
				state = 1;
				if(! release()){
					lock.wait();
				}
			}
		}
	}
}
```
简单测试：
```java
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 *
 * @author nico
 * @email ainililia@163.com
 */

public class Test3 {

	public static void main(String[] args) throws InterruptedException {
		int sum = 0;
		int count = 1000;
		int loop = 1000;
		ThreadPoolExecutor tpe = new ThreadPoolExecutor(100, 100, 0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());

		while(count -- > 0){
			AtomicDownLatch as = new AtomicDownLatch(loop);
			for(int i = 0; i < loop; i ++){
				tpe.execute(as::countDown);
			}
			as.await();
			sum ++;
		}

		System.out.println(sum);
		tpe.shutdown();
	}
}
```
测试结果：
```powershell
1000
```
