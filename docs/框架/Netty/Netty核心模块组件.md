## 1.Bootstrap,ServerBootstrap

一个Netty应用通常由一个Bootstrap开始，主要作用是配置整个Netty程序，串联各个组件，Netty中Bootstrap是客户端程序启动引导类，ServerBootstrap是服务端启动引导类。

常见方法:
```java
public ServerBootstrap group(EventLoopGroup parentGroup,EventLoopGroup childGroup)//用于服务器端，设置两个EventLoop

public B group(EventLoopGroup group)//用于客户端，设置一个 EventLoop

public B channel(Class<? extends C> channelClass)//该方法用来设置一个服务器端的通道实现
public<T> B option(ChannelOption<T> option, T value)//用来给ServerChannel添加配置

public<T>ServerBootstrap childOption(ChannelOption<T> childOption, T value)//用来给接收到的通道添加配置
public ServerBootstrap childHandler(ChannelHandler childHandler)//该方法用来设置业务处理类（自定义的handler)

public ChannelFuture bind(int inetPort)//该方法用于服务器端，用来设置占用的端口号

public ChannelFuture connect(String inetHost, int inetPort)//该方法用于客户端，用来连接服务器端

```

## 2. Future,ChannelFuture
   
注册对IO操作的监听，当操作成功或者失败时会自动触发注册的监听事件

常见方法:
```java
Channel channel()//返回当前正在进行的IO操作通道
ChannelFuture sync()//等待异步操作执行完毕
```

## 3. Channel

Netty网络通信组件，用于执行异步的网络IO操作，调用立即返回一个`ChannelFuture`实例，不同协议有着不同的`Channel`类型与之对应

## 4. Selector

Netty基于Selector实现IO多路复用，通过Selector一个线程监听多个连接的Channel事件

## 5. ChannelHandler

`ChannelHandler`是一个接口，用来处理IO事件或拦截IO事件，将其转发到ChannelPipeline(业务处理链)中的下一个处理程序

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220111234505.png)

```java

```

## 6. ChannelPipeline
它是一系列Handler的集合，负责处理拦截`inbound`和`outbound`的事件和操作，相当于一条链(责任链模式)

每个`Channel`有且仅有一个`ChannelPipeline`与之对应

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220111234947.png)

## 7. ChannelOption

ChannelOption.SO_BACKLOG:对应TCP/IP协议中listen函数的backlog参数，初始化服务器可连接队列大小，服务端将暂时不能处理的客户端连接请求放在队列中等待处理

ChannelOption.SO_KEEPALIVE:保持连接活动状态

ChannelOption.SO_REUSEADDR:意味着地址可以复用:->某个进程占用了80端口,然后重启进程,原来的socket1处于TIME-WAIT状态,进程启动后,使用一个新的socket2,要占用80端口,如果这个时候不设置SO_REUSEADDR=true,那么启动的过程中会报端口已被占用的异常。
