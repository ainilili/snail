## 写之前
这里我们继续使用第一节中的服务端的例子，对于客户端，我们简单改造一下，使之可以向服务端发送一些我们输入的信息：
```java
public class SimpleClient {

	public static void main(String[] args) throws InterruptedException, IOException {
		EventLoopGroup group = new NioEventLoopGroup();
		Scanner in = null;
		try {
			Bootstrap bootstrap = new Bootstrap()
					.group(group)
					.channel(NioSocketChannel.class)
					.handler(new SimpleClientChannelInitializer());

			//这一步指定连接服务端的地址和端口并连接
			Channel channel = bootstrap.connect("127.0.0.1", 8080).sync().channel();

      //这样我们的客户端就可以实时的向服务端发送自定义消息了
			in = new Scanner(System.in);
			while(in.hasNext()){
				channel.writeAndFlush(in.nextLine());
			}

			channel.closeFuture().sync();
		} finally {
			group.shutdownGracefully().sync();
			if(in != null) in.close();
		}
	}

}
```
之后，我们只需要将初始化Channel、编解码、数据处理这三个部分完成即可。
## 第一步：初始化Channel
对于Channel初始化的例子在第二节中也有展露，它可以通过继承``ChannelInitializer``并重写``initChannel``方法来实现。

客户端的``ChannelInitializer``：
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
服务端的``ChannelInitializer``：
```java
public class SimpleServerChannelInitializer extends ChannelInitializer<SocketChannel>{

	@Override
	protected void initChannel(SocketChannel ch) throws Exception {
		ch.pipeline()
		.addLast(new StringDecoder())
        .addLast(new StringEncoder())
        .addLast(new SimpleServerEchoHandler());
	}

}
```
## 第二步：编解码
netty提供了许多默认的编解码器，当然我们也可以通过集成``ByteToMessageDecoder``和``MessageToByteEncoder``等可来自定义编解码，这里我们使用``StringDecoder``和``StringEncoder``来作为默认的编解码器。
## 第三步：数据处理
实现一个简单的数据处理器只需要继承``ChannelInboundHandlerAdapter``类即可，因为它可以监听到外来的事件，对于客户端，我们也可以使用``SimpleChannelInboundHandler``来代替，原因如下：
> SimpleChannelInboundHandler在接收到数据后会自动release掉数据占用的Bytebuffer资源(自动调用Bytebuffer.release())

简单来说，客户端在接收到消息之后很少会向其他端转发，所以可以直接释放``Bytebuffer``，而服务端常常需要响应客户端，如果在读取完毕之后还没有响应完毕，这是关闭``ByteBuffer``将会引起服务端的IO异常。

客户端数据处理器（只读）：
```java
public class SimpleClientEchoHandler extends SimpleChannelInboundHandler<String>{

	Logger logger = LoggerFactory.getLogger(SimpleClientEchoHandler.class);

	@Override
	protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
		logger.info("Client Received：{}", String.valueOf(msg));
	}

}
```

服务端数据处理器（读和写）：
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
## 测试
服务端：
```powershell
[INFO ] 00:46:19.119 [main] o.nico.middleware.netty.SimpleServer - Server Initialized On Port 8080 !!
[INFO ] 00:46:30.874 [nioEventLoopGroup-3-1] o.n.m.netty.SimpleServerEchoHandler - Server Received：hello
[INFO ] 00:47:10.091 [nioEventLoopGroup-3-1] o.n.m.netty.SimpleServerEchoHandler - Server Received：netty
```
客户端：
```powershell
[INFO ] 00:46:27.924 [nioEventLoopGroup-2-1] o.n.m.n.SimpleClientChannelInitializer - Client Connected Server Successful !!
hello
[INFO ] 00:46:30.881 [nioEventLoopGroup-2-1] o.n.m.netty.SimpleClientEchoHandler - Client Received：hello
netty
[INFO ] 00:47:10.092 [nioEventLoopGroup-2-1] o.n.m.netty.SimpleClientEchoHandler - Client Received：netty
```
