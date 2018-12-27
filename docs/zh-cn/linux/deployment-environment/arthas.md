## 一、Arthas简介
`Arthas` 是Alibaba开源的Java诊断工具，深受开发者喜爱。

当你遇到以下类似问题而束手无策时，`Arthas`可以帮助你解决：

0. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
0. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
0. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
0. 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
0. 是否有一个全局视角来查看系统的运行状况？
0. 有什么办法可以监控到JVM的实时运行状态？

`Arthas`采用命令行交互模式，同时提供丰富的 `Tab` 自动补全功能，进一步方便进行问题的定位和诊断。

使用说明可以看下官方的文档[https://alibaba.github.io/arthas/](https://alibaba.github.io/arthas/)

## 二、安装及使用
以下通过CentOS系统举例来安装Arthas
#### 1.安装Java环境
首先去Oracle下载JDK，可以通过```wget```或者```curl -O```来下载到Linux主机上。

最方便的就是下载一个tar.gz格式的压缩包，然后通过```tar -zxf```解压，以下是笔者最终存放jdk的物理路径
```powershell
[nico@VM_0_17_centos jdk1.8.0_181]$ pwd
/usr/lib/java-1.8.0/jdk1.8.0_181
```

接下来配置环境变量，通过```vim /etc/profile```进入编辑操作，并增加以下配置
```powershell
export JAVA_HOME=/usr/lib/java-1.8.0/jdk1.8.0_181
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib/
export PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH
```
退出并保存，通过执行```source /etc/profile```立即生效

然后分别执行```java```和```javac```验证环境是否生效
#### 2.安装Arthas
Arthas非常小巧，官方有一个可直接执行的sh脚本供我们使用，可以通过以下指令下载
```powershell
curl -L https://alibaba.github.io/arthas/install.sh | sh
```
启动Arthas
```powershell
./as.sh
```
#### 3.安装ElasticSearch
Arthas启动需要存在至少一个及以上的Java进程，这里我们为了方便测试，直接安装ElasticSearch。

和安装JDK的方式类似，我们去官方下载一个```tar.gz```的压缩包，然后解压
```powershell
[nico@VM_0_17_centos elasticsearch]$ ll
total 95612
drwxr-xr-x 9 nico root     4096 Sep 19 09:03 elasticsearch-6.4.0
-rw-r--r-- 1 nico root 97901357 Aug 23 23:21 elasticsearch-6.4.0.tar.gz
```
ElasticSearch的不允许**root**用户启动，所以笔者用的是**nico**账号，创建账号过程如下
```powershell
adduser nico
passwd nico
#输入密码
#将as.sh和elasticsearch的目录权限赋予nico账户
chown nico as.sh
chown -R nico elasticsearch
su nico
```
以上指令请分开到对应的目录执行，执行完毕之后我们进入elasticsearch目录下的bin目录中，启动elasticsearch
```powershell
./elasticsearch
```
#### 4.使用Arthas监控ElasticSearch
进入```as.sh```所在目录，启动Arthas
```powershell
./as.sh
```
```powershell
[nico@VM_0_17_centos arthas]$ ./as.sh
Arthas script version: 3.0.4
Found existing java process, please choose one and hit RETURN.
* [1]: 19670 org.elasticsearch.bootstrap.Elasticsearch
```
我们看到，当前只有ElasticSearch一个进程，输入1监控ElasticSearch
```powershell
[nico@VM_0_17_centos arthas]$ ./as.sh
Arthas script version: 3.0.4
Found existing java process, please choose one and hit RETURN.
* [1]: 19670 org.elasticsearch.bootstrap.Elasticsearch
1
Calculating attach execution time...
Attaching to 19670 using version 3.0.4...

real	0m0.227s
user	0m0.177s
sys	0m0.035s
Attach success.
Connecting to arthas server... current timestamp is 1537320967
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
  ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.
 /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'


wiki: https://alibaba.github.io/arthas
version: 3.0.4
pid: 19670
timestamp: 1537320967728

$
```
启动成功，Arthas提供一个shell的操作模式来供我们去监控Java进程，具体指令可以看下官方的[Wiki](https://alibaba.github.io/arthas/quick-start.html)
#### 5.安装过程可能遇到的问题
(1)遇到报错```java.security.AccessControlException: Access Denied ```

官方解决办法
```powershell
Add the permission in client.policy (for the application client), or in server.policy (for EJB/web modules) for the application that needs to set the property. By default, applications only have “read” permission for properties.

For example, to grant read/write permission for all the files in the codebase directory, add or append the following to client.policy or server.policy:

grant codeBase "file:/.../build/sparc_SunOS/sec/-" {
   permission java.util.PropertyPermission "*", "read,write";
 };
```
java.policy所在的目录为JDK所在目录下的相对目录```jre/lib/security```
```powershell
vim java.policy
```
尾部增加一行即可
```powershell
permission java.security.AllPermission;
```
