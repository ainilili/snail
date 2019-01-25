# 定义

定义一系列算法，封装每个算法，并使它们可以互换。策略模式可以让算法独立于使用它的客户端。

# 实现原理

把对象的功能实现定义成一个接口，对象中依赖一个功能接口，通过改变接口的实现，来达到不同的功能。

### 代码实现：
    
   定义功能接口
    
    public interface CallBehavior {
   
       // 叫，不同动物叫声不同
       void call();
    }
   
   功能接口实现
   
    public class Quack implements CallBehavior {
    
        @Override
        public void call() {
            System.out.println("鸭子叫!");
        }
    }
   
    public class Squeak implements CallBehavior {
   
       @Override
       public void call() {
           System.out.println("鸡叫k!");
       }
    }
    
    
   定义对象类，并依赖功能接口，通过改变接口的具体实现达到不同的功能
   
    public class Duck {
    
        private CallBehavior behavior;
    
        public void performQuack() {
            if (behavior != null) {
                behavior.call();
            }
        }
        public void setQuackBehavior(CallBehavior behavior) {
            this.behavior = behavior;
        }
    }
    
   测试：
         
    public class Client {
    
        public static void main(String[] args) {
            Duck duck = new Duck();
            CallBehavior quack = new Quack();
            duck.setQuackBehavior(quack);
            duck.performQuack();
            CallBehavior squeak = new Squeak();
            duck.setQuackBehavior(squeak);
            duck.performQuack();
        }
    }

    
   输出：
   
    鸭子叫!
    鸡叫k!
    
### JDK实现

- java.util.Comparator#compare()
- javax.servlet.http.HttpServlet
- javax.servlet.Filter#doFilter()