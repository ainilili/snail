## 定义
工厂模式屏蔽了对象的实例化过程，提供单独的入口去获取同一抽象接口下的多个实现。
- **优点**：扩展性高，屏蔽了实例化过程。
- **缺点**：增加了系统复杂度。

## 实现
**场景：** 通过武器工厂去获取不同的武器！

抽象武器类：
```java
public abstract class Armament {
	public abstract String name();
}
```
坦克：
```java
public class Tank extends Armament{

	@Override
	public String name() {
		return "tank";
	}

}
```
炸弹：
```java
public class Bomb extends Armament{

	@Override
	public String name() {
		return "bomb";
	}

}
```
飞机：
```java
public class Airplane extends Armament{

	@Override
	public String name() {
		return "air plane";
	}

}
```
最后的是武器工厂：
```java
public class ArmamentFactory {

	public static Armament getArmament(String name){
		if(null == name || name.equals("")){
			return null;
		}
		if(name.equals("airplane")){
			return new Airplane();
		}else if(name.equals("tank")){
			return new Tank();
		}else if(name.equals("bomb")){
			return new Bomb();
		}else{
			return null;
		}
	}

}
```
测试：
```java
public class Test {

	public static void main(String[] args) {

		Armament armament = ArmamentFactory.getArmament("airplane");
		System.out.println(armament.name());

		armament = ArmamentFactory.getArmament("tank");
		System.out.println(armament.name());

		armament = ArmamentFactory.getArmament("bomb");
		System.out.println(armament.name());
	}
}
```
输出结果：
```powershell
air plane
tank
bomb
```
