## 写在前面
近期准备补一下数据结构，尤其是关于Tree系列的，其中，二叉树（Binary Tree）可以算是最简单的之一，所以打算从之入手，将各种Tree的结构和操作都进一步了解一遍，以来充实自己的闲余时间！

本文主要围绕二叉树中最简单的实现：**二叉搜索树**。
## 介绍
二叉搜索树（Binary Search Tree），也叫作**二叉排序树**（Binary Sort Tree），后文中统一简称为BST，顾名思义，它的每个结点的叶子数量不得超过两个：

![二叉树结构图](https://github.com/ainilili/snail/blob/master/images/binary-sort-tree-1-1.jpg?raw=true)

BST插入都是按照一定的规则顺序的，例如图中的结点，F的左边一定比F的值小，F右边一定比F的值大，当然这些都是可以自定义的，不过到头来，总是要有一定的规则进行存储，这样才会方便我们用来检索需要的内容。

BST的基本操作相对于平衡树或者红黑树来讲算是非常简单的，BST并不在乎整个树的平衡性，仅仅保证了结点的有序规则不被打乱，这种问题将会导致大量结点下的查询性能下降，因此便引出了更高级的树种（之后也将会进一步学习）。因此，实现一个BST并不难，我们可以通过Java去实现一个简单的BST来加深对二叉树的了解！
## 实现
首先是实现思路，我们主要实现插入、查询和删除的功能，其中查询是最简单的，因为不需要变动整个树的结构，删除是较复杂的，接下来将会一一解析其实现思路。
### 插入
被插入结点与root结点做比较，如果大于，则取root的右结点继续比较，反之则取左结点继续比较，如果相等，则更新当前结点的值。
```java
public Object insert(int index, Object value, Node cur) {
		while(true) {
			if(cur.index == index) {
				//命中则更新
				cur.value = value;
				break;
			}else {
				if(index < cur.index) {
					if(null == cur.left) {
						//无命中则添加
						cur.left = new Node(index, value, cur);
						cur.left.isLeft = true;
						size ++;
						break;
					}else {
						//指向左结点
						cur = cur.left;
					}
				}else{
					if(null == cur.right) {
						//无命中则添加
						cur.right = new Node(index, value, cur);
						size ++;
						break;
					}else {
						//指向右结点
						cur = cur.right;
					}
				}
			}
		}
		return value;
	}
```
这就是一个简单的深度查找过程，如果使用递归可能更好理解一些，但是为了防止StackOverFlow，并没用采用之。

代码中从``cur``这个结点开始，首先判断是否命中目标，也就是``cur.index == index``如果为true，则代表命中，命中的话则直接更新。

如果没有命中，则要向``cur``结点的分支方向搜索，如果到了叶子结点还未命中，则要进行插入操作（直接加结点），否则就修改``cur``指针，继续重复当前动作。
### 查询
查询类似于插入操作，不同的是查询更为简单：
```java
public Node find(int index, Node cur) {
	while(cur != null && cur.index != index) {
		cur = index < cur.index ? cur.left : cur.right;
	}
	return cur;
}
```
深度搜索即可！
### 删除
BST的删除是比较复杂的，不像插入那般，只需要操作一个结点的``left``或者``right``变量引用就可以了，它涉及着多个结点的变化：
```java
@Override
public Object remove(int index) {
	//首先寻找该结点
	Node target = find(index, root);
	if(target == null) {
		//该结点为空则直接返回null
		return null;
	}else {
		//优先考虑将被移除结点的左结点连接到右结点的左分支
		Node n = target.right != null ? target.right : target.left;

		if(target.left != null && target.right != null) {
			Node leftLoopStarter = n;
			//寻找目标结点的右结点下的最左结点
			while(leftLoopStarter.left != null) {
				leftLoopStarter = leftLoopStarter.left;
			}
			leftLoopStarter.setLeft(target.left);
		}

		if(target.parent == null) {
			//只剩下root结点不允许删除
			if(target.left == null && target.right == null) {
				throw new RuntimeException("再删就没了！");
			}else {
				//当删除root结点时，优先考虑将它的右结点作为新的root结点
				root = n;
				n.parent = null;
			}
		}else {
			//移除响应的结点
			if(target.isLeft) {
				target.parent.setLeft(n);
			}else {
				target.parent.setRight(n);
			}
		}
		size --;
		return target.value;
	}
}
```
上述代码不足表达整个过程，我们以具体的demo来演示删除过程，假如初始的树形状如下：
```java
       8               
   4       12       
 2   6   10   16   
1 3 5 7 # 11 14 17
```
移除结点1，叶子结点，直接移除：
```java
       8               
   4       12       
 2   6   10   16   
# 3 5 7 # 11 14 17
```
移除结点2，用右结点代替之：
```java
       8               
   4       12       
 3   6   10   16   
# # 5 7 # 11 14 17
```
移除结点4，其左结点放于右结点最左结点之后，并用右结点代替之：
```java
       8               
   6       12       
 5   7   10   16   
3 # # # # 11 14 17
```
移除结点16，同上：
```java
       8               
   6       12       
 5   7   10   17   
3 # # # # 11 14 #
```
移除结点8，同上：
```java
               12                               
       10              17               
   6       11      14      #       
 5   7   #   #   #   #   #   #   
3 # # # # # # # # # # # # # # #
```
### 格式化
在调试树的正确性时，无法避免要去查看当前树结点的状态，所以将树格式化并显示出来很有必要，再着手之前，我们首先要**找规律**：
```java
               a 			          //4-15  首距2^4-1	无结点间隔
       b               c 			  //3-7   首距2^3-1	结点间隔15
   d       e       f       g 		  //2-3   首距2^2-1	结点间隔7
 h   i   g   k   l   m   n   o 		//1-1   首距2^1-1	结点间隔3
a a a a a a a a a a a a a a a a 	   //0-0   首距2^0-1	结点间隔1

设高度为n
求出首距的公式：2^n-1
求出两结点间隔的公式：2^(n+1)-1
```
由上引出，二叉树的格式化还是很有规律的，我们便可以依据以上公式去写一个算法来格式化输出BST：
```java
@Override
public String toString() {
	//保存树形
	StringBuilder builder = new StringBuilder();
	//按层序保留结点
	List<Object> cache = new ArrayList<>(50);
	//使用栈对树做层序遍历
	Queue<Node> queue = new LinkedBlockingQueue<Node>();
	queue.add(root);

	int depth = 0; //临时深度
	int maxDepth = getMaxDepth(); //最大深度
	while(! queue.isEmpty()) {
		Node cur = queue.poll();
		if(cur.left != null) {
			queue.add(cur.left);
		}else if(cur.depth() < maxDepth){
			//填补空缺
			queue.add(new Node(0, '#', cur));
		}
		if(cur.right != null) {
			queue.add(cur.right);
		}else if(cur.depth() < maxDepth){
			//填补空缺
			queue.add(new Node(0, '#', cur));
		}
		if(depth != cur.depth()) {
			//深度切换，将高度保存
			depth = cur.depth();
			cache.add(depth);
		}
		//加入该结点
		cache.add(cur);
	}
	//将保留结点渲染为树形
	for(int index = 0; index < cache.size(); index ++) {
		Object o = cache.get(index);
		if(o instanceof Integer) {
			builder.append(System.lineSeparator());
			builder.append(getOffset(maxDepth - (Integer)o));
		}else {
			if(index == 0) {
				builder.append(getOffset(maxDepth));
			}
			builder.append(((Node)o).value);
			builder.append(getOffset(maxDepth - ((Node)o).depth() + 1));
		}
	}
	return builder.toString();
}

/**
 * @param height 当前结点高度 = 最大深度 - 当前结点深度
 * @return 首距或者两结点的间隔距离
 */
public String getOffset(double height) {
	StringBuilder builder = new StringBuilder();
	int count = (int) Math.pow(2d, height) - 1;
	while(count -- > 0) {
		builder.append(" ");
	}
	return builder.toString();
}

/**
 * @return 当前树的最大深度
 */
public int getMaxDepth() {
	Queue<Node> queue = new LinkedBlockingQueue<Node>();
	queue.add(root);
	int depth = 0;
	while(! queue.isEmpty()) {
		Node cur = queue.poll();
		if(cur.left != null) queue.add(cur.left);
		if(cur.right != null) queue.add(cur.right);
		if(depth != cur.depth()) depth = cur.depth();
	}
	return depth;
}
```
## 总结
通过BST引导入门是一个很不错的选择！

源码地址：[传送门](https://github.com/ainilili/java-study-demo/blob/master/src/main/java/org/nico/data/structure/tree/BinarySortTree.java)
