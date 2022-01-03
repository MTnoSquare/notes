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
```java

```