# 定义
为对象动态添加功能。组合模式是部分和整理的关系，而装饰器模式只是额外增加某些功能。

# 实现原理

装饰者（Decorator）和具体组件（ConcreteComponent）都继承自组件（Component）,**装饰者组合了一个组件**，这样它可以装饰其它装饰者或者具体组件。

就是把这个装饰者套在被装饰者之上，从而动态扩展被装饰者的功能。

装饰者的方法有一部分是自己的，这属于它的功能，然后调用被装饰者的方法实现，从而也保留了被装饰者的功能。可以看到，具体组件应当是装饰层次的最低层，因为只有具体组件的方法实现不需要依赖于其它对象。

### 代码实现：
设计不同种类的饮料，饮料可以添加配料，比如可以添加牛奶，并且支持动态添加新配料。每增加一种配料，该饮料的价格就会增加，要求计算一种饮料的价格。

下面实现在 DarkRoast 饮料上新增新添加 Mocha 配料，之后又添加了 Milk 配料。DarkRoast 被 Mocha 包裹，Mocha 又被 Milk 包裹。它们都继承自相同父类，都有 cost() 方法，外层类的 cost() 方法调用了内层类的 cost() 方法。

    
   定义一个组件（Component）接口,这里为饮料。装饰者和具体组件都要实现它。
   
    public interface Beverage {
    
        public double cost();
    }
    
   具体组件实现DarkRoast饮料
   
    public class DarkRoast implements Beverage {
   
       @Override
       public double cost() {
           return 1;
       }
    }
    
   定义一个装饰者抽象类，实现了饮料接口，拥有饮料的方法，并关联了饮料接口对象。
    
    public abstract class CondimentDecorator implements Beverage {
    
        protected Beverage beverage;
    
        public CondimentDecorator(Beverage beverage) {
            this.beverage = beverage;
        }
    }
    
   装饰者实现类Milk，继承了装饰者类
   
    public class Milk extends CondimentDecorator {
    
        public Milk(Beverage beverage) {
            super(beverage);
        }
    
        @Override
        public double cost() {
            return 1 + beverage.cost();
        }
    }
    
   装饰者实现类Mocha，继承了装饰者类
   
    public class Mocha extends CondimentDecorator {
    
        public Mocha(Beverage beverage) {
            super(beverage);
        }
    
        @Override
        public double cost() {
            return 1 + beverage.cost();
        }
    }

   测试：当执行cost()方法时会一层一层从内往外调用。
   
    public class Client {
   
       public static void main(String[] args) {
           Beverage beverage = new DarkRoast();// 一杯饮料
           beverage = new Mocha(beverage);// 饮料里加mocha
           beverage = new Milk(beverage);// 饮料里加牛奶
           System.out.println(beverage.cost());//总价
       }
   }