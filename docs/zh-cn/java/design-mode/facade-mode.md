## 定义
外观者模式将屏蔽了复杂的方法调用过程，将之包装为一个更容易使用的入口对外提供服务。

## 实现
**场景**：实现一个简单的计算器

计算接口：
```java
public interface Calculate {
    public int cal(int a, int b);
}
```

加法实现：
```java
public class AddCalculate implements Calculate{

    @Override
    public int cal(int a, int b) {
        return a + b;
    }

}
```

减法实现：
```java
public class SubCalculate implements Calculate{

    @Override
    public int cal(int a, int b) {
        return a - b;
    }

}
```

外观者：
```java
public class CalculateFacade {

    private Calculate addCalculate;

    private Calculate subCalculate;

    public CalculateFacade() {
        this.addCalculate = new AddCalculate();
        this.subCalculate = new SubCalculate();
    }

    public int add(int a, int b) {
        return this.addCalculate.cal(a, b);
    }

    public int sub(int a, int b) {
        return this.subCalculate.cal(a, b);
    }
}
```

测试：
```java
public class FacadeTest {

    @Test
    public void test() {
        CalculateFacade cf = new CalculateFacade();

        System.out.println(cf.add(1, 1));
        System.out.println(cf.sub(2, 1));
    }
}
```

测试结果：
```powershell
2
1
```

## 总结
让方法调用更加优雅！
