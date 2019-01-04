## 定义
建造者模式的设计思想是将复杂的结构组件化，以后一步步的拼装组件去构建新的对象。

**最典型的例子就是点餐场景**：不同价格的汉堡和可乐需要不同的包装盒，最终的点单情况也要呈现在一个清单之上。

我们来使用建造者模式去构建以上场景，则首先要分析的是如何将上述场景进行结构拆分。很明显，清单由多个汉堡或可乐组成，再深入一层，汉堡和可乐又必须有一个合适的包装盒，那么关键的结构为三种：**清单**、**食物**、**包装盒**。
## 实现
上述场景过于传统，我们这次房间家具摆设为场景，使用建造者模式来编写一个简单的Demo，不过在此之前，我们首先要进行场景分析：
 - 一个房间由多种家具组成，如：沙发、床、桌子、椅子
 - 一个家具由多种属性组成，如：颜色、材质、长宽高
 - 材质分好多种：柚木、板式、实木、红木

材质抽象类：
```java
public abstract class Texture {
	public abstract String name();
}
```
红木材质：
```java
public class Rosewood extends Texture{

	@Override
	public String name() {
		return "红木";
	}

}
```
实木材质：
```java
public class Solidwood extends Texture{

	@Override
	public String name() {
		return "实木";
	}

}
```
家具抽象类：
```java
public abstract class Furniture {

	public abstract String name();

	public abstract Texture texture();

	public abstract  String color();

	public abstract  float width();

	public abstract  float height();

	public abstract  float length();

}
```
椅子：
```java
public class Chair extends Furniture{

	@Override
	public String name() {
		return "椅子";
	}

	@Override
	public Texture texture() {
		return new Rosewood();
	}

	@Override
	public String color() {
		return "红黑色";
	}

	@Override
	public float width() {
		return 40;
	}

	@Override
	public float height() {
		return 60;
	}

	@Override
	public float length() {
		return 40;
	}

}
```
桌子：
```java
public class Table extends Furniture{

	@Override
	public String name() {
		return "桌子";
	}

	@Override
	public Texture texture() {
		return new Solidwood();
	}

	@Override
	public String color() {
		return "木黄色";
	}

	@Override
	public float width() {
		return 200;
	}

	@Override
	public float height() {
		return 120;
	}

	@Override
	public float length() {
		return 200;
	}

}
```
房间：
```java
public class Room {

	private List<Furniture> furnitures;

	public Room(){
		this.furnitures = new ArrayList<>();
	}

	public final void append(Furniture f){
		this.furnitures.add(f);
	}

	public final String info(){
		final StringBuilder info = new StringBuilder();
		furnitures.forEach(f -> {
			info.append("##################" + System.lineSeparator());
			info.append("名称：" + f.name() + System.lineSeparator());
			info.append("颜色：" + f.color() + System.lineSeparator());
			info.append("材质：" + f.texture().name() + System.lineSeparator());
			info.append("长：" + f.length() + System.lineSeparator());
			info.append("宽：" + f.width() + System.lineSeparator());
			info.append("高：" + f.height() + System.lineSeparator());
		});
		return info.toString();
	}
}
```
测试：
```java
public class Test {

	public static void main(String[] args) {
		Room room = new Room();
		room.append(new Chair());
		room.append(new Table());

		System.out.println(room.info());

	}
}
```
输出结果：
```powershell
##################
名称：椅子
颜色：红黑色
材质：红木
长：40.0
宽：40.0
高：60.0
##################
名称：桌子
颜色：木黄色
材质：实木
长：200.0
宽：200.0
高：120.0
```
## 总结
建造者模式细化了复杂对象初始化的过程，将结构分离开来，一步一步的去构造一个完整的对象，这就促使我们可以在这个过程中加入一些业务逻辑，例如对于拼装子结构时我们可以做一些校验之类的工作，不过建造者模式并不适用于所有场景，并且会产生很多建造类，请务必在合适的场景使用合适的设计模式！
