## WAL
为提高MySQL更新数据的效率，避免频繁的硬盘读写，InndDB引入了WAL(Write Ahead Log)机制,当更新数据库数据时，先写入日志，再写入DB文件中。(Redo Log)

>MySQL创建与更新数据的步骤：沿着 B+ tree到达叶子节点进行插入与更新操作，最坏情况下每一层都需要进行一次随机硬盘IO,而插入数据还会可能会导致页分裂问题。

>页分裂：https://zhuanlan.zhihu.com/p/98818611

## redo log

**redo log是一个顺序写入，大小固定的环形日志**,记录了对物理数据页的修改。(物理日志)

redo log维护了两个指针，一个write-pos记录当前写入位置，一个check-pos记录当前要擦除的位置，擦除记录需要把数据持久化到.ibd文件中。

**数据写入流程**：

当一个写事务或者更新事务执行时，InnoDB首先取出对应的Page，然后进行修改。当事务提交时，将内存中的redo log buffer 强制刷新到硬盘中。

InnoDB的Master Thread会定时地我们修改的页面(脏页)刷到硬盘，此时数据真正写入到.ibd文件中。

## bin log

由MySQL的server层实现，主要作用：
* 主从复制
* 记录MySQL中的逻辑操作，比如更新了哪条数据

binlog的日志格式：

* Statement：只记录执行的SQL语句，占用空间较少(如果主从复制中SQL语句调用了系统函数，容易出现主从不一致情况)
* Row：记录更新前和更新后的数据，占用空间多。

由于两个日志的存在，需要通过两阶段提交读来保持redo log和bin log的一致性

**redolog 和bin log区别**

|          |            redo log            |         bin log          |
| :------: | :----------------------------: | :----------------------: |
|   大小   |   固定大小，循环写，会被覆盖   |     追加写，不会覆盖     |
| 实现层面 |         InnoDB存储引擎         |     MySQL的Server层      |
|   内容   | 物理日志，记录某个数据页的修改 | 逻辑日志，语句的原始逻辑 |
|   作用   |        提高写入数据效率        |    主从复制，数据恢复    |

## 两阶段提交读

**目的:** 为了保证redo log和binlog在逻辑上的一致性

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220115195426.png)


## undo log

undo log用来回滚行记录到某个版本，保存事务未提交之前的版本数据，供其他并发事务进行快照读，是为实现事务原子性出现的产物。

InnoDB使用undo log实现MVCC和进行事务回滚。
## 参考
* 极客时间，《MySQL 实战 45 讲》


