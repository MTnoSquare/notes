# Socket
## 概念
套接字(Socket)，就是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。一个套接字就是网络上进程通信的一端，是应用程序通过网络协议进行通信的接口，是应用程序与网络协议栈进行交互的接口.

操作系统中以五元组确定一个Socket[目标端口，目标IP地址，源端口，源IP地址，协议(Tcp/Udp)]
## 工作流程
基于TCP的客户端与服务端交互
* 服务端和客户端初始化 `socket`，得到文件描述符；
* 服务端调用` bind`，将绑定在 IP 地址和端口;服务端调用 `listen`，进行监听；
* 服务端调用 `accept`，等待客户端连接；
* 客户端调用 `connect`，向服务器端的地址和端口发起连接请求；
* 服务端 `accept` 返回用于传输的 `socket` 的文件描述符；
* 客户端调用 `write` 写入数据；服务端调用 `read` 读取数据；
* 客户端断开连接时，会调用 `close`，那么服务端 `read` 读取数据的时候，就会读取到了 `EOF`，待处理完数据后，服务端调用 `close`，表示连接关闭。
![](https://pic4.zhimg.com/80/v2-175313ef59cf6ab78336318305980f53_720w.jpg)
## 函数基本操作
[socket--socket()、bind()、listen()、connect()、accept()](https://www.cnblogs.com/straight/articles/7660889.html)

[linuxSocket编程](https://www.cnblogs.com/skynet/archive/2010/12/12/1903949.html)