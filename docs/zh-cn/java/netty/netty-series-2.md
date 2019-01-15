## Channel
在Netty框架中，Channel是其中之一的核心概念，是Netty网络通信的主体，由它负责同对端进行网络通信、注册和数据操作等功能。本文我们来详细地分析 Netty 中的 Channel以及跟Channel相关的其他概念，包括``ChannelPipeline``、``ChannelHandlerContext``、``ChannelHandler``等。
## ChannelPipeline
ChannelPipeline持有I/O事件拦截器ChannelHandler的链表，由ChannelHandler对I/O事件进行拦截和处理，可以方便地通过新增和删除ChannelHandler来实现不同的业务逻辑定制，不需要对已有的ChannelHandler进行修改，能够实现对修改封闭和对扩展的支持
## ChannelHandlerContext
每个ChannelHandler被添加到ChannelPipeline后，都会创建一个ChannelHandlerContext并与之创建的ChannelHandler关联绑定。ChannelHandlerContext允许ChannelHandler与其他的ChannelHandler实现进行交互。ChannelHandlerContext不会改变添加到其中的ChannelHandler，因此它是安全的。
## ChannelHandler
ChannelHandler是netty的核心处理部分，我们通过继承ChannelHandler以及他的衍生类并重写对应的方法去处理对应的事件，简单来讲，我们如果想实现编解码以及消息处理部分，那必定离不开ChannelHandler。

netty中最常用的ChannelHandler的实现就是``ChannelInboundHandlerAdapter``和``ChannelOutboundHandlerAdapter``，其中``ChannelInboundHandlerAdapter``负责用来监听外部传来的事件，例如接收到消息或者是监听到一个新的连接：
```java
public class SimpleServerEchoHandler extends ChannelInboundHandlerAdapter{

	Logger logger = LoggerFactory.getLogger(SimpleServerEchoHandler.class);

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		logger.info("Server Received：{}", String.valueOf(msg));
		ctx.writeAndFlush(msg);
	}

}
```
而``ChannelOutboundHandlerAdapter``则是监听自己的一些事件，如``write``和``flush``等事件：
```java
public class SimpleServerWriteHandler extends ChannelOutboundHandlerAdapter{

	Logger logger = LoggerFactory.getLogger(SimpleServerWriteHandler.class);

	@Override
	public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
		logger.info("Excute This Method After Writed !!");
	}
}
```
在Server端触发写事件时，将会调用SimpleServerWriteHandler类的``write``方法。

另外一个最常用的就是``ChannelInitializer``，它负责提供初始化``channel``的简单入口，上一节中我们用它来初始化``client``和``server``的channel：
```java
public class SimpleClientChannelInitializer extends ChannelInitializer<SocketChannel>{

	Logger logger = LoggerFactory.getLogger(SimpleClientChannelInitializer.class);

	@Override
	protected void initChannel(SocketChannel ch) throws Exception {

		ch.pipeline()
		.addLast(new StringDecoder())
        .addLast(new StringEncoder())
        .addLast(new SimpleClientEchoHandler());

		logger.info("Client Connected Server Successful !!");
	}

}
```
