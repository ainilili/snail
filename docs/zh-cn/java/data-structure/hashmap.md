## 引导
在了解``HashMap``之前，我们应该先明白两个概念：``Hash``和``Map``，这可以帮助我们更容易了解``HashMap``的运行原理。

那么何为``Hash``，又何为``Map``呢？

### Hash
之前写过一篇关于Hash的文章 [Hash](/zh-cn/java/data-structure/hash.md)

### Map
Map是一种``K-V``形式的数据结构，一个唯一的key，会唯一对应一个value。也就是说，在Map容器里不允许两个一模一样的key。

一个简单的Map结构如下：
```java
{
  "key1":"value1",
  "key2":"value2",
  "key3":"value3"
}
```
对于这种数据结构，并且Map会对外提供一些方法来实现对内部数据的操作：
```java
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

这里有两个数据结构需要我们知道：
 - **Table**：哈希表，存放Node元素。
 - **Node**：结点元素，存放``K-V``组对信息，其结构是一个链表/红黑树。

另外，在HashMap内部有一些关键属性我们也要了解一下：
 - **DEFAULT_INITIAL_CAPACITY**：Table数组初始长度，默认为``1 << 4``，``2^4`` = 16。
 - **MAXIMUM_CAPACITY**：Table数组最高长度，默认为``1 << 30``，``2^30`` = 1073741824。
 - **DEFAULT_LOAD_FACTOR**：负载因子，当总元素数 > 数组长度 * 负载因子时，Table数组将会扩容，默认为0.75。
 - **TREEIFY_THRESHOLD**：树化阈值，当单个Table内Node数量超过该值，则会将链表转化为红黑树，默认为8。
 - **UNTREEIFY_THRESHOLD**：链化阈值，当扩容期间单个Table内Entry数量小于该值，则将红黑树转化为链表，默认为6。
 - **MIN_TREEIFY_CAPACITY**：最小树化阈值，当Table所有元素超过改值，才会进行树化（为了防止前期阶段频繁扩容和树化过程冲突）。
 - **size**：Table数组当前所有元素数。
 - **threshold**：下次扩容的阈值（数组长度 * 负载因子）

HashMap的内部有着一个Table数组，而这个数组的初始长度为``DEFAULT_INITIAL_CAPACITY``参数值，Table数组存放的元素类型就是Node，它是一个单向链表：
```java
static class Node<K,V> implements Map.Entry<K,V> {
  final int hash; //key的hash值
  final K key;  //key
  V value;  //value
  Node<K,V> next; //下一个结点
}
```
每个Table中存的Node元素相当于链表的``header``，``next``指向下一个结点，而这种链式结构的存在正是为了解决``hash冲突``：
> **hash冲突**：两个元素的经过Hash散列之后分在同一个组内，我们将之解释为Hash冲突

在JDK1.7之前的版本，hash冲突的解决方法是将被冲突的Node结点放于一个链表中，而Table中的元素则是链头，当然在JDK1.8中，当Table中链长超过``TREEIFY_THRESHOLD``阈值后，将会将链表转变为红黑树的实现``TreeNode``：
```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
  TreeNode<K,V> parent;  // red-black tree links
  TreeNode<K,V> left;
  TreeNode<K,V> right;
  TreeNode<K,V> prev;    // needed to unlink next upon deletion
  boolean red;
}
```
当发生hash冲突的Node不断变多，那么这个链将会越来越长，那么遍历碰撞key时的耗时就会不断增加，这也就直接导致了性能的不足，从JDK1.8开始，HashMap对于单个Table中的Node超出某个阈值时，将会开始树化操作（链表转化为红黑树），这对于搜索的性能将会有很大的提升，而插入和删除的操作所带来的性能影响微乎其微。

### put方法
在``HashMap``的内部会有一个Table数组，这个数组的当前长度就是我们要实现映射的目标范围，当我们执行``put``方法时，``key``和``value``要经历这些事情：
 - 通过``Hash``散列获取到对应的Table
 - 遍历Table下的Node结点，做更新/添加操作
 - 扩容检测

具体实现我们可以根据源码来详细了解一下：
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        // HashMap的懒加载策略，当执行put操作时检测Table数组初始化。
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        //通过``Hash``函数获取到对应的Table，如果当前Table为空，则直接初始化一个新的Node并放入该Table中。
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //输入的key命中了当前Table的首元素，直接更新。
            e = p;
        else if (p instanceof TreeNode)
            //如果当前Node类型为TreeNode，调用``putTreeVal``方法。
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //如果不是TreeNode，则就是链表，遍历并与输入key做命中碰撞。
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    //如果当前Table中不存在当前key，则添加。
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        //超过了``TREEIFY_THRESHOLD``则转化为红黑树。
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    //做命中碰撞，使用hash、内存和equals同时判断（不同的元素hash可能会一致）。
                    break;
                p = e;
            }
        }
        if (e != null) {
            //如果命中不为空，更新操作。
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        //扩容检测。
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
对于其过程中的关于Node链表和红黑树的转换过程我们可以暂时屏蔽掉，那么整个流程并不是很绕，那么我们继续深入的来看一下HashMap的扩容实现。

### resize方法
HashMap的扩容大致的实现是将老Table数组中所有的Entry取出来，重新对其hashcode做``Hash``散列到新的新的Table之中，也就是一个``re-put``的过程，具体还是通过源码来讲解：
```java
final Node<K,V>[] resize() {
    //保留老的hash表
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    //如果之前的容量大于0
    if (oldCap > 0) {
        //如果超出最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            //扩容阈值为int最大值
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //否则计算扩容后的阈值
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0)
        // 如果之前的容量等于0，并且之前的阈值大于零，则新的hash表长度就等于它
        newCap = oldThr;
    else {              
        // 初始阈值为零表示使用默认值
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    //如果新的阈值为 0 ，就得用 新容量*加载因子 重计算一次
    if (newThr == 0) {

        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    //常见扩容后的hash表
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab; //A
    if (oldTab != null) {
        //遍历旧的hash表，将之内部元素转移到新的hash表
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    //如果当前Table内只有一个元素，重新做hash散列并赋值
                    newTab[e.hash & (newCap - 1)] = e; //B
                else if (e instanceof TreeNode)
                    //如果旧哈希表中这个位置的桶是树形结构，就要把新哈希表里当前桶也变成树形结构
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { //保留旧哈希表桶中链表的顺序
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {  //遍历当前Table内的Node，赋值给新的Table
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
### get方法
在我们看完HashMap对于put方法的实现之后，get方法则显得简单易懂，其代码与put相近无几，主要差别是没有了扩容和添加/更新的操作：
```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //判断hash表是否为空，表重读是否大于零并且当前key对应分布的表内是否有Node存在
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash &&
            ((k = first.key) == key || (key != null && key.equals(k))))
            // 检测第一个Node，命中则不需要在做do...while...循环
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                //如果Table内是树形结构，则使用对应的检索方法
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do { //如果是链表，则做while循环，直到命中或者遍历结束
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
### containsKey方法
根据get方法的结果是否为空就可以直到是否包含该key：
```java
public boolean containsKey(Object key) {
    return getNode(hash(key), key) != null;
}
```
### remove方法
同样类似于put操作，首先会查找对应的key所在位置，如果为空，则不操作，反之，将之移除：
```java
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    //判断hash表是否为空，表重读是否大于零并且当前key对应分布的表内是否有Node存在
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            // 第一个Node命中
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                //如果Table内是树形结构，则使用对应的检索方法
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do { //如果是链表，则做while循环，直到命中或者遍历结束
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            //如果命中到了对应的Node，则根据Node结构进行对应的移除操作
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            //修改hash表元素数
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```
## 为何线程不安全？
看完了HashMap的实现之后，就该谈一谈它为什么存在线程安全问题！

### 数据丢失
首先，我们将目光放在put方法的实现中，假设有两个线程在同时进行put操作，对应的数据分别为：
```
thread-1： put(1, 'abc');
thread-2： put(1, 'efg');
```
假设此时Hash表的长度为10，且已经有两个元素在，负载因子为默认值0.75f，那么操作过程一定不会扩容，并且两个线程put的key都是1，那么它们将会分配到同一个table中，下方代码为put方法中的其中一段，其主要作用是遍历当前表内Node，寻找与当前key一样的Node结点，之后再做添加/更新操作。
```java
for (int binCount = 0; ; ++binCount) {
   if ((e = p.next) == null) {
       p.next = newNode(hash, key, value, null); // A
       if (binCount >= TREEIFY_THRESHOLD - 1)
           treeifyBin(tab, hash);
       break;
   }
   if (e.hash == hash &&
       ((k = e.key) == key || (key != null && key.equals(k))))
       break;
   p = e;
}
```
假设两个线程同时执行到了``A``这个位置，此时获取到的``p``是统一个对象，下一刻，cpu运转，两个线程同时运行，那么``p.next``的值将会是最后一个线程put的value值，而前一个则会丢失，这就会导致丢数据的情况！

当然该情景同样会发生于``resize``和``remove``操作，至于为什么，大家可以思考一下！
### size不准确
这个就很简单了，为什么不准确呢，来看一下size变量在HashMap内部的定义：
```
transient int size;
```
内存不可见并且增减操作未加锁，多线程操作下属于非原子操作！
### 闭环死锁
这个问题在JDK1.8版本的HashMap中已经不存在了，至于为啥，我要先讲一下在1.8之前的HashMap为什么会存在闭环死锁问题！

从``闭环``这个名词上我们分析一下是什么问题，什么是闭环的，如果链表形成了一个环会不会就是闭环呢？而链表如何才会形成环？带着这些问题，我们在脑海中抽象出一个模型：
```
graph LR
A-->B
B-->A
```
假设某一个Table中的Node链表发生了上述问题，那么我们在遍历时进行``do{ }while ((e = e.next) != null);``操作就会发生死锁的问题，那么看来我们的猜想方向是正确的，那么我们就具体分析一下HashMap在什么操作之中会产生闭环的问题，不过在此之前，我们要明白因果：
```
因：???
果：闭环
```
我们知道，只有当两个结点内部的``next``相互引用对方的时候才会死锁，这种场景只能在两个已经存在同一个链上的结点同时以``相反的方向``被操作``next``引用的时候才会发生，而在HashMap内部，符合这种场景的只有一个方法：``resize``，那我们就来看一下JDK1.7的``resize``方法实现：
```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    boolean oldAltHashing = useAltHashing;
    useAltHashing |= sun.misc.VM.isBooted() &&
            (newCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
    boolean rehash = oldAltHashing ^ useAltHashing;
    //fu
    transfer(newTable, rehash);
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```
进入``transfer``方法中，其内部实现了扩容过程：
```java
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) { // A
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```
我们发现，在JDK1.7的HashMap的扩容实现中，老的Table中的Node链的顺序赋值给新的Table中时的操作是反置的：
```java
e.next = newTable[i];
newTable[i] = e;
e = next;
```
上述操作是将当前Node的next指针指向当前Table的头结点，之后当前Node又变为了Table的头结点，此时假设A、B两个线程同时执行到了``transfer``方法中的``A``位置，并且此时的``oldTable``和``newTable``的结构是这样的：
```
oldTable[]
table-1: a -> b -> c -> null
table-2: null
table-3: null

newTable[]
table-1: null
table-2: null
table-3: null
table-4: null
table-5: null
table-6: null
```
如果很巧，两个线程在同一个CPU上执行，那么就会存在一个抢占时间片的场景，假设A先抢到了时间片，然后执行一番操作之后，``oldTable``和``newTable``的结构如下：
```
oldTable[]
table-1: a -> null
table-2: null
table-3: null

newTable[]
table-1: null
table-2: c -> b -> a
table-3: null
table-4: null
table-5: null
table-6: null
```
之后还没等它做``oldTable = newTable``操作，B抢到了时间片，并也做了同样一番操作，``oldTable``和``newTable``的结构如下：
```
oldTable[]
table-1: a -> null
table-2: null
table-3: null

newTable[]
table-1: null
table-2: a -> c -> b -> a
table-3: null
table-4: null
table-5: null
table-6: null
```
此时A或者B谁先``oldTable = newTable``已经无所谓了，因为``newTable``中已经产生了闭环，之后在进行get或者put操作时，如果不小心触发到了while循环，那将会一直死循环：
```java
do{
  //do some thing
}while ((e = e.next) != null);  //e = e.next将会永不为空
```
从上述场景产生的过程中我们发现，``a -> c -> b -> a``这种闭环问题的罪魁祸首是因为1.7中的HashMap在扩容时为了免去再次遍历链表，很聪明的将当前结点作为新链表的头结点，这就会导致顺序反转，所以无序化导致了闭环的产生，而这种问题不仅仅是在HashMap中体现，Mysql的死锁问题的原因常常也是因为反序加行锁导致的！

而在开头说过，JDK1.8已经避免了这个问题，这是为什么呢？看下代码就知道了：
```java
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
       newTab[j] = loHead;  //A
   }
   if (hiTail != null) {
       hiTail.next = null;
       newTab[j + oldCap] = hiHead; //B
   }
}
```
同样是扩容的操作，JDK1.8中的HashMap通过两个链分别去存储头结点和尾结点以保证它有序，并且不会频繁的去赋值``newTable``，而是在循环之后直接赋值（请注意A、B标记处），这样就非常简单的避免了产生闭环的陷阱！
