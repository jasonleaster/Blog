---
title: 'Database System: simple-db lab4'
date: 2018-01-25 23:41:46
tags: Database, MIT
---
lab 4开始体验基于锁的并发事务控制，这是本人最喜欢的一节。并发锁的控制策略和实现在我看来是非常有意思的。

### 什么是事务？

先谈谈我们在系统实现时遇到的挑战。有时，我们需要一连串的操作对外表现得就像一个操作一样，要么这一串操作都被执行了，要么都不执行，不会对外表现出执行了一半的中间状态。有时，可能会有多个用户客户端同时并发的访问同一个数据块，两者不会相互影响，不出现数据篡改等情况。 

对于并发执行时，解决并发问题的特性归纳总结有四方面的特性(ACID)，人们把满足这四个特性的执行操作称为事务(Transaction)。这四个特性分别是原子性(A)、一致性(C)、隔离性(I)、持久性(D)。

特别是在数据库管理系统(DBMS)中，最小的逻辑执行单元即事务。我们说“事务”主要是关注它的四个属性。

Atomicity: All actions in the txn happen, or non happen. ("all or nothing")

Consistency: If each txn is consistent and the DB starts consistent, then it ends up consistent. ("it looks correct to me")

Isolation: Execution of one txn is isolated from that of other txns. ("as if alone")

Durability: If a txn commits, its effects persist.("survive failures")

具体的针对SQL来说，一个新的SQL以关键字`BEGIN`起始，以`COMMIT`或`ABORT`结束。一旦commit了，在该事务内的操作对数据产生的影响都会保存持久化。如果是abort中断，那么该事务内的数据改变都会被撤销，好像这个事务中的操作都没有发生过一样。事务让一组动作变成了一个不可分割的单元。

接着说说经典的转账例子。Jason的某银行账户有100块，他发起请求将这100块转账到自己的支付宝账户，如果这个转账请求的消息发送出去之后恰巧由于网络传输延迟或者丢包影响，远端服务器并没有收到这个转账请求，此时远端服务器和本地客户端都挂掉了，请问Jason的这一百块是不是就蒸发了？如果转账操作能以事务的形式执行多好，要么发生，要么不发生，具有原子性。那么原子性如何实现呢？

对于数据库而言，实现原子性有两个比较典型的方式:
1. 日志
DBMS负责记录事务内的所有的操作，并提供针对性的"撤销方法"。

2. 影子页
DBMS对事务内弄脏的数据页提前拷贝，左右数据增删改操作基于数据的拷贝来操作。仅当事务commit之后才会将这些被事务修改后的数据拷贝替换原始数据页并对外公开访问。

### 锁的基本类型

共享锁: 对于读操作之间共享
排它锁: 对于读写操作都互斥，即排他锁不仅与排它锁互斥，而且还与共享锁互斥，阻塞共享锁的获取

下面是共享锁和排他锁的兼容性矩阵:

Lock Type| Shared | Excluded 
---------|--------|--------- 
Shared   | T      | F       
Excluded | F      | F       

排它锁/写锁是非常霸道的, 他持有数据的时候，别人读都不行。

### 二阶段锁(Two-Phase Locking)

在数据库的事务处理中，二阶段锁协议只是一种并发控制的策略方法，他的出现是为了解决并发冲突序列化的问题。

两段锁协议的内容:
1. 在对任何数据进行读、写操作之前，事务首先要获得对该数据的封锁
2. 在释放一个封锁之后，事务不再获得任何其他封锁。

从事务的角度来看，锁的控制分为两个阶段，扩展阶段(Growing)和收缩阶段(Shrinking)。
![images](/images/img_for_2018_01/TwoPhaseLocking.jpg)

一旦获取锁的阶段过去了，即进入到收缩阶段，那么是不再允许获取锁的。
![images](/images/img_for_2018_01/TwoPhaseLockingVialationDemo.jpg)

> The two phase locking rule can be summarized as: never acquire a lock after a lock has been released. -- Wikipedia

但是以上二阶段协议对于并发控制的约束还是很弱，事务没有结束之前就释放锁会引入很多问题，例如典型的会导致Cascading Abort。
![images](/images/img_for_2018_01/2PL_CascadingAbort.jpg)
上图中左边事务T1在Commit之前释放了A锁，那么此时事务T2对A锁竞争成功并对A锁控制的数据进行读写操作，但是T1突然中断需要回滚，那么T2读到的关于A锁数据的内容都需要回滚，T2也被迫需要回滚，故形成了一种级联回滚的事件。

针对由于事务过程中释放write lock尔后回滚，进一步脏读引起的Cascading Abort的问题，解决方案就是S2PL.

二阶段锁也分三种类型:

* 保守式二阶段锁(Conservative two-phase locking)
C2PL与2PL的区别就在于，C2PL要求事务在开始之前就获取所有需要的锁，拿不到就不开始。说C2PL能预防死锁，那是因为既然事务开始之前能拿到所有的锁，就可以对锁排序，或构建一定的顺序关系，竞争成功则继续进行，竞争失败就等待重试。

* 严格二阶段锁(Strict two-phase locking)
S2PL基于2PL，而不是C2PL，也就是说S2PL是可能死锁的。仅在事务结束的时候（commit or rollback），才释放所有 write lock，read lock 则正常释放。因为事务结束才释放write lock，避免了一个事务回滚引发其他事务级联回滚。

* 强严格二阶段锁(Strong strict two-phase locking)
SS2PL也是基于2PL，仅在事务结束的时候（commit or rollback），才释放所有锁，包括write lock 跟 read lock 都是结束后才释放。

所以这个时候我们能够梳理明白，2PL能够很大程度上序列化事务并发竞争。但是这还不够，2PL存在dirty read的问题，会进一步导致cascading abort，解决这个问题的症结在于commit之前释放了write lock，于是我们不commit就不释放，把锁的持有时间拉长一点，并发度低一点，就有了S2PL。但是不像C2PL，S2PL和SS2PL都没有在事务开始之前获取所有的锁，都是在事务过程中去获取锁，故也会有死锁的可能。

### 死锁

什么是死锁？
A deadlock is a cycle of transactions waiting for locks to be released by each other.

处理死锁的两种方法: 死锁检测(deadlock detection)和死锁预防(deadlock prevention).

关于死锁检测，DBMS创建一个"等待图"(waits-for graph)即可，
1. 等待图中所有的节点都代表事务
2. Ti -> Tj 两个节点的连接表示事务i正在等待事务j释放事务j持有的锁

实际上在simpledb的实现中，我也是用的map数据结构来实现这种映射关系，key就是当前事务，value就是当前事务正在等待的事务们，由于后者有多个故采用了Set结构。

``` java

    /**
     * 页面共享锁管理器
     */
    private final Map<PageId, Set<TransactionId>> sharedLockManager = new ConcurrentHashMap<>();

    /**
     * 页面排它锁管理器
     */
    private final Map<PageId, TransactionId> exclusiveLockManager = new ConcurrentHashMap<>();
```

接下来只需要在map构建的图结构里面搜索就能发现是否有环结构，如果有环，那就是死锁，DBMS根据情况让环中部分事务回滚，打破环，进而解决循环依赖死锁的问题。

关于选取哪个事务被牺牲回滚的问题，主要考虑以下几个方面:
1. 事务生存时长
2. 事务被执行的次数
3. 持有锁的数目
等等

至于死锁的预防就很简单了，当一个事务尝试获取的锁被其他事务持有时，权衡一下，回滚其中一个就可以预防死锁。这种策略不需要"waits-for"图，因为根本就没有等待。。。

但是有一点需要注意，当一个事务由于并发竞争策略预防死锁而被杀死的时候，重新运行时，他的优先级(或时间戳)应该与之前被杀时一样，即继承之前被杀时的时间戳。


### 意向锁

考虑一个问题，如果一个事务需要对1000个tuple加锁，是不是得执行1000次加锁操作呢？

显然，大家都不希望这么繁琐。最好锁能够分层级(表、行等等)的对数据进行锁定。

![images](/images/img_for_2018_01/databaseHierarchy.jpg)
