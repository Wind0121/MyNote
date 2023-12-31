# Multi-Version Concurrency Control
- 多版本并发控制 (MVCC)是一个比并发控制协议**更大的概念**
- DBMS 在数据库中维护单个逻辑对象的多个物理版本。
	- 当事务写入对象时，DBMS 创建该 对象的新版本。
	- 当事务读取对象时，它读取的是事务开始时存在的最新版本
- MVCC 的基本概念/好处是写者不阻塞写者，读者不阻塞读者。
	- 这意味着一个事务可以修改一个对象，而其他事务则读取旧版本。
- 使用 MVCC 的一个优点是只读事务可以在不使用任何类型的锁的情况下读取数据库的一致快照。此外， MVCC 可以很容易地支持时间旅行查询

有四个重要的MVCC设计策略：
- 并发控制协议
- 版本存储
- 垃圾收集
- 索引管理

单纯的MVCC最高只能实现快照隔离，快照隔离不是可串行化的，因而通常要配合其他并发控制协议来实现。

# 并发控制协议
![[Pasted image 20231216150939.png]]
就是前两节课提到的T/O、OCC、2PL

# 版本存储
- 版本存储是指 DBMS 如何存储逻辑对象的不同物理版本，以及事务如何找到对它们可见的最新版本。
- DBMS 使用元组的指针字段为每个逻辑元组创建一个版 本 链 ，它本质上是一个按时间戳排序的版本链表。 这允许 DBMS 在运行时找到对特定事务可见的版本。
- 索引总是指向链的“头”，根据实现的不同，它可能是最新的版本，也可能是最旧的版本。
- 不同的存储方案决定了每个版本的存储位置/内容

## Approach #1: Append-Only Storage
- 逻辑元组的所有物理版本都存储在同一个表空间中。
- 版本在表中混合在一起，每次更新只是将元组的一个新版本追加到表中，并更新版本链。
- 这个链可以是旧到新(O2N)，需要在查找时进行链式遍历，也可 以是新到旧(N2O)，需要为每个新版本更新索引指针
![[Pasted image 20231216151513.png]]

## Approach #2: Time-Travel Storage
- DBMS 维护一个单独的表，称为时间旅行表，用于存储旧版本的元组
- 每次更新时，DBMS 都会将旧版本的元组复制到时间旅行表中，并用新数据覆盖主表中的元组
- 主表中的元组指针指向时间旅行表中的过去版本
![[Pasted image 20231216151805.png]]

## Approach #3: Delta Storage
- 类似于时间旅行存储，但不是整个过去的元组，DBMS 只存储增量，或元组之间的变化，即所谓的增量存储段
- 事务可以通过迭代增量来重新创建旧版本
- 这导致写入速度比时间旅行存储更快，但读取速度更慢
![[Pasted image 20231216151951.png]]

# 垃圾回收
随着时间的推移，DBMS 需要从数据库中删除可回收的物理版本。如果一个版本没有活动的事务可以 “看到”这个版本，或者它是由一个被中止的事务创建的，那么这个版本就是可回收的

## Approach #1: Tuple-level GC
DBMS 通过直接检查元组来查找旧版本。有两种方法可以实现这一点：
- Background Vacuuming：单独的线程定期扫描表并寻找可回收的版本。一个简单的优 化是维护一个“脏页位图”，记录自上次扫描以来哪些页被修改过。这使得线程可以跳过没有修改过的页面。
- Cooperative Cleaning：工作线程在遍历版本链时识别可回收版本。这只适用于 O2N 链

## Approach #2: Transaction-level GC
- 每个事务负责跟踪它们自己的旧版本，因此 DBMS 不必扫描元组。每个事务维护自己的读/写集合。
- 当一个事务完成时，垃圾收集器可以使用它来标识要回收哪些元组。DBMS 确定完成事务创建的所有版本何时不再可见。
- 实际上就是事务记录哪些tuple被修改过，从而不需要进行定期扫描
![[Pasted image 20231216153613.png]]

# 索引管理
所有主键(pkey)索引总是指向版本链头。DBMS 更新 pkey 索引的频率取决于当元组更新时系统是否创建新版本。如果一个事务更新了一个(多个)pkey 属性，那么这被视为一个删除，然后是一个插入。

