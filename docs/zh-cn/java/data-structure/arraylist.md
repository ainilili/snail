## 引导
``ArrayList``是List大家组的一份子，祖先是``Collection``，也是我们最常用的数据结构之一，它的实现原理正如其名，其内部是一个对象数组：
```
transient Object[] elementData;
```
ArrayList使用一个变量来控制当前的游标，当我们做add或者remove操作的时候就会移动游标：
```
private int size;
```
``ArrayList``大部分的基本操作都是围绕这两个变量展开的。
### add方法
```java
public boolean add(E e) {
   ensureCapacityInternal(size + 1);  // A
   elementData[size++] = e; // B
   return true;
}
```
当我们向ArrayList添加一个元素的时候，在``A``这一步会检测是否需要扩容，扩容的检测机制也很简单：
```java
private void ensureExplicitCapacity(int minCapacity) {
   modCount++;

   // overflow-conscious code
   if (minCapacity - elementData.length > 0)
       grow(minCapacity);
}
```
如果发现``size + 1``大于数组长度，则认为数组不够用，需要扩容，扩容的过程在``grow``方法中进行：
```java
private void grow(int minCapacity) {
   // overflow-conscious code
   int oldCapacity = elementData.length;
   int newCapacity = oldCapacity + (oldCapacity >> 1);
   if (newCapacity - minCapacity < 0)
       newCapacity = minCapacity;
   if (newCapacity - MAX_ARRAY_SIZE > 0)
       newCapacity = hugeCapacity(minCapacity);
   // minCapacity is usually close to size, so this is a win:
   elementData = Arrays.copyOf(elementData, newCapacity);
}
```
扩容时，首先会计算出新的容器大小，然后使用``Arrays.copyOf``复制数组。扩容检测结束，则执行
```java
elementData[size++] = e; // B
```
元素赋值并移动下标！
### remove方法
ArrayList中的remove操作有两种方式，第一种通过下标移除：
```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;  
    if (numMoved > 0) //A
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved); //B
    elementData[--size] = null; // C

    return oldValue;
}
```
首先会在``A``处判断要移除的元素是否是当前尾部元素，如果是，则直接执行``C``，否则，会将该元素之后的所有元素向前移移位，这里同样使用的是``Arrays.copyOf``。

另一种移除方式是通过判断是否是同一个对象来决定是否移除：
```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```
这里很简单，通过遍历``elementData``然后依次判断是否和被移除的元素相等，如果相等则移除，并返回``true``。通过代码可以看到，如果将一个元素执行多次``add``，那么需要执行同样次数的``remove``才可以删干净!
### subList方法
首先说明一下，这个方法要慎用，从方法名字和方法参数来看，让人感觉是将List截取一部分供大家使用，就像String的``subString``方法一样，其返回值是一个新的对象，然而其内部实现却并非如此：
```Java
public List<E> subList(int fromIndex, int toIndex) {
   subListRangeCheck(fromIndex, toIndex, size);
   return new SubList(this, 0, fromIndex, toIndex);
}
```
通过源码可以看到，首先会进行范围检测，之后会返回一个``SubList``对象，貌似没什么问题，别急，我们进一步看一下``SubList``这个类：
```Java
SubList(AbstractList<E> parent,
                int offset, int fromIndex, int toIndex) {
    this.parent = parent;
    this.parentOffset = fromIndex;
    this.offset = offset + fromIndex;
    this.size = toIndex - fromIndex;
    this.modCount = ArrayList.this.modCount;
}
```
该类的构造方法需要传入一个List对象并将其赋值给SubList的parent变量，毋容置疑，这个对象就是当前的ArrayList对象，还有``offset``、``fromIndex``、``toIndex``。

此时我们会发现有些不妙，为啥一个新的List还会有from和to这种东西？于是我们带着好奇心去看一看它的``add``方法：
```Java
public void add(int index, E e) {
    rangeCheckForAdd(index);
    checkForComodification();
    parent.add(parentOffset + index, e); //A
    this.modCount = parent.modCount;
    this.size++;
}
```
注意``A``这一步，parent不就是我们构造方法中传进来的``ArrayList``对象吗！！这里竟然操作的是原来的ArrayList对象，而并非快照！

由此看来，subList方法获取的SubList对象的操作是基于原ArrayList对象内部的``elementData``数组的，如果只读的情况下，这个方法人畜无害，如果想要对其操作，就要慎重使用，因为它会影响原来的ArrayList对象内部元素！
