# 概念
Netty基于主从Reactor线程模型  

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/IO%2Fnetty%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.jpg)  
## 说明
1. Netty将主从Reactor抽象为BossGroup线程池（接收客户端连接）和WorkGroup线程池（负责网络读写）
2. BossGroup和WorkGroup都是`NioEventLoopGroup`
   * Boss NioEventLoop执行步骤:  
        1. 轮询Accept事件
        2. 处理accept事件，与client建立连接，生成niosocketchannel，注册到某个worker NioEventLoop的selector
        3. 处理任务队列的任务（runAllTasks）
    * Worker NioEventLoop执行步骤:  
        1. 轮询read，write事件
        2. 处理read，write事件，使用pipeline在对应NioSocketChannel处理
        3. 处理任务队列的任务（runAllTasks）


# 代码示例
## 服务端
```java

public class MyServer {
    public static void main(String[] args) throws Exception {
        //创建两个线程组 boosGroup、workerGroup
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            //创建服务端的启动对象，设置参数
            ServerBootstrap bootstrap = new ServerBootstrap();
            //设置两个线程组boosGroup和workerGroup
            bootstrap.group(bossGroup, workerGroup)
                //设置服务端通道实现类型    
                .channel(NioServerSocketChannel.class)
                //设置线程队列得到连接个数    
                .option(ChannelOption.SO_BACKLOG, 128)
                //设置保持活动连接状态    
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                //使用匿名内部类的形式初始化通道对象    
                .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            //给pipeline管道设置处理器
                            socketChannel.pipeline().addLast(new MyServerHandler());
                        }
                    });//给workerGroup的EventLoop对应的管道设置处理器
            System.out.println("Netty服务端启动就绪");
            //绑定端口号，启动服务端
            ChannelFuture channelFuture = bootstrap.bind(6666).sync();
            //对关闭通道进行监听
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```
## 服务端处理器
```java

/**
 * 自定义的Handler需要继承Netty规定好的HandlerAdapter
 * 才能被Netty框架所关联，有点类似SpringMVC的适配器模式
 **/
public class MyServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    //ChannelHandlerContext ctx 是上下文对象，含有管道pipeline，通道channel
    //Object msg 客户端发送的数据
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //获取客户端发送过来的消息
        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println("收到客户端" + ctx.channel().remoteAddress() + "发送的消息：" + byteBuf.toString(CharsetUtil.UTF_8));
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //发送消息给客户端
        ctx.writeAndFlush(Unpooled.copiedBuffer("服务端已收到消息", CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //发生异常，关闭通道
        ctx.close();
    }
}
```
## 客户端
```java
public class MyClient {

    public static void main(String[] args) throws Exception {
        //客户端需要一个事件循环组
        NioEventLoopGroup eventExecutors = new NioEventLoopGroup();
        try {
            //创建bootstrap对象，配置参数
            Bootstrap bootstrap = new Bootstrap();
            //设置线程组
            bootstrap.group(eventExecutors)
                //设置客户端的通道实现类型    
                .channel(NioSocketChannel.class)
                //使用匿名内部类初始化通道
                .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            //添加客户端通道的处理器
                            ch.pipeline().addLast(new MyClientHandler());
                        }
                    });
            System.out.println("客户端准备就绪");
            //连接服务端
            ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 6666).sync();
            //对通道关闭进行监听
            channelFuture.channel().closeFuture().sync();
        } finally {
            //关闭线程组
            eventExecutors.shutdownGracefully();
        }
    }
}
```
## 客户端处理器
```java


public class MyClientHandler extends ChannelInboundHandlerAdapter {
    //当通道就绪时就会触发该方法
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        //发送消息到服务端
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello", CharsetUtil.UTF_8));
    }
    //当通道有读取事件时触发
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //接收服务端发送过来的消息
        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println("收到服务端" + ctx.channel().remoteAddress() + "的消息：" + byteBuf.toString(CharsetUtil.UTF_8));
    }
}  
```  
  

# 异步模型
## 概念  
Netty中的I/O操作是异步的，包括bind，write，connect等操作会返回一个ChannelFuture。调用者通过`Future-Listener`机制获得IO操作结果  
> Future-Listener机制:当Future刚刚创建处于非完成状态，调用者通过channelfuture来获取操作执行的状态，注册监听函数来执行完成后的操作。  
> 
> 常见方法：  
> isDone()      当前操作是否完成  
> isSuccess()   已完成的当前操作是否成功  
> getCause()    已完成的当前操作失败原因  
> isCancelled() 已完成的当前操作是否被取消  
> addListener() 注册监听器，当操作已完成，通知指定监听器；  

## Future说明
1. 表示异步执行结果，通过它提供的方法检查执行是否完成。 
2. ChannelFuture是一个接口 `public interface ChannelFuture extends Future<void>`,通过添加监听器监听事件。
```java
//绑定端口是异步操作
//绑定一个端口并且同步,生成了一个ChannelFuture 对象
        //启动服务器(并绑定端口)
        ChannelFuture cf = bootstrap.bind( 6668).sync();
        
        //给cf注册监听器，监控我们关心的事件
        cf.addListener(new ChannelFutureListener( {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                if (cf.isSuccessQ) {
                    System.out.println("监听端口6668 成功");
                } else {
                    System.out.println("监听端口6668 失败");
                }
            }
        });

```
