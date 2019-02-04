## 适用场景
1. 行为随状态改变而改变的场景。 
2. 条件. 分支语句的代替者。

> 代码中包含大量与对象状态有关的条件语句，消除“复杂”的 `if...else`等条件选择语句。

## 定义
让一个对象在其内部状态改变的时候，其行为也随之改变。状态模式需要对每一个系统可能获取的状态创立一个状态类的子类。当系统的状态变化时，系统便改变所选的子类。

## 实现原理
通过更改封装对象内部状态，可以在运行时更改行为，无需借助条件语句，提高可维护性。

### 代码示例
状态接口：State
```java
public interface State {

    void action(StateContext context, String name);

}
```
初始状态类
```java
public class InitializeState implements State {

    @Override
    public void action(StateContext context, String name) {
        System.out.println("init state");
        context.setState("admin".equals(name) ? new FinishState() : new CancelState());
        context.doAction(name);
    }
}
```
撤销状态类
```java
public class CancelState implements State {

    @Override
    public void action(StateContext context, String name) {
        System.out.println("cancel state");
    }
}
```
完成状态类
```java
public class FinishState implements State {

    @Override
    public void action(StateContext context, String name) {
        System.out.println("finish state");
    }
}
```
状态上下文类
```java
public class StateContext {

    private State state;

    public StateContext() {
        this.state = new InitializeState();
    }

    public void setState(State state) {
        this.state = state;
    }

    public void doAction(String name) {
        state.action(this, name);
    }
}
```
测试类
```java
public class StateClient {

    public static void main(String[] args) {
        StateContext context = new StateContext();
        context.doAction("admin");
        System.out.println("---------------");
        context = new StateContext();
        context.doAction("user");
    }
}
```
输出：

    init state
    finish state
    ---------------
    init state
    cancel state
    
### 总结
状态模式允许一个对象基于内部状态拥有不同的行为，Context会将行为委托个当前状态对象，但是其对`开闭原则`支持不是很好

优点：
 1. 封装了转换规则。 
 2. 枚举可能的状态，在枚举状态之前需要确定状态种类。 
 3. 将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为。 
 4. 允许状态转换逻辑与状态对象合成一体，而不是某一个巨大的条件语句块。 
 5. 可以让多个环境对象共享一个状态对象，从而减少系统中对象的个数。

缺点：
1. 状态模式的使用必然会增加系统类和对象的个数。 
2. 状态模式的结构与实现都较为复杂，如果使用不当将导致程序结构和代码的混乱。 
3. 状态模式对"开闭原则"的支持并不太好，对于可以切换状态的状态模式，增加新的状态类需要修改那些负责状态转换的源代码，否则无法切换到新增状态，而且修改某个状态类的行为也需修改对应类的源代码。

### 参考
[维基百科](https://en.wikipedia.org/wiki/State_pattern)
