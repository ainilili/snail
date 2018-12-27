## Java环境的安装
系统

### 安装Open JDK
```powershell
yum list | grep openjdk
yum install java-1.8.0-openjdk
```
### 安装Oracle JDK
首先去Oracle下载JDK [传送门](https://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html)，将之下载到完成之后通过传到Linux上，注意，作者下载的版本是``jdk1.8.0_181``。

之后```tar -zxf```解压：
```powershell
tar -zxf xxxxxx.tar.gz
```
以下是笔者最终存放jdk的物理路径：
```powershell
[nico@VM_0_17_centos jdk1.8.0_181]$ pwd
/usr/lib/java-1.8.0/jdk1.8.0_181
```
其中``jdk1.8.0_181``就是解压后的目录!

接下来配置环境变量，通过```vim /etc/profile```进入编辑操作，并增加以下配置
```powershell
export JAVA_HOME=/usr/lib/java-1.8.0/jdk1.8.0_181
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib/
export PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH
```
退出并保存，通过执行```source /etc/profile```立即生效

然后分别执行```java```和```javac```验证环境是否生效
