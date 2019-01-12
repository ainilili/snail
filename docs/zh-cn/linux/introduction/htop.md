## 简介
如何在Linux下查看计算机的CPU或者内存占用情况？很多同学第一时间想到的是使用``top``指令查看，但是其可视化及可操作性实在是太弱了，于是，``htop``诞生了！以下抄自百度百科：0
> ``htop`` 是一个 Linux 下的交互式的进程浏览器，可以用来替换Linux下的top命令！

可能上述的介绍不太会让人感觉到眼前一亮，那么来张图体验一下：

![](https://github.com/ainilili/snail/blob/master/images/htop-1-1.jpg?raw=true)

上图是博主的VPS下资源使用情况的快照，大家有没有发现界面很高大上，这就是``htop``。它提供可操作性化的界面，更加直观的数据图表，使得我们更容易的去掌控计算机资源！
## 安装
centos系统请使用：
```
sudo yum install htop
```
ubuntu系统请使用：
```
apt-get install htop
```
## 使用
直接在命令行下输入``htop``即可打开操作界面。在最顶部分别为计算器的``CPU``、``内存``的使用情况及计算机进程数：

![](https://github.com/ainilili/snail/blob/master/images/htop-1-2.jpg?raw=true)

中间部分则为各个进程信息：

![](https://github.com/ainilili/snail/blob/master/images/htop-1-3.jpg?raw=true)

可以通过点击顶部选项进行排序，或者通过上下键进行选择对应的进程！

最下面的部分是一些设置和操作：

![](https://github.com/ainilili/snail/blob/master/images/htop-1-4.jpg?raw=true)

我们可以设置过滤查找目标进程或者杀掉选中进程，当然还有很多功能，这里留给大家去主动挖掘！

## 总结
很实用的工具！
