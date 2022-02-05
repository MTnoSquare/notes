# BIO
同步并阻塞，服务器实现模式为一个连接一个线程。  
 
![alt bio](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/IO/bio.png)
### 适用场景
连接数小且固定的架构。
### 存在问题
当连接不做任何事情时会造成不必要的线程开销。  

# NIO
![alt nio ](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/IO%2Freactor%E6%A8%A1%E5%9E%8B.jpg)  
同步非阻塞，基于Reactor模式。服务器实现模式为一个线程处理多个请求（连接），由多路复用器轮询到连接有I/O请求进行处理。
### 适用场景
连接数多且连接较短（轻操作），如聊天系统，弹幕系统等。
# AIO
异步非阻塞，基于Proactor模式。它节省了NIO中select函数遍历事件通知队列的代价
### 适用场景
连接数多且连接较长

# Reactor
为解决传统阻塞I/O模型的缺点，基于**I/O复用模型**和**基于线程池复用线程资源**  
![alt Reactor](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/IO%2Freactor%E6%A8%A1%E5%9E%8B.jpg)
## 核心组成
* **Reactor**:在一个单独线程中运行，负责监听和分发事件，分发给处理程序对IO事件进行处理。（电话接线员）
* **Handlers**：处理实际事件（接线员通知的人）


## 模式分类
### 单Reactor单线程
![ ](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/IO%2F%E5%8D%95reactor%E5%8D%95%E7%BA%BF%E7%A8%8B.jpg)  
1. Reactor通过Select（IO多路复用）监听多个连接请求，通过dispatch进行分发  
2. 如果是建立连接请求，由Acceptor通过Accept处理，然后创建Handler处理连接完成后的业务
3. 如果不是建立连接，则调用对应的Handler来响应
4. Handler会完成Read->业务处理->Send的流程
#### 优缺点分析
**优点**:  
1. 简单，一个线程完成  
     
**缺点**:
1. 无法完全发挥多核CPU性能，导致性能瓶颈   
2. 线程意外终止或者进入死循环会导致整个模块不可用，不可靠  

**使用场景**  
客户端数量有限，业务处理快速。比如**Redis的业务处理**
### 单Reactor多线程
![ ](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/IO%2F%E5%8D%95Reactor%E5%A4%9A%E7%BA%BF%E7%A8%8B.jpg)  
1. 前面与单线程模型一致，但是这里的Handler只负责响应事件，不做业务处理，
2. work线程池分配独立线程完成真正的业务，并将结果返回给handler
3. handler收到响应后，通过send将结果返回给client  

**优点**:  
1. 发挥多核CPU的处理能力。
     
**缺点**:  
1. 多线程数据共享和访问比较复杂，reactor处理所有事件监听和响应在单线程运行，高并发场景容易出现性能瓶颈
 
### 主从Reactor多线程  
![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/IO/%E4%B8%BB%E4%BB%8EReactor%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.jpg)
1. 父线程处理建立连接请求，通过Acceptor处理并分配给子Reactor  
2. 子线程监听已经建立的连接的请求，分配给handler进行事件处理  
3. Reactor主线程可以对应多个Reactor子线程.  
   
**优点**:  
1. 父线程和子线程数据交互简单职责明确  
     
**缺点**:
1. 编程复杂度较高  
   
**使用场景**:  
Nginx，Memcached，Netty

## Reactor模式总结  
1. 响应快
2. 最大程度避免多线程及同步问题，避免多线程/进程的切换开销
3. 扩展性好
4. 复用性好


# Proactor
![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/IO%2Fproactor.jpg)
1. Procator Initiator负责创建Procator和Handler，并将Procator和Handler都通过Asynchronous operation processor注册到内核。
2. Asynchronous operation processor负责处理注册请求，并完成IO操作。完成IO操作后会通知procator。
3. procator根据不同的事件类型回调不同的handler进行业务处理。handler完成业务处理，handler也可以注册新的handler到内核进程。  
   
**缺点**：

1. 编程复杂性，由于异步操作流程的事件的初始化和事件完成在时间和空间上都是相互分离的，因此开发异步应用程序更加复杂。应用程序还可能因为反向的流控而变得更加难以Debug；
2. 内存使用，缓冲区在读或写操作的时间段内必须保持住，可能造成持续的不确定性，并且每个并发操作都要求有独立的缓存，相比Reactor模型，在Socket已经准备好读或写前，是不要求开辟缓存的；
3. 操作系统支持，Windows下通过IOCP实现了真正的异步 I/O，而在Linux系统下，Linux2.6才引入，并且异步I/O使用epoll实现的，所以还不完善。  
**因此在 Linux 下实现高并发网络编程都是以Reactor模型为主。**
