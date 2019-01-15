## 初识
netty是一款优秀的网络编程框架，将复杂的网络编程封装成优雅简单的API，让开发者轻松使用和部署。

我们所熟知的网络编程分为客户端和服务端，服务端负责处理客户端的请求并向客户端进行分发，客户端则是面向用户，它负责向用户显示操作面板（部分领域没有）并向服务端发送请求，一个服务端可能会有多个客户端链接，而一个客户端只能链接一个服务端。

更复杂一点的网络编程又会分为NIO（New I/O）、BIO（Block I/O)和AIO（Asynchronous I/O），netty的优势在于NIO领域，很多知名的开源目前的最新版本都会嵌入netty作为网络内核，这就间接的说明了我们确实非常有必要去学习一下netty！

## 使用netty开发一个服务端
下文中使用的netty的版本为``4.1.29``，使用maven可以轻松依赖：
```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.29.Final</version>
</dependency>
```
使用netty的api开发一个服务端非常的简单：
```java
public class SimpleServer {

	public static void main(String[] args) throws InterruptedException {

		EventLoopGroup parentGroup = new NioEventLoopGroup();
		EventLoopGroup childGroup = new NioEventLoopGroup();
		try {
			ServerBootstrap bootstrap = new ServerBootstrap()
			.group(parentGroup, childGroup)
			.channel(NioServerSocketChannel.class) //指定启动一个NIO类型的服务器
			.localAddress(new InetSocketAddress(8080)) //指定端口
			.childHandler(new DefaultChannelInitializer()); //指定一个ChannelHandler类型的处理器，在服务初始化之后调用处理器中的方法

			ChannelFuture f = bootstrap.bind().sync();

			f.channel().closeFuture().sync(); //阻塞直到服务关闭
		} finally {
			parentGroup.shutdownGracefully();
			childGroup.shutdownGracefully();
		}

	}
}
```
## 使用netty开发一个客户端
netty的客户端和服务端构建代码并没有太大的差异：
```java
public class SimpleClient {

	public static void main(String[] args) throws InterruptedException, IOException {
		EventLoopGroup group = new NioEventLoopGroup();
		try {
			Bootstrap bootstrap = new Bootstrap()
					.group(group)
					.channel(NioSocketChannel.class)
					.handler(new DefaultChannelInitializer());
      //这一步指定连接服务端的地址和端口并连接
			Channel channel = bootstrap.connect("127.0.0.1", 8080).sync().channel();
			channel.closeFuture().sync();
		} finally {
			group.shutdownGracefully().sync();
		}
	}

}
```
