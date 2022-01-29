## 传输协议
```
+---------------+---------------+-----------------+-------------+
|  Magic Number |  Package Type | Serializer Type | Data Length |
|    4 bytes    |    4 bytes    |     4 bytes     |   4 bytes   |
+---------------+---------------+-----------------+-------------+
|                          Data Bytes                           |
|                   Length: ${Data Length}                      |
+---------------------------------------------------------------+
```
## 如何通信
Socket和Netty
### Socket
基于BIO实现，字节流读取和输入。

服务端创建线程池对accpet队列上的连接进行处理。

客户端不会主动关闭socket连接。(频繁创建导致资源消耗)
### Netty
基于NIO实现

## 服务调用细节

## 服务器服务的注册与发现
`ServiceProvider`负责保存和提供服务实例对象

`ServiceRegistry`负责注册服务到Nacos中去

对于服务实现类，使用`@Service`注解

在抽象类实现扫描注解方法`scanServices()`，核心是利用反射，利用Service注解判断该类是否为服务类，将其创建实例存放在`ServiceProvider`实现类中，用`ConcurrentHashMap`进行存储，**同时注册到Nacos中**。

由于一个服务可能实现了多个接口，所以不同的接口该服务都要进行注册

## 服务的注销

单例模式创建钩子

钩子里调用注销服务的方法
```
Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            NacosUtil.clearRegistry();
            //关闭所有线程池
            ThreadPoolFactory.shutDownAll();
        }));
```

该方法用来在jvm中增加一个关闭的钩子。**当程序正常退出,系统调用 System.exit方法或虚拟机被关闭时才会执行**添加的shutdownHook线程。其中shutdownHook是一个已初始化但并不有启动的线 程，当jvm关闭的时候，会执行系统中已经设置的所有通过方法addShutdownHook添加的钩子，当系统执行完这些钩子后，jvm才会关闭。所以 可通过这些钩子在jvm关闭的时候进行内存清理、资源回收等工作。


