# Redis
## 概念
Redis是一个key-value的内存数据库，每秒可以处理超过10万次读写操作。

**Redis速度快的原因**
1. 内存存储
2. 单线程实现：Redis6.0后引入了多线程，主要是处理网络IO的请求，命令等执行仍是单线程。
3. IO多路复用
4. 优化的数据结构
5. Redis自己构建了VM(虚拟内存)机制

## 基本的数据结构
1. **String**
   
   String的数据结构为简单动态字符串(Simple Dynamic String,缩写SDS)。是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配.

**常用命令**：
   * SET key value:设置指定key的值
   * GET key：获取指定key的值
   * INCR/DECR key：将key中存储的数字值增/减1

**应用场景**：
   * 缓存
   * 计数器
   * 共享用户Session 

2. **Hash**
   
   Redis hash 是一个 string 类型的 field（字段） 和 value（值） 的映射表，hash 特别适合用于存储对象。Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

**常用命令**：
   * HDEL key field1[field2]:删除一个或多个哈希表字段
   * HEXISTS key field:查看哈希表key中，指定的字段是否存在
   * HGET key field：获取存储在哈希表中指定字段的值。 

3. **List**
   
   Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

**常用命令**：
   *  lpush/rpush  <key><value1><value2><value3> .... 从左边/右边插入一个或多个值。
   *  lpop/rpop  <key>从左边/右边吐出一个值。值在键在，值光键亡。
   *  rpoplpush  <key1><key2>从<key1>列表右边吐出一个值，插到<key2>列表左边。

4. **Set**
   
   Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以自动排重的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

    Redis的Set是string类型的无序集合。它底层其实是一个value为null的hash表，所以添加，删除，查找的复杂度都是O(1)。

**常用命令**：
   *   sadd <key><value1><value2> ：将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略
   *   spop <key>：随机从该集合中吐出一个值。
   *   sinter <key1><key2>：返回两个集合的交集元素。
   * sunion <key1><key2>：返回两个集合的并集元素。
   * sdiff <key1><key2>：返回两个集合的差集元素(key1中的，不包含key2中的)

**应用场景**：
   *  共同好友，共同关注
5. **Zset**

    Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。不同之处是有序集合的每个成员都关联了一个评分（score）,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复的 。因为元素是有序的, 可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

    zset底层使用了两个数据结构：
   1. hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。
   2. 跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

>关于跳跃表详细，请看[跳表](/数据结构/跳表.md)

**常用命令**：

   * zadd  <key><score1><value1><score2><value2>：将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
   * zrange <key><start><stop>  [WITHSCORES] ：返回有序集 key 中，下标在<start><stop>之间的元素
   * zrangebyscore key minmax [withscores] [limit offset count]：
返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 


**应用场景**：
   * 排行榜
   * 带权重的队列

6. **Bitmap**：

    位图，看作一个以位为单位的数组，每个单元只能存1或0，可以用它统计某用户是否存在等，节约内存

7. **Hyperloglog**：

    统计基数，每个键固定12KB内存，可以用它统计网页UV。它的统计规则基于概率完成，有一定误差

8. **Geospatial**
   
   存储地理位置信息。

## Redis过期策略和淘汰机制
### 过期策略
**定期删除+惰性删除**

**定期删除**：

**redis默认是每隔100ms就随机抽取一些设置了过期时间的key，检查是否过期，如果过期就删除。**

假设redis里放了10W个key，都设置了过期时间，你每隔几百毫秒就检查全部的key，那redis很有可能就挂了，CPU负载会很高，都消耗在检查过期的key上。注意，这里不是每隔100ms就遍历所有设置过期时间的key，那样就是一场性能灾难。实际上redis是每隔100ms就随机抽取一些key来检查和删除的。

**缺点**：定期删除可能会导致很多过期的key到了时间并没有被删除掉。这时应用惰性删除。

**惰性删除**：

获取某个key的时候，redis会检查这个key，如果设置了过期时间并且已经过期了，此时就会删除，不会给你返回任何东西。

**缺点**：如果定期删除漏掉了很多过期的key，也没及时去查，也就没走惰性删除。此时依旧有可能大量过期的key堆积在内存里，导致内存耗尽。

### 内存淘汰机制
为解决过期策略遗漏删除key导致内存耗尽

* **allkeys-lru**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key，这个是最常用的。
* **allkeys-random**：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。
* **volatile-lru**：当内存不足以容纳新写入数据时，在**设置了过期时间**的键空间中，移除最近最少使用的key。
* **volatile-random**：当内存不足以容纳新写入数据时，在**设置了过期时间**的键空间中，随机移除某个key。
* **volatile-ttl**：当内存不足以容纳新写入数据时，在**设置了过期时间**的键空间中，有更早过期时间的key优先移除。


## 事务
**概念**：

Redis事务并不具有原子性，中间某条指令失败不会导致回滚，其后的命令仍然会被执行

Redis事务中所有命令都会序列化按顺序执行，不被其他客户端发送来的命令请求打断。

**事务相关命令**

1. `multi`开启事务
2. `exec`执行事务块内的命令
3. `discard`取消事务
4. `watch`监视key，如果事务执行前key被改动，事务将打断。

## 持久化
Redis提供了RDB(Redis Database)和AOF(Append Only File)两种持久化机制

### RDB
RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘。也是默认的持久化方式，这种方式是就是将内存中数据以快照的方式写入到二进制文件中,默认的文件名为` dump.rdb`。

#### 触发方式
对于RDB来说，提供了三种机制：**save、bgsave、自动化**。


1. save 触发方式

    时间复杂度： O(N)， N 为要保存到数据库中的 key 的数量。

    SAVE 命令执行一个同步保存操作，将当前 Redis 实例的所有数据快照 (snapshot) 以 RDB 文件的形式保存到硬盘。

    **该命令会阻塞当前Redis服务器，执行save命令期间，Redis不能处理其他命令，直到RDB过程完成为止。**


2. bgsave

    与save不同，该命令执行后，Redis会fork出一个新的子进程，负责将数据保存到磁盘然后退出，原来的Redis父进程继续处理客户端请求。(fork用到了写时复制技术，copy-on-write)，阻塞只发生在fork阶段，一般时间很短。基本上 Redis 内部所有的RDB操作都是采用 bgsave 命令。


3. 自动化触发方式
   
在配置文件中配置，如：

   * **save**：这里是用来配置触发 Redis的 RDB 持久化条件，也就是什么时候将内存中的数据保存到硬盘。比如 save m n。表示 m 秒内数据集存在 n 次修改时，自动触发 bgsave。

    比如说我们在配置文件中配置：`save  60 5`只要在 60s 内修改了 5 次 key，就会触发持久化操作
不需要持久化，那么你可以注释掉所有的 save 行来停用保存功能。

   * **stop-writes-on-bgsave-error**：默认值为 yes。当启用了RDB且最后一次后台保存数据失败，Redis是否停止接收数据。这会让用户意识到数据没有正确持久化到磁盘上，否则没有人会注意到灾难（disaster）发生了。如果Redis重启了，那么又可以重新开始接收数据了


#### RDB优缺点

**优点**：

1. RDB文件紧凑，全量备份，非常适合用于进行备份和灾难恢复。
2. 生成RDB文件的时候，redis主进程会fork()一个子进程来处理所有保存工作，主进程不需要进行任何磁盘IO操作。
3. RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。 

**缺点**：
1. 需要一定的时间间隔进程操作！如果 redis 意外宕机了，这个最后一次修改数据就没有了
2. fork 进程的时候，会占用一定的内存空间

### AOF
以日志的形式来记录每个**写操作**（增量保存），将Redis执行过的所有写指令记录下来(读操作不记录)， **只许追加文件但不可以改写文件**，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大
#### 触发方式
1. 每次修改同步 `always`：同步持久化，每次更新操作后调用 fsync() 将数据写到磁盘，性能较差但数据完整性比较好
2. 每秒同步 `everysec`：异步操作，每秒记录。可能会丢失这1s的数据
3. 不同步 `no`：从不同步，等操作系统进行数据缓存同步到磁盘

配置参数： `appendfsync everysec`
### RDB OR AOF?
一般结合使用

||RDB|AOF|
|:-:|:-:|:-:|
|持久化方式|定时对整个内存快照|记录每一次执行的命令|
|数据完整性|不完整，两次备份间会丢失|相对完整，取决于刷盘策略|
|文件大小|会有压缩，体积小|记录命令，体积大|
|宕机恢复速度|快|慢|
|数据恢复优先级|低，因为完整性不如AOF|高，因此完整性高|
|系统资源占用|高，大量CPU和内存|低，磁盘IO资源，但AOF重写时会占用大量CPU和内存资源|
|使用场景|追求速度，容忍数分钟的数据丢失|对数据安全性要求高|



## 主从复制

**概念**： 主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master/leader)，后者称为从节点(slave/follower)

   * 数据的复制是单向的，只能由主节点到从节点。
   * Master以写为主，Slave 以读为主。
   * 默认情况下，每台 Redis 服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。

**作用**：
  * **数据冗余**：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
  * **故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
  * **负载均衡**：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
  * **高可用（集群）基石**：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是 Redis 高可用的基础。

**复制原理**：
    Slave 启动成功连接到 master 后会发送一个 sync 同步命令，Master 接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集的命令，在后台进程执行完毕之后，master 将传送整个数据文件到 slave，并完成一次完全同步。

   * **全量复制**：slave 服务在接收到数据库文件数据后，将其存盘并加载到内存中。
   * **增量复制**：Master 继续将新的所有收集到的修改命令依次传给 slave，完成同步

>注意：只要是重新连接 master，将自动执行全量复制

**数据同步原理**：

  * Replication Id：简称replid，是数据集的标记，id一致说明是同一数据集，每一个master有唯一的replid，slave会继承master的replid
  * offset：偏移量，随着记录在repl_baklog中的数据增多而逐渐增大。如果slave的offset小于master的offset，说明slave数据落后于master，需要更新

**主从集群优化**

* 限制一个master上的slave数量，如果slave太多，采用主-从-从链式结构
* 适当提高repl_baklog的大小，发现slave宕机时尽快实现故障恢复，避免全量同步
* 在master中配置`repl-diskless-sync yes`启用无磁盘复制，避免全量同步时的磁盘IO


## 哨兵模式

**概念**：

反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库。

哨兵模式是一种特殊的模式，首先 Redis 提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待 Redis 服务器响应，从而监控运行的多个 Redis 实例。

### 哨兵的工作模式

1. 每个Sentinel以每秒一次的频率向它所知的Master，Slave以及其他Sentinel发送一个PING命令
2. 如果最后一次回复PING命令的时间超过down-after-milliseconds所指定的值，该实例会被当前Sentinel标记为主观下线
3. 此时所有sentinel要以每秒一次的频率确认该Master是否进入主观下线状态
4. 当有达到配置文件指定的值的数量的Sentinel在指定时间范围确认Master进入主观下线，那么该Master被标记为客观下线。
5. sentinel节点投票选举一个sentinel节点进行故障处理，在从节点中选取一个主节点，其他从节点复制新主节点数据


### 选举master的标准

1. 首先判断slave节点与master节点断开时间的长短，超过指定值(down-after-milliseconds*10)排除该节点
2. 判断slave节点的slave-priority值，越小优先级越高，为0则永不参与选举
3. 若slave-priority一样，则判断slave的offset值，越大说明数据越新，优先级越高
4. 最后判断slave节点的运行id大小，越小优先级越高(该值唯一，不可能相同)
## 分布式锁
为了保证多台服务器在执行某一段代码时保证只有一台服务器执行。

**Redis对分布式锁的实现**

Redis实现分布式锁主要利用Redis的`setnx`命令。`setnx`是SET if not exists(如果不存在，则 SET)的简写。

**加锁**：使用`setnx key value`命令，如果key不存在，设置value(加锁成功)。如果已经存在lock(也就是有客户端持有锁了)，则设置失败(加锁失败)。

**解锁**：使用`del`命令，通过删除键值释放锁。释放锁之后，其他客户端可以通过setnx命令进行加锁。

key的值可以根据业务设置，比如是用户中心使用的，可以命令为USER_REDIS_LOCK，value可以使用uuid保证唯一，用于标识加锁的客户端。保证加锁和解锁都是同一个客户端。

**超时机制**：如果持有锁的进程宕机会导致死锁，通过`expire`设置key的超时时间


用lua脚本执行保证操作的原子性。
## Cluster模式
哨兵模式每个阶段存储的数据是一样的，浪费内存，且不好在线扩容，因此Redis3.0后引入了Cluster集群模式，实现了Redis的**分布式存储**。对数据进行分片，也就是说**每台Redis节点上存储不同的内容**，来解决在线扩容的问题。并且，它也提供复制和故障转移的功能。

### 集群节点的通讯
一个Redis集群由多个节点组成，通过Gossip协议进行通信，节点之间不断交换信息，常用的Gossip信息分为4种，分别是ping pong meet fail .
1. meet
   
   **通知新节点加入**。消息发送者**通知接收者加入到当前集群**，meet消息通信正常完成后，接收节点会加入到集群中并进行周期性的ping、pong消息交换。

2. ping
   
   集群内交换最频繁的消息，集群内每个节点每秒向多个其他节点发送ping消息，用于**检测节点是否在线和交换彼此状态信息**。

3. pong

   当接收到ping、meet消息时，**作为响应消息回复给发送方确认消息正常通信**。pong消息内部封装了自身状态数据。节点也可以向集群内广播自身的pong消息来通知整个集群对自身状态进行更新。

4. fail

   当节点判定集群内另一个节点下线时，会向集群内广播一个fail消息，其他节点接收到fail消息之后把对应节点更新为下线状态。

>每个节点是通过集群总线(cluster bus) 与其他的节点进行通信的。通讯时，使用特殊的端口号，即对外服务端口号加10000。例如如果某个node的端口号是6379，那么它与其它nodes通信的端口号是 16379。nodes 之间的通信采用特殊的二进制协议。

### Hash Slot插槽算法

Cluster集群使用的分布式算法不是一致性Hash，而是Hash Slot插槽算法。

插槽算法把整个数据库切分为16384个slot(槽)，每个进入Redis的键值对根据key进行散列(如果key中包含"{}"，且“{}”中至少包含1个字符，那么根据“{}”中的部分进行散列)，分配到其中一个。使用的哈希映射也比较简单，用CRC16算法计算出一个16位的值，再对16384取模。

集群中的每个节点负责一部分的hash槽，比如当前集群由A,B,C三个节点，那么
* 节点A负责0-5460号哈希槽
* 节点B负责5461-10922号哈希槽
* 节点C负责10923-16383号哈希槽


## 缓存穿透，击穿，雪崩

### 缓存穿透

**概念**：

缓存穿透是指查询的数据在数据库是没有的，那么在缓存中自然也没有，所以，在缓存中查不到就会去数据库取查询。

**解决方案**：

1. 小数据用Bitmap，大数据用布隆过滤器

2. 对于返回为NULL的依然缓存，对于抛出异常的返回不进行缓存,注意不要把抛异常的也给缓存了。采用这种手段的会增加缓存的维护成本，需要在插入缓存的时候删除这个空缓存，当然可以通过设置较短的超时时间来解决这个问题。

### 缓存击穿
**概念**：

对于某些key设置了过期时间，但是其是热点数据，如果某个key失效，可能大量的请求打过来，缓存未命中，然后去数据库访问，此时数据库访问量会急剧增加。

**解决方案**：

1. **分布式锁**：加载数据的时候可以利用分布式锁锁住这个数据的Key,在Redis中直接使用setNX操作即可，对于获取到这个锁的线程，查询数据库更新缓存，其他线程采取重试策略，这样数据库不会同时受到很多线程访问同一条数据。

2. **异步加载**：由于缓存击穿是热点数据才会出现的问题，可以对这部分热点数据采取到期自动刷新的策略，而不是到期自动淘汰。

### 缓存穿透
**概念**：

缓存雪崩是指缓存不可用或者大量缓存由于超时时间相同在同一时间段失效，大量请求直接访问数据库，数据库压力过大导致系统雪崩。


**解决方案**：
1. **采用多级缓存**：不同级别缓存设置的超时时间不同，及时某个级别缓存都过期，也有其他级别缓存兜底。
2. **缓存的过期时间可以取个随机值**，比如以前是设置10分钟的超时时间，那每个Key都可以随机8-13分钟过期，尽量让不同Key的过期时间不同。
3. **增加缓存系统可用性**,通过监控关注缓存的健康程度，根据业务量适当的扩容缓存。


## 缓存一致性

使用缓存有三种经典的模式
1. **Cache-Aside Pattern**，即旁路缓存模式

**Cache-Aside读流程**

Cache-Aside Pattern的读请求流程如下：
   1. 读的时候，先读缓存，缓存命中的话，直接返回数据
   2. 缓存没有命中的话，就去读数据库，从数据库取出数据，放入缓存后，同时返回响应。

**Cache-Aside写流程**

Cache-Aside Pattern的写请求流程如下：

更新的时候，先更新数据库，然后再删除缓存。


2. **读写穿透**

和旁路缓存模式类似，不过在缓存层上加了一层封装

3. **异步缓存写入**

和读写穿透模式类似，但是**只更新缓存不直接更新数据库**通过批量异步方式更新数据库。适合频繁写的场景(InnoDB Buffer Pool就是使用该模式)


**操作缓存时应该删除而不是更新**

* 举个🌰，A和B发起写操作(A先于B)，AB更新完数据库后，由于网络原因，B先更新了缓存，A再更新缓存。这时缓存保存的是A的数据，也就是脏数据。而且，写多读少的情况下，数据可能还没读又被更新，浪费性能。

**双写的时候为什么先操作数据库？**
* 同样举个🌰，A发起写操作，先删除缓存；B发起读操作，缓存未命中所以读数据库的老数据(因为A将要更新)，B把老数据放入缓存；A在数据库写入新数据。