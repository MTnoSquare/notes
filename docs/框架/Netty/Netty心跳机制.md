## 心跳机制

* 心跳是在TCP长连接中，客户端和服务端定时向对方发送数据包通知对方自己还在线，保证连接的有效性的一种机制
* 在服务器和客户端之间一定时间内没有数据交互时, 即处于 idle 状态时, 客户端或服务器会发送一个特殊的数据包给对方, 当接收方收到这个数据报文后, 也立即发送一个特殊的数据报文, 回应发送方, 此即一个 PING-PONG 交互. 自然地, 当某一端收到心跳消息后, 就知道了对方仍然在线, 这就确保 TCP 连接的有效性.

**心跳实现:**

* 使用TCP协议层的Keeplive机制，但是该机制默认的心跳时间是2小时，依赖操作系统实现不够灵活；

* 应用层实现自定义心跳机制，比如Netty提供了`IdleStateHandler,ReadTimeoutHandler,WriteTimeoutHandler`三个handler检测连接有效性

## Netty IdleStateHandler心跳机制实例
在NettyClient注册`Handler.addLast(new IdleStateHandler(0, 5, 0, TimeUnit.SECONDS))`，即设定每5秒进行一次写检测，如果5秒内write()方法未被调用则触发一次`userEventTrigger()`方法，该方法在Handler类中实现：
```java

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if(evt instanceof IdleStateEvent){
            IdleState state = ((IdleStateEvent) evt).state();
            if(state == IdleState.WRITER_IDLE){
                logger.info("发送心跳包[{}]", ctx.channel().remoteAddress());
                Channel channel = ChannelProvider.get((InetSocketAddress) ctx.channel().remoteAddress(),
                        CommonSerializer.getByCode(CommonSerializer.DEFAULT_SERIALIZER));
                RpcRequest rpcRequest = new RpcRequest();
                rpcRequest.setHeartBeat(true);
                //设置一个Listener监测服务端是否接收到心跳包，如果接收到就代表对方在线，不用关闭Channel
                channel.writeAndFlush(rpcRequest).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
            }
        }else {
            super.userEventTriggered(ctx, evt);
        }
    }
```
在NettyServer类中向管道注册`Handler.addLast(new IdleStateHandler(30, 0, 0, TimeUnit.SECONDS))`，即设定IdleStateHandler心跳检测每30秒进行一次读检测，如果30秒内ChannelRead()方法未被调用则触发一次userEventTrigger()方法：
服务端实现：
```java
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if(evt instanceof IdleStateEvent){
            IdleState state = ((IdleStateEvent) evt).state();
            if(state == IdleState.READER_IDLE){
                logger.info("长时间未收到心跳包，断开连接……");
                ctx.close();
            }
        }else {
            super.userEventTriggered(ctx, evt);
        }
    }
```
## IdleStateHandler原理

1. 当服务器和客户端没有任何读写交互时，并超过了给定时间，则会触发用户handler的userEventTriggered方法。用户可以在这个方法中尝试向对方发送信息，如果发送失败，则关闭连接 
2. IdleStateHandler实现基于EventLoop的定时任务，每次读写都会记录一个值，在定时任务运行时，通过计算当前时间和设置时间和上次事件发生时间的结果，来判断是否空闲
3. Netty通过IdleStateHandler实现最常见的心跳机制不**是一种双向心跳的PING-PONG模式，而是客户端发送心跳数据包，服务端接收心跳但不回复，因为如果服务端同时有上千个连接，心跳的回复需要消耗大量网络资源**
4. 如果服务端一段时间内没有收到客户端的心跳数据包则认为客户端已经下线，将通道关闭避免资源的浪费；在这种心跳模式下服务端可以感知客户端的存活情况，无论是宕机的正常下线还是网络问题的非正常下线，服务端都能感知到，而客户端不能感知到服务端的非正常下线
## 参考链接
https://blog.csdn.net/u013967175/article/details/78591810