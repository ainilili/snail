## CountDownLatch是什么
CountDownLatch简称CDL，它的作用挂起当前线程，在多个线程状态达到一致的时候再继续运行当前线程，在实际应用中相当广泛，一个简单的流程图如下：

![](https://github.com/ainilili/snail/blob/master/images/count-down-latch-1-1.jpg?raw=true)

对于CountDownLatch的构造需要传入一个int数值作为计数器的初始值，并拥有两个核心方法：
 - **countDown**：减少计数，当计数大于0时，await将会一直阻塞。
 - **await**：对当前线程阻塞，直到计数器减为0，线程将会重新运行。

CountDownLatch的实现依赖于AQS，核心是通过``Unsafe``的``park``和``unpark``实现线程的挂起和启动当前线程来实现线程的阻塞和执行。

这里简单介绍一下AQS，它是``Doug Lea``大佬写的一款``FIFO``等待队列同步框架，内部使用``CLH``自旋锁实现。并且，AQS有两种锁模式：``独占``和``共享``，显然，CountDownLatch是后者。

知道了CountDownLatch原理，那么实现一个CountDownLatch的思路将会有很多，使用AQS我们可以轻松的实现一个CountDowanLatch，当然，我们也可以换一个思路！
## 另一种方式实现CountDownLatch
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
