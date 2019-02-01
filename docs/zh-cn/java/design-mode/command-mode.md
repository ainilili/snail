## 适用场景
在某些场合，比如要对行为进行"记录、撤销/重做、事务"等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将"行为请求者"与"行为实现者"解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。

## 定义
将一个请求封装为一个对象，从而使我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作(Action)模式或事务(Transaction)模式。

## 实现原理


### 代码示例
```java
// 抽象命令类
    public interface Command {
        void execute(int i);
    }
```
```java
// 电视机类
    public class Television {
        public void open() {
            System.out.println("打开电视");
        }
    
        public void close() {
            System.out.println("关闭电视");
        }
    
        public void changeChannel(int i) {
            System.out.println("切换频道" + i);
        }
    }
```
```java  
// 遥控器类
    public class Control {
        private Command openTVCommand;
        private Command closeTVCommand;
        private Command changeChannelCommand;
    
        public int nowChannel = 0;       //当前频道
        public int priorChannel;     //前一个频道，用于执行返回操作
    
        public Control(Command openTVCommand, Command closeTVCommand,
            Command changeChannelCommand) {
            this.openTVCommand = openTVCommand;
            this.closeTVCommand = closeTVCommand;
            this.changeChannelCommand = changeChannelCommand;
        }
    
        public void open() {
            openTVCommand.execute(0);
        }
    
        public void close() {
            closeTVCommand.execute(0);
        }
    
        public void change() {
            priorChannel = nowChannel;
            nowChannel ++;
            changeChannelCommand.execute(nowChannel);
        }
    
        public void changeUndo() {
            changeChannelCommand.execute(priorChannel);
            int tempChannel;
            tempChannel = priorChannel;
            priorChannel = nowChannel;
            nowChannel = tempChannel;
        }
    }
```
具体命令对象：
```java
    public class OpenCommand implements Command {
    
        private Television tv;
    
        public OpenCommand(Television tv) {
            this.tv = tv;
        }
    
        @Override
        public void execute(int i) {
            tv.open();
        }
    }
```
```java
    public class CloseCommand implements Command{
    
        private Television tv;
    
        public CloseCommand(Television tv) {
            this.tv = tv;
        }
    
        @Override
        public void execute(int i) {
            tv.close();
        }
    }
```
```java
    public class ChangeChannelCommand implements Command{
    
        private Television tv;
    
        public ChangeChannelCommand(Television tv) {
            this.tv = tv;
        }
    
        @Override
        public void execute(int i) {
            tv.changeChannel(i);
        }
    }
```

```java
// 客户端
    public class Client {
    
        public static void main(String[] args) {
            Command openCommand,closeCommand,changeCommand;
            Television tv = new Television();
            openCommand = new OpenCommand(tv);
            closeCommand = new CloseCommand(tv);
            changeCommand = new ChangeChannelCommand(tv);
            Control control = new Control(openCommand, closeCommand, changeCommand);
            control.open();
            control.change();
            control.change();
            control.changeUndo();
            control.changeUndo();
            control.changeUndo();
            control.close();
        }
    
    }
```

  输出：
  
    打开电视
    切换频道1
    切换频道2
    切换频道1
    切换频道2
    切换频道1
    关闭电视

### 总结
命令模式本质上就是将命令封装打包，将命令入口和命令执行对象职责进行分离。
命令发送者只需要如何发送命令，不需要关心命令执行过程。
在发送者和接收者两者间是通过命令对象进行沟通的。请求命令本身就当做一个对象在两者间进行传递，它封装了接收者和一组动作。

优点：
1. 解耦，降低了系统的耦合性
2. 新的命令可以很容易添加到系统中去

缺点：
1. 可能会导致某些系统有过多的具体命令类

### 参考
[维基百科](https://en.wikipedia.org/wiki/Command_pattern)
