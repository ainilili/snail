# 定义
定义对象之间的一对多依赖，当一个对象状态改变时，它的所有依赖都会收到通知并且自动更新状态。

主题（Subject）是被观察的对象，而其所有依赖者（Observer）称为观察者。

# 实现原理

主题依赖了观察者的一个集合，观察者依赖了一个主题，生成一个观察者需要在主题进行注册，主题更新后对每个观察者进行推送

### 代码实现：
    
   定义主题接口，拥有对观察者的操作方法
    
    public interface Subject {
    
        // 注册观察者
        public void registerObserver(Observer observer);
    
        // 移除观察者
        public void removeObserver(Observer observer);
    
        // 提醒观察者
        public void notifyObserver();
    }
   
   定义一个观察者接口，提供一个操作接口，方便主题更新时对观察者进行操作
   
    public interface Observer {
    
        void update(float temp, float humidity, float pressure);
    }
   
   定义主题接口实现类，依赖了一个观察者集合，当对主题进行更新时，循环通知各个观察者
   
    public class WeatherSubject implements Subject {
    
        private List<Observer> observers;
        private float temperature;
        private float humidity;
        private float pressure;
    
        public WeatherSubject() {
            observers = new ArrayList<>();
        }
    
        public void setMeasurements(float temperature, float humidity, float pressure) {
            this.temperature = temperature;
            this.humidity = humidity;
            this.pressure = pressure;
            notifyObserver();
        }
    
        @Override
        public void registerObserver(Observer observer) {
            observers.add(observer);
        }
    
        @Override
        public void removeObserver(Observer observer) {
            int i = observers.indexOf(observer);
            if (i >= 0) {
                observers.remove(i);
            }
        }
    
        @Override
        public void notifyObserver() {
            for (Observer o : observers) {
                o.update(temperature, humidity, pressure);
            }
        }
    }
    
   定义观察者实现类，每个观察者通过构造方法传入它所订阅的主题，并在主题中注册该观察者
    
    public class StatisticsDisplay implements Observer {
    
        public StatisticsDisplay(Subject weatherData) {
            weatherData.registerObserver(this);
        }
    
        @Override
        public void update(float temp, float humidity, float pressure) {
            System.out.println("StatisticsDisplay.update: " + temp + " " + humidity + " " + pressure);
        }
    }
    
   定义另一个观察者实现类
      
    public class CurrentConditionsDisplay implements Observer {
    
        public CurrentConditionsDisplay(Subject weatherData) {
            weatherData.registerObserver(this);
        }
    
        @Override
        public void update(float temp, float humidity, float pressure) {
            System.out.println("CurrentConditionsDisplay.update: " + temp + " " + humidity + " " + pressure);
        }
    
    }
   
   测试：
         
    public class Client {
    
        public static void main(String[] args) {
            WeatherSubject subject = new WeatherSubject();
            Observer statis = new StatisticsDisplay(subject);
            Observer current = new CurrentConditionsDisplay(subject);
            subject.setMeasurements(22f, 29f, 100f);
        }
    }
    
   输出：
   
    StatisticsDisplay.update: 22.0 29.0 100.0
    CurrentConditionsDisplay.update: 22.0 29.0 100.0
    
### JDK实现

第一眼看到这个模式就感觉这不就是java的监听器嘛，其实监听器就是通过这种模式实现的。

- java.util.Observer
- java.util.EventListener
- javax.servlet.http.HttpSessionBindingListener