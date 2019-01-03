## 定义
抽象与实现解耦，例如一个生产线的机器，将参数设定与具体生产的过程实现分离开，使整个生产线更加灵活，用一套参数也可以使用不同的工作模式。

其中参数设定与生产过程属于两种不同的维度，桥接模式所做的事情就是将不同的维度联结在一起！
## 实现
**场景**：不同速率的生产线使用同一套生产模式。

生产线抽象类：
```java
public abstract class Pipeline {

	protected int speed;

	protected Work work;

	public Pipeline(int speed, Work work) {
		super();
		this.speed = speed;
		this.work = work;
	}

	/**
	 * 生产线动作抽象方法
	 */
	public abstract void start();

}
```
工作模式接口：
```java
public interface Work {
	public void process(int speed);
}
```
A工作模式实现：
```java
public class WorkA implements Work{

	@Override
	public void process(int speed) {
		System.out.println("Work-A is producing by speed " + speed + "/s");		
	}

}
```
A流水线实现：
```java
public class PipelineA extends Pipeline{

	public PipelineA(int speed, Work work) {
		super(speed, work);
	}

	@Override
	public void start() {
		new Thread(){
			@Override
			public void run(){
				while(true){
					try {
						Thread.sleep(1000L/speed);
						work.process(speed);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
		}.start();

	}

}
```
测试：
```java
public class Test {

	public static void main(String[] args) {

		Pipeline pl2 = new PipelineA(2, new WorkA());
		Pipeline pl5 = new PipelineA(5, new WorkA());

		pl2.start();
		pl5.start();
	}
}
```
输出结果：
```powershell
Work-A is producing by speed 5/s
Work-A is producing by speed 5/s
Work-A is producing by speed 2/s
Work-A is producing by speed 5/s
Work-A is producing by speed 5/s
Work-A is producing by speed 5/s
Work-A is producing by speed 2/s
Work-A is producing by speed 5/s
Work-A is producing by speed 5/s
...
...
```
## 总结
桥接模式解耦能力强，使系统更利用扩展且整体结构划界清晰明了，对使用者的抽象设计能力有一定的要求。
