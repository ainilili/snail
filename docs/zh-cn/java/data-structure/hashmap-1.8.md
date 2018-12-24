## 引导
在了解``HashMap``之前，我们应该先明白两个概念：``Hash``和``Map``，这可以帮助我们更容易了解``HashMap``的运行原理。

那么何为``Hash``，又何为``Map``呢？

### Hash散列
所谓``Hash``，可以看成是一个函数，我们称之为``散列函数``：
 - **散列函数**：就是把任意长度的输入（又叫做预映射pre-image）通过散列算法变换成固定长度的输出，该输出就是散列值

简单来说，``Hash``函数可以将一个范围的数字映射到另一个范围之中，这种需求在很多场景都非常有用，如``分库分表``，``负载均衡``等等，可以将之抽象为一句话：``给输入的值分组``，当然，看完这篇文章你就会明白这句话所表达的真正的含义！

在我们日常使用中，我们的``Hash``散列更多的应用于处理对象的分组之中，但是``Hash``的入参大多都是面向数字的，那么我们该如何对对象进行散射呢？这里就要引入一个新的概念：``HashCode``。

有Java基础的同学都知道，在Java中，Object对象是每一个普通对象的父类，这就导致每个对象都有一个``hashCode()``方法，而部分对象对该方法进行了重写。我们可以通过该方法获取到当前对象的``哈希码``,简称``hashCode``：
 - **哈希码(hashCode)**：哈希码并不是完全唯一的，它是一种算法，让同一个类的对象按照自己不同的特征尽量的有不同的哈希码，但不表示不同的对象哈希码完全不同。也有相同的情况，看程序员如何写哈希码的算法。

``hashCode``本身的出现就是为了提高哈希表的性能，当然我们也可以写一个自己的哈希算法去计算一个对象的hash值，对于概念上来讲，我们只需要明白可以通过一个java对象的``hashCode``使用``Hash``函数，获取到该对象所属的分组即可，而一个最简单的``Hash``函数实现就是``hashCode``对于目标范围最大值做``Mod``运算：
```
int hash(int hashcode, int maxlen){
  return hashcode & maxlen;
}
```

当然在实际中，我们可能会对hashcode做一些额外的移位，逻辑运算等操作使之分布更加均匀。
### Map
Map是一种``K-V``形式的数据结构，一个唯一的key，会唯一对应一个value。也就是说，在Map容器里不允许两个一模一样的key。

一个简单的Map结构如下：
```
{
  "key1":"value1",
  "key2":"value2",
  "key3":"value3"
}
```
对于这种数据结构，并且Map会对外提供一些方法来实现对内部数据的操作：
```
V put(K key, V value)
V get(Object key)
V remove(Object key)
boolean containsKey(Object key)
```
可见Map对于我们操作``K-V``形式的数据非常方便，实现的方式有很多，最简单粗暴的实现方式是使用``List``来存储每一个``K-V``组对，对于每种方法的实现只需要暴力循环碰撞即可，对于少量数据这种做法未必不可，如果数据量庞大之千万，我们就要换一种更加高效，速度更快的实现方式：``HashMap``。
## HashMap
Map在Java中的实现有很多，``HashMap``便是其中之一，在``JDK``漫长的版本更新中，``HashMap``的实现也是在不断的更新着：
 - **<=JDK1.7**：Table数组 + Entry链表
 - **>=JDK1.8**：Table数组 + Entry链表/红黑树

本文我们跳过JDK1.7的实现，来看一下1.8中``HashMap``源码所带来的魅力冲击！
### 实现原理
对于各个版本的``HashMap``实现原理，主线流程都是一成不变的：

![hashmap原理流程图](https://github.com/ainilili/snail/blob/master/docs/images/hashmap-1.8-1-1.jpg?raw=true)

这里有两个结构需要我们知道：
 - **Table**：哈希表，存放Entry元素。
 - **Entry**：存放``K-V``组对信息，其结构是一个链表/红黑树。

另外，在HashMap内部有一些关键属性我们也要了解一下：
 - **DEFAULT_INITIAL_CAPACITY**：Table数组初始长度，默认为``1 << 4``，``2^4`` = 16。
 - **MAXIMUM_CAPACITY**：Table数组最高长度，默认为``1 << 30``，``2^30`` = 1073741824。
 - **DEFAULT_LOAD_FACTOR**：负载因子，当总元素数 > 数组长度 * 负载因子时，Table数组将会扩容，默认为0.75。
 - **TREEIFY_THRESHOLD**：树化阈值，当单个Table内Entry数量超过该值，则会将链表转化为红黑树，默认为8。
 - **UNTREEIFY_THRESHOLD**：链化阈值，当扩容期间单个Table内Entry数量小于该值，则将红黑树转化为链表，默认为6。
 - **MIN_TREEIFY_CAPACITY**：最小树化阈值，当Table所有元素超过改值，才会进行树化（为了防止前期阶段频繁扩容和树化过程冲突）。
 - **size**：Table数组当前所有元素数。
 - **threshold**：下次扩容的阈值（数组长度 * 负载因子）

### Put操作
在``HashMap``的内部会有一个Table数组，这个数组的当前长度就是我们要实现映射的目标范围，当我们执行``put``方法时，``key``和``value``要经历这些事情：
 - 通过``Hash``散列获取到对应的Table
 - 遍历Table下的Entry元素，做更新/添加操作
- 检测扩容及扩容

具体实现我们可以根据源码来详细了解一下：
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length; // A
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null); //B
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p; //C
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value); //D
        else {
            for (int binCount = 0; ; ++binCount) { //E
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null); //F
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash); //G
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k)))) //H
                    break;
                p = e;
            }
        }
        if (e != null) { //I
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize(); //J
    afterNodeInsertion(evict);
    return null;
}
```
注意代码中的标记号，它们都是精髓：
 - **A**：HashMap的懒加载策略，当执行put操作时检测Table数组初始化。
 - **B**：通过``Hash``函数获取到对应的Table，如果当前Table为空，则直接初始化一个新的Node并放入该Table中。
 - **C**：输入的key命中了当前Table的首元素，直接更新。
 - **D**：如果当前Node类型为TreeNode，调用``putTreeVal``方法。
 - **E**：如果不是TreeNode，则就是链表，遍历并与输入key做命中碰撞。
 - **F**：如果当前Table中不存在当前key，则添加。
 - **G**：超过了``TREEIFY_THRESHOLD``则转化为红黑树。
 - **H**：做命中碰撞，使用hash、内存和equals同时判断（不同的元素hash可能会一致）。
 - **I**：如果e不为空，更新操作。
 - **J**：扩容检测并扩容。

对于其过程中的关于Entry链表和红黑树的转换过程我们可以暂时屏蔽掉，那么整个流程并不是很绕，那么我们继续深入的来看一下HashMap的扩容实现。

### Resize操作
HashMap的扩容大致的实现是将老Table数组中所有的Entry取出来，重新对其hashcode做``Hash``散列到新的新的Table之中，也就是一个``re-put``的过程，具体还是通过源码来讲解：
```Java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```
