## 定义
适配器模式（Adapter Pattern）是作为两个不兼容的接口之间的桥梁。这种类型的设计模式属于结构型模式，它结合了两个独立接口的功能。

这种模式涉及到一个单一的类，该类负责加入独立的或不兼容的接口功能。举个真实的例子，读卡器是作为内存卡和笔记本之间的适配器。您将内存卡插入读卡器，再将读卡器插入笔记本，这样就可以通过笔记本来读取内存卡。

适配器的主要意图是将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。
## 实现
仅仅通过概念是不太容易理解的，来举个**栗子**好了！

**场景**：某科技公司在2018年负责开发智能AI机器人，经过工作人员两个月的努力，一共开发出了``Alpha``和``Beta``两个版本的机器人，它们的功能各有优劣，所以Leader希望让两个版本兼容在一起工作，这样显得更加智能，于是落实工作落在了Nico身上。

**实现**：Nico接到任务之后，仔细思考了一番，心里大喜，所幸自己学过设计模式，其中``适配器模式``就能解决老大安排下的问题，于是三下五除二就撸出了实现代码！

Robot-Alpha型号接口：
```java
public interface RobotAlpha {
	public String dialogue(String input);
}
```
Robot-Beta型号接口：
```java
public interface RobotBeta {
	public String dialogue(String input);
}
```
Robot-Beta型号成型产品：
```java
public class RobotBetaProduct implements RobotBeta{

	@Override
	public String dialogue(String input) {
		return input.replace("吗", "").replaceAll("[?|？]", "!");
	}

}
```
Robot-Alpha型号成型产品的适配器型号：
```java
public class RobotAlphaAdapter implements RobotAlpha{

	private RobotBeta robotBeta;

	public RobotAlphaAdapter(RobotBeta robotBeta) {
		this.robotBeta = robotBeta;
	}

	@Override
	public String dialogue(String input) {
		if(input.endsWith("吗") || input.endsWith("?") || input.endsWith("？")) {
			return robotBeta.dialogue(input);
		}else {
			return "嗯嗯！";
		}
	}

}
```
测试：
```java
public class Test {

	public static void main(String[] args) {

		RobotAlpha alpha = new RobotAlphaAdapter(new RobotBetaProduct());
		Scanner sc = new Scanner(System.in);
		String input;

		while(sc.hasNextLine()) {
			input = sc.nextLine();
			String answer = alpha.dialogue(input);
			System.out.println(answer);
		}

		sc.close();
	}
}
```
运行结果：
```powershell
input: 你好
robot: 嗯嗯！
input: 能听懂话吗？
robot: 能听懂话!
input: 真的？
robot: 真的!
```
## 总结
很适合做多版本的兼容。
