# 定义
利用共享的方式来支持大量细粒度的对象，这些对象一部分内部状态是相同的。

# 实现原理

在享元接口中 定义一个方法，即为对象的外部状态，每个对象的外部状态都不同
在接口的实现类中添加一个成员变量作为识别标志，在创建工厂类中添加一个Map以识别标志为key，即以对象内部相同状态为key，
调用方法即可改变其它状态

### 代码实现：
    
   定义组享元接口 拥有改变外部对象的方法
    
    public interface Flyweight {
    
        /**
         * 外部状态，每个享元对象的外部状态不同
         *
         * @param extrinsicState
         */
        public void extrinsicState(String extrinsicState);
    }
   
   定义一个享元接口实现类，内部状态为成员变量，提供外部状态实现
   
    public class ConcreteFlyweight implements Flyweight {
    
        private String intrinsicState;
    
        public ConcreteFlyweight(String intrinsicState) {
            this.intrinsicState = intrinsicState;
        }
    
        @Override
        public void extrinsicState(String extrinsicState) {
            System.out.println("Object address: " + System.identityHashCode(this));
            System.out.println("IntrinsicState: " + intrinsicState);
            System.out.println("ExtrinsicState: " + extrinsicState);
        }
    }
   
   定义享元对象工厂类，定义一个全局变量map存储享元对象。
   
    public class FlyweightFactory {
    
        private HashMap<String, Flyweight> flyweights = new HashMap<>();
    
        public Flyweight getFlyweight(String intrinsicState) {
            if (!flyweights.containsKey(intrinsicState)) {
                Flyweight flyweight = new ConcreteFlyweight(intrinsicState);
                flyweights.put(intrinsicState, flyweight);
            }
            return flyweights.get(intrinsicState);
        }
    }  
    
   测试
    
    public class Client {
    
        public static void main(String[] args) {
            FlyweightFactory factory = new FlyweightFactory();
            Flyweight flyweight1 = factory.getFlyweight("aa");
            Flyweight flyweight2 = factory.getFlyweight("aa");
            flyweight1.extrinsicState("x");
            flyweight2.extrinsicState("y");
        }
    }
    
   结果
   
    Object address: 985934102
    IntrinsicState: aa
    ExtrinsicState: x
    Object address: 985934102
    IntrinsicState: aa
    ExtrinsicState: y
   
### JDK使用：
   
Java 利用缓存来加速大量小对象的访问时间。

- java.lang.Integer#valueOf(int)
- java.lang.Boolean#valueOf(boolean)
- java.lang.Byte#valueOf(byte)
- java.lang.Character#valueOf(char)
   