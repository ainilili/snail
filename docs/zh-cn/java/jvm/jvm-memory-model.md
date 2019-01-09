## 内存模型

![jvm内存模型](https://github.com/ainilili/snail/blob/master/images/jvm-memory-model-1-1.jpg?raw=true)

### 虚拟机栈
每个线程都有个私有的栈，随着线程的创建而创建。栈里面存着的是一种叫“栈帧”的东西，每个方法都会创建一个栈帧，栈帧中存放了局部变量（基本数据类型和对象引用）、操作数栈、方法出口等信息。栈的大小可以固定也可以动态扩展。当栈调用深度大于JVM所允许的返回，会抛出stackoverflowerror的错误。不过这个深度范围不是一个恒定的值。

虚拟机还有另一种错误，那就是当申请不到空间时，会抛出OutOfMemoryError。这里有个小细节需要注意，catch 捕获到的是throwable，而不是exception。因为stackoverflowerror和OutOfMemoryError都不书序Exception 的子类。
### 本地方法栈
这部门主要与虚拟机用到的本地native方法相关，一般情况下，java程序员并不需要关系部门的内容。
### PC 寄存器
PC 寄存器，页脚程序计数器，JVM支持多个线程同时运行，每个线程都有自己的程序计数器，倘若当前执行的是JVM的方法，则该寄存器中保存当前执行指令的地址；倘若执行的是native方法，则PC寄存器中为空。
### 堆
堆内存是JVM所有线程共享的部分，在虚拟机启动的时候就已经创建。所有的对象和数组都在堆上进行分配，这部门空间可通过GC进行回收，当申请不到空间时会抛出OutOfMemoryError。
### 方法区
方法区也是所有线程共享。主要用与存储类的信息、常量池、方法数据、方法代码等。方法区逻辑上数据堆的一部分，但是为了与堆进行区分，通常又叫“非堆”。

## PermGen（永久带）
绝大部门java程序员都应该见过“java.lang.OutOfMemoryError.PermGen space”这个异常。这里的“PermGen space”其实指的就是方法区。不过方法区和”PermGen space“又有着本质的区别。前者是JVM规范，后后者则是JVM规范的一种实现，并且只有HotSpot 才有 ”PremGen space“，而对于其他类型的虚拟机，如 JRockit（Oracle）、J9（IBM）并没有”PremGen space“。由于方法区主要存储类的相关信息，所以对于动态生成类的情况比较容易出现永久带的内存溢出。最典型的的长久就是，在jsp页面比较的情况，容易出现永久代内存溢出。

## Metaspace（元空间）
其实，溢出永久代的工作从JDK1.7就开始了，JDK1.7中，存储在永久代的部分数据就已经转移到java

Heap或者 Native Heap。但永久代仍存在于JDK1.7中，并没有完全移除，譬如符号引用（Symbols）转移到了native heap。字面量（interned string）转移到了java heap；类的静态变量转移到了java Heap。

元空间的本质就是永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于，元空间不在虚拟机，而是使用本地内存。因此，默认情况下，元空间的大小受本地内存的现在，可以通过以下参数来指定元空间的大小：

 - **-XX:MetaspaceSize**，初试空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整，如果释放了大量的空间，就适当的降低该值；如果释放了很少的空间，那么就在不超过MaxMetaspaceSize时，适当的提高该值。
 - **-XX:MaxMetaspaceSize**，最大空间，默认是没有限制。
除了上面两个指定大小的选项以外，还有两个与GC相关的属性:
 - **-XX:MinMetaspaceFreeRatio**，在GC之后，最小的Metaspace剩余空间容量的百分比，减少为分配空间所导致的垃圾收集
 - **-XX:MaxMetaspaceFreeRatio**，在GC之后，最大的Metaspace剩余空间容量的百分比，减少为释放空间所导致的垃圾收集

## 总结
**为什么JDK8中永久代向元空间转换，总结下以下几点原因**：
1. 字符串在永久代中，容易出现性能问题和内存溢出。
1. 类及方法和信息等比较难确定其大小，因此对于永久代的代销指定比较困难，太小容易出现永久代溢出，太大容易出现老年代溢出。
1. 永久代会为GC带来不必要的复杂度，并且回收效率偏低。
1. Oracle可能会将HotSpot 与JRockit 合二为一。
