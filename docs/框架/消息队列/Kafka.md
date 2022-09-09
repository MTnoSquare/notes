## 概念

1. **消息：Record**  指 Kafka 处理的主要对象。
   
2. **主题：Topic**  主题是承载消息的逻辑容器，在实际使用中多用来区分具体的业务。
   
3. **分区：Partition**  一个有序不变的消息序列。每个主题下可以有多个分区。
   
4. **消息位移：Offset**  表示分区中每条消息的位置信息，是一个单调递增且不变的值。
   
5. **副本：Replica**  Kafka 中同一条消息能够被拷贝到多个地方以提供数据冗余，这些地方就是所谓的副本。副本还分为领导者副本和追随者副本，各自有不同的角色划分。副本是在分区层级下的，即每个分区可配置多个副本实现高可用。

6.  **生产者：Producer**  向主题发布新消息的应用程序。
    
7.  **消费者：Consumer**  从主题订阅新消息的应用程序。

8.  **消费者位移：Consumer Offset**  表征消费者消费进度，每个消费者都有自己的消费者位移。
    
9.  **消费者组：Consumer Group**  多个消费者实例共同组成的一个组，同时消费多个分区以实现高吞吐。
    
10. **重平衡：Rebalance**  消费者组内某个消费者实例挂掉后，其他消费者实例自动重新分配订阅主题分区的过程。Rebalance 是 Kafka 消费者端实现高可用的重要手段。

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220906110832.png)

## 配置参数
//TODO
### Broker

**log.dirs**：这是非常重要的参数，指定了 Broker 需要使用的若干个文件目录路径,没有默认值,需要亲自指定.

## 分区策略

生产者实现 `org.apache.kafka.clients.producer.Partitioner`接口的`partition()`方法,并设置partitioner.class参数为你自己实现类的 Full Qualified Name

```java
//消息数据 : topic、key、keyBytes、value和valueBytes
//集群信息 : cluster
int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster);
```

1. **轮询策略(默认)**
 
Round-robin 策略，即顺序分配。比如一个主题下有 3 个分区，那么第一条消息被发送到分区 0，第二条被发送到分区 1，第三条被发送到分区 2，以此类推。当生产第 4 条消息时又会重新开始，即将其分配到分区 0

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220906132933.png)

2. **随机策略**

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220906133159.png)

3. **按key分配**

Kafka 允许为每条消息定义消息键，简称为 Key。这个 Key 的作用非常大，它可以是一个有着明确业务含义的字符串，比如客户代码、部门编号或是业务 ID 等；也可以用来表征消息元数据。

一旦消息被定义了 Key，那么你就可以保证同一个 Key 的所有消息都进入到相同的分区里面，由于每个分区下的消息处理都是有顺序的，故这个策略被称为按消息键保序策略，如下图所示。

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220906133315.png)

```java
List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
return Math.abs(key.hashCode()) % partitions.size();
```

## 消息不丢失配置

### 生产者

1. 生产者使用带有回调通知的发送API,不使用`producer.send(msg)`，而要使用 `producer.send(msg, callback)`

2. 设置 `acks = all` 表明所有副本 Broker 都要接收到消息，该消息才算是“已提交”(最高级别的"已提交"定义)

3. 设置` retries `为一个较大的值  当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了 retries > 0 的 Producer 能够自动重试消息发送，避免消息丢失。



### Broker

1. 设置 `unclean.leader.election.enable = false`。它控制的是哪些 Broker 有资格竞选分区的 Leader。如果一个 Broker 落后原先的 Leader 太多，那么它一旦成为新的 Leader，必然会造成消息的丢失。故一般都要将该参数设置成 `false`，即不允许这种情况的发生。  

2. 设置 `replication.factor >= 3`。这也是 Broker 端的参数。其实这里想表述的是，最好将消息多保存几份，毕竟目前防止消息丢失的主要机制就是冗余。  

3. 设置` min.insync.replicas > 1`。这依然是 Broker 端参数，控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。  

4. 确保` replication.factor > min.insync.replicas`。如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。推荐设置成 replication.factor = min.insync.replicas + 1。

### 消费者

1. 手动提交位移  `enable.auto.commit`设置为false

## 消息积压

1. 先修复 consumer 的问题，确保其恢复消费速度，然后将现有 consumer 都停掉。
2. 新建一个 topic，partition 是原来的 10 倍，临时建立好原先 10 倍的 queue 数量。
3. 然后写一个临时的分发数据的 consumer 程序，这个程序部署上去消费积压的数据，消费之后不做耗时的处理，直接均匀轮询写入临时建立好的 10 倍数量的 queue。
4. 接着临时征用 10 倍的机器来部署 consumer，每一批 consumer 消费一个临时 queue 的数据。这种做法相当于是临时将 queue 资源和 consumer 资源扩大 10 倍，以正常的 10 倍速度来消费数据。
5. 等快速消费完积压数据之后，得恢复原先部署的架构，重新用原先的 consumer 机器来消费消息。

## 高吞吐量

### 顺序写

传统磁盘io需要经过**寻道、旋转和数据传输**三个步骤。而Kafka采用追加写的方式,省去了前两个步骤

Kafka 中每个分区是一个有序的，不可变的消息序列，新的消息不断追加到 Partition 的末尾，在 Kafka 中 Partition 只是一个逻辑概念，Kafka 将 Partition 划分为多个 Segment，每个 Segment 对应一个物理文件，Kafka 对 segment 文件追加写，这就是顺序写文件。

### 零拷贝

`producer `生产消息到 `Broker` 时，`Broker` 会使用 `pwrite()` 系统调用【对应到 Java NIO 的 `FileChannel.write()` API 按偏移量写入数据，此时数据都会先写入`page cache`。`consumer` 消费消息时，`Broker `使用 `sendfile() `系统调用对应 `FileChannel.transferTo() API`，零拷贝地将数据从 `page cache` 传输到 `broker` 的 `Socket buffer`，再通过网络传输。

`page cache`中的数据会随着内核中 `flusher` 线程的调度以及对 `sync()/fsync()` 的调用写回到磁盘，就算进程崩溃，也不用担心数据丢失。另外，如果` consumer` 要消费的消息不在`page cache`里，才会去磁盘读取，并且会顺便预读出一些相邻的块放入 `page cache`，以方便下一次读取。

因此如果` Kafka producer `的生产速率与 `consumer` 的消费速率相差不大，那么就能几乎只靠对 `broker page cache` 的读写完成整个生产 - 消费过程，磁盘访问非常少。

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220906151336.png)

### 网络模型

Kafka实现了和Netty一样的Reactor模型,基于NIO,实现多路复用

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220906150632.png)

### 批量与压缩

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220906173408.png)

发送消息依次经过以下处理器：

* Serialize：键和值都根据传递的序列化器进行序列化。优秀的序列化方式可以提高网络传输的效率。
* Partition：决定将消息写入主题的哪个分区，默认情况下遵循 murmur2 算法。自定义分区程序也可以传递给生产者，以控制应将消息写入哪个分区。
* Compress：默认情况下，在 Kafka 生产者中不启用压缩.Compression 不仅可以更快地从生产者传输到代理，还可以在复制过程中进行更快的传输。压缩有助于提高吞吐量，降低延迟并提高磁盘利用率。
* Accumulate：Accumulate顾名思义，就是一个消息累计器。其内部为每个 Partition 维护一个Deque双端队列，队列保存将要发送的批次数据，Accumulate将数据累计到一定数量，或者在一定过期时间内，便将数据以批次的方式发送出去。记录被累积在主题每个分区的缓冲区中。根据生产者批次大小属性将记录分组。主题中的每个分区都有一个单独的累加器 / 缓冲区。
* Group Send：记录累积器中分区的批次按将它们发送到的代理分组。批处理中的记录基于 batch.size 和 linger.ms 属性发送到代理。记录由生产者根据两个条件发送。当达到定义的批次大小或达到定义的延迟时间时。

Producer、Broker 和 Consumer 使用相同的压缩算法，在 producer 向 Broker 写入数据，Consumer 向 Broker 读取数据时甚至可以不用解压缩，最终在 Consumer Poll 到消息时才解压，这样节省了大量的网络和磁盘开销。

### 分区并发

Kafka 的 Topic 可以分成多个 Partition，每个 Paritition 类似于一个队列，保证数据有序。同一个 Group 下的不同 Consumer 并发消费 Paritition，分区实际上是调优 Kafka 并行度的最小单元，因此，可以说，每增加一个 Paritition 就增加了一个消费并发。

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220906172819.png)

Kafka 消息是以 Topic 为单位进行归类，各个 Topic 之间是彼此独立的，互不影响。每个 Topic 又可以分为一个或多个分区。每个分区各自存在一个记录消息数据的日志文件。

Kafka 每个分区日志在物理上实际按大小被分成多个 Segment。

* segment file 组成：由 2 大部分组成，分别为 index file 和 data file，此 2 个文件一一对应，成对出现，后缀”.index”和“.log”分别表示为 segment 索引文件、数据文件。
* segment 文件命名规则：partion 全局的第一个 segment 从 0 开始，后续每个 segment 文件名为上一个 segment 文件最后一条消息的 offset 值。数值最大为 64 位 long 大小，19 位数字字符长度，没有数字用 0 填充。

>index 采用稀疏索引，这样每个 index 文件大小有限，Kafka 采用mmap的方式，直接将 index 文件映射到内存，这样对 index 的操作就不需要操作磁盘 IO。mmap的 Java 实现对应 MappedByteBuffer 。