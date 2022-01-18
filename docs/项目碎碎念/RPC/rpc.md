## 如何通信
Socket和Netty
### Socket
基于BIO实现，字节流读取和输入。

服务端创建线程池对accpet队列上的连接进行处理。

客户端不会主动关闭socket连接。(频繁创建导致资源消耗)
### Netty
基于NIO实现

