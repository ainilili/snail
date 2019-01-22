## Lambda介绍
Lambda，别名函数式编程，维基百科给出以下介绍：
 > 函数式编程是一种编程范式。它把计算当成是数学函数的求值，从而避免改变状态和使用可变数据。它是一种声明式的编程范式，通过表达式和声明而不是语句来编程。

Lambda表达式基于数学中的λ演算得名，直接对应于其中的lambda抽象(lambda abstraction)，是一个匿名函数，即没有函数名的函数。Lambda表达式可以表示闭包（注意和数学传统意义上的不同）。

λ 演算是数理逻辑中的一个形式系统，在函数抽象和应用的基础上，使用变量绑定和替换来表达计算。讨论 λ 演算离不开形式化的表达。在本文中，我们尽量集中在与编程相关的基本概念上，而不拘泥于数学上的形式化表示。λ 演算实际上是对前面提到的函数概念的简化，方便以系统的方式来研究函数。

## Java中的Lambda
自Java8面世以后，也就代表着java从此以后同样支持lambda语法，使得之前繁琐的操作都可以使用简便的语法进行代替，最具代表性的改革就是新增的Stream类，让我们对一个集合的排序、过滤、映射和采集更加方便！

我们拟定一个场景，对于给定的一个int数组，过滤掉负数，并对剩余的元素进行排序，在java8之前我们的实现需要这么写：
```java
int[] array = {7, -2, 3, 5, -9, 3, -5, -1, 6, 8, 20};
List<Integer> list = new ArrayList<Integer>();
//过滤负数
for(int i: array) {
    if(i >= 0) list.add(i);
}
//排序
Collections.sort(list);

for(int i: list) {
    System.out.println(i);
}
```

使用Stream之后：
```java
int[] array = {7, -2, 3, 5, -9, 3, -5, -1, 6, 8, 20};
Arrays.stream(array)
    .filter(a -> a >= 0)    //过滤
    .sorted()               //排序
    .forEach(System.out::println);
```
可以看到，实现的过程更加简洁和优雅，lambda大大节省了代码空间，提升了代码可读性，但使用的难度也随之提高，对于传统的编程方式，lambda语法无疑是一次重大的冲击。

## Java中Lambda语法的使用
### 函数式接口
什么是**函数式接口**呢？在Java8之前，我们想实现一个接口，最简单的方式直接使用匿名类：
```java
Comparator<Integer> comparator = new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1 > o2 ? 1 : -1;
    }
};
```
这里要注意，Comparator是一个接口类型，它的内部只有一个需要被实现的方法，那么我们将之称之为函数式接口，一般的函数式接口都会加上``@FunctionalInterface``注解，如果该接口待实现的方法超出两个，你的IDE就会提醒你这不是一个规范的函数式接口，对于符合的，我们就可以使用lambda语法进行初始化：
```java
Comparator<Integer> comparator = (o1, o2) -> o1 > o2 ? 1 : -1;
```
将之与java8之前的实现对比，我们发现有很多共同之处，我们来分析一下lambda的实现：
```java
(o1, o2) -> o1 > o2 ? 1 : -1;
```
将上部分以``->``做分割线，分成两部分，它们分别是``(o1, o2)``和``o1 > o2 ? 1 : -1``。很明显，前者代表着函数的两个入参，后者代表着两个入参的逻辑实现，由此可得，lambda由两部分组成：``入参定义``和``逻辑实现``。

对于一个函数式接口，我们可以用简单的lambda语法去实现接口内唯一的待实现方法，反推一下，对于lambda这种匿名的函数定义风格，如果一个接口存在两个待实现的方法，lambda则无法具体表示实现的是哪一个方法，由此反推可得，一个函数式接口最多只能有一个待实现方法。
### JDK对Lambda的支持
通过函数式接口的定义和lambda实现我们知道了lambda语法的一个简单格式，但是在开发过程中，我们不可能对于每一个lambda的应用都定义个函数式接口，实际上，JDK中已经存在了很多lambda函数：
 - **Function<T, R>**：接受一个参数输入，输入类型为 T，输出类型为 R。 抽象方法为``R apply(T)``。
 - **BiFunction<T, U, R>**：接受两个参数输入, T 和 U 分别是两个参数的类型，R 是输出类型。抽象方法为``R apply(T, U)``。
 - **Consumer<T>**：接受一个输入，没有输出。抽象方法为 ``void accept(T t)``。
 - **Predicate<T>**：接受一个输入，输出为 boolean 类型。抽象方法为 ``boolean test(T t)``。
 - **Supplier<T>**：没有输入，一个输出。抽象方法为 ``T get()``。
 - **BinaryOperator<T>**：接受两个类型相同的输入，输出的类型与输入相同，相当于 BiFunction<T,T,T>。
 - **UnaryOperator<T>**：接受一个输入，输出的类型与输入相同，相当于 Function<T, T>。
 - **BiPredicate<T, U>**：接受两个输入，输出为 boolean 类型。抽象方法为 boolean test(T t, U u)。

它们分别应用于不同的场景，以下将会有几个演示，首先使用lambda实现一个计算器：
```java
BinaryOperator<Integer> cal = (a, b) -> a + b;
System.out.println(bo.apply(1, 2)); // 3
```
再来一个，使用lambda实现对数字正负的判断
```java
int a = 1;
int b = -1;
Predicate<Integer> predicate =  i -> i >= 0;
System.out.println(predicate.test(a));  //true
System.out.println(predicate.test(b));  //false
```
## 总结
在Stream中，lambda的应用非常广泛，我们如果想讲lambda更熟练的掌握，需要自己亲自的去使用lambda，在实战中去真正体会lambda的强大之处。
## 参考文章
 - [深入理解Java函数式编程]https://www.ibm.com/developerworks/cn/java/j-understanding-functional-programming-1/index.html?ca=drs-
 - [Java 8 中的 Streams API 详解]https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/
 - [百度百科——Lambda表达式](https://baike.baidu.com/item/Lambda%E8%A1%A8%E8%BE%BE%E5%BC%8F)
