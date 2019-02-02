## 适用场景
1. 一次性实现一个算法的不变的部分，并将可变的行为留给子类来实现。
2. 各子类中公共的行为应被提取出来并集中到一个公共父类中以避免代码重复。
3. 控制子类的扩展。

## 定义
模板方法模式准备一个抽象类，将部分逻辑以具体方法及具体构造子类的形式实现，然后声明一些抽象方法来迫使子类实现剩余的逻辑。不同的子类可以以不同的方式实现这些抽象方法，从而对剩余的逻辑有不同的实现。先构建一个顶级逻辑框架，而将逻辑的细节留给具体的子类去实现。

## 实现原理
定义算法的步骤，并将这些步骤的实现延迟到子类，并将公共的行为提取到公共父类中避免重复

### 代码示例
```java
// 抽象游戏类
public abstract class Game {

    abstract void init();

    abstract void start();

    abstract void end();

    // 模板方法
    final void play() {
        init();
        start();
        end();
    }
}
```
```java
// 棋游戏实现类
public class Chess extends Game {

    @Override
    void init() {
        System.out.println("init chess game");
    }

    @Override
    void start() {
        System.out.println("start chess game");
    }

    @Override
    void end() {
        System.out.println("end chess game");
    }
}

```
```java
// 扑克游戏实现类
public class Poker extends Game {

    @Override
    void init() {
        System.out.println("init poker game");
    }

    @Override
    void start() {
        System.out.println("start poker game");
    }

    @Override
    void end() {
        System.out.println("end poker game");
    }
}
```
```java
// 玩家实现类
public class Player {
    public static void main(String[] args) {
        Game game = new Chess();
        game.play();
        game = new Poker();
        game.play();
    }
}
```

输出：

      init chess game
      start chess game
      end chess game
      
      init poker game
      start poker game
      end poker game
      
### 总结
为我们提供了一种代码复用的重要技巧。

优点：
1. 封装不变部分，扩展可变部分。 
2. 提取公共代码，便于维护。 
3. 行为由父类控制，子类实现。

缺点：
1. 每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大。

为防止恶意操作，一般模板方法都加上`final`关键词。

### 参考
[维基百科](https://en.wikipedia.org/wiki/Template_method_pattern)
