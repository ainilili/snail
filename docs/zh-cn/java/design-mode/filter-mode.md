## 定义
Filter正如它的含义一样，起到过滤的作用，并允许多种过滤规则相互组合，我们最熟知的例子就是``Tomcat``中的``filter``，它允许我们定义多个过滤器，当请求来临之后，将会逐个匹配并执行。
## 实现
举一个简单的例子，要求将一组数中的负数和偶数过滤掉。

通过简单的分析可知，要求中有两种规则：
 - 过滤负数
 - 过滤偶数

Filter抽象类：
```java
public abstract class AbstractFilter<T> {
    public abstract void filter(List<T> list);
}
```

负数过滤器：
```java
public class NegativeFilter extends AbstractFilter<Integer>{

    @Override
    public void filter(List<Integer> list) {
        for(int index = list.size() -1; index >= 0; index --) {
            if(list.get(index) < 0) list.remove(index);
        }
    }

}
```

偶数过滤器：
```java
public class EvenFilter extends AbstractFilter<Integer>{

    @Override
    public void filter(List<Integer> list) {
        for(int index = list.size() -1; index >= 0; index --) {
            if(list.get(index) % 2 != 1) list.remove(index);
        }
    }

}
```

测试：
```java
public class Test {

    public static void main(String[] args) {

        AbstractFilter<Integer> evenFilter = new EvenFilter();
        AbstractFilter<Integer> negativeFilter = new NegativeFilter();

        List<Integer> list = new ArrayList<Integer>();
        list.addAll(Arrays.asList(1, -1, 5, 3, 4, 7, -9, 100, 27));

        System.out.println("Before filter:" + list);

        negativeFilter.filter(list);
        System.out.println("After negative filter:" + list);

        evenFilter.filter(list);
        System.out.println("After even filter:" + list);

    }
}
```

测试结果：
```powershell
Before filter:[1, -1, 5, 3, 4, 7, -9, 100, 27]
After negative filter:[1, 5, 3, 4, 7, 100, 27]
After even filter:[1, 5, 3, 7, 27]
```
## 总结
这种模式让数据处理更加优雅，配合责任链模式更佳！
