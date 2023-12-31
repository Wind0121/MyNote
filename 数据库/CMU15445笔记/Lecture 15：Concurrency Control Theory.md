# Transaction（事务）
事务是在数据库上执行一系列的一个或多个操作(例如SQL查询)，以执行一些高级功能。

事务必须是原子的，不能是部分的。

事务是数据库执行的最基本单位：默认一个SQL语句就是一个事务，当然也可以指定多个SQL语句为一个事务

数据库本身只关注数据的读出和写入，不知道外部世界的要求。因而数据库需要单单依靠SQL语句来判断是否能并发执行事务

# Definition
数据库：一个数据库可以表示为一组固定的命名数据对象。这些对象可以是属性、元组、页面、表。

事务：事务是在这些对象上的一系列读写操作(即 R(A)， W(B))。为了简化讨论，这个定义假设数据库是一个固定大小的，因此操作只能是读取和更新，而不是插入或删除。

事务的边界由客户端定义。在 SQL 中，事务以 BEGIN 命令开始。事务的结果是 COMMIT 或 ABORT。
- 对于 COMMIT，要么将事务的所有修改保存到数据库中，要么 DBMS 覆盖它并中止
- 对于 ABORT（ROLLBACK） ，事务的所有更改都将撤消，因此就像事务从未发生过一样。中断可以是自己造成的，也可以是由 DBMS 引起的

**事务正确性**的标准：ACID
- Atomicity（原子性）：事务内的操作要么都发生，要么都不发生
- Consistency（一致性）：数据库在事务开始前和结束后是一致的，即总的规则不会不胜变化
- Isolation（隔离性）：一个事务在执行时，应有一种与其他事务隔离的错觉
- Durability（持久性）：事务提交后，对数据库的影响应该是永久的

# ACID：Atomicity
事务执行的结局：要么都执行，要么都不执行
- Commit after completing all its actions
- Abort (or be aborted by the DBMS) after executing some actions

## 实现方式一：日志
- 记录所有的操作，以便进行回滚
- 同时在内存和硬盘上记录日志
- 几乎所有现代系统都使用日志

## 实现方式二：影子分页
- DBMS 生成由事务修改的页面的副本，事务对这些副本进行更改。只有当事务提交时，页面才可见
- 运行时通常比基于日志的 DBMS 慢
- 恢复比基于日志的DBMS快
- 通常不被选择

# ACID：Atomicity
数据库所表示的“世界”在逻辑上是正确的。所有关于数据的问题都给出了逻辑正确的答案。

## 数据库一致性
数据库准确地表示它正在建模的真实世界的实体，并遵循完整性约束。(例如，一个人的年 龄不可能为负数)。此外，未来的事务应该在数据库内部看到过去提交的事务的影响

## 事务一致性
如果在事务开始(单独运行)之前数据库是一致的，那么在事务开始之后它也将是一致的
（例如，银行转账，不可能取出100，只转出90）

事务一致性是应用程序的责任。DBMS无法控制这个。

# ACID：Isolation
- DBMS 给事务提供了它们在系统中单独运行的错觉。它们看不到并发事务的影响。
- 这相当于数据库每次似乎只执行一个事务
- 但事实上，数据库中的事务时并发的

## 并发控制
并发控制协议是 DBMS 在运行时如何决定来自多个事务的操作的适当交错

有两种类型的并发控制协议：
1. 悲观的：DBMS假定会发生冲突，从而不执行并发
2. 乐观的：DBMS假定很少发生冲突，从而执行并发，如果冲突再回滚

DBMS 执行操作的顺序称为执行计划。我们希望交错事务来最大化并发性，同时确保输出是“正确的“。因而并发控制协议的目标是生成一个**等同于某些串行执行的执行时间表**：
- 串行调度：不交错事务的调度方式，这种方式一定是正确的，结果仅仅取决于想以什么方式执行事务。例子如下：
![[Pasted image 20231212195255.png]]
- 等效调度：对于任何数据库状态，如果执行第一个调度的效果与执行第二个调度的效果相同，则两个调度是等效的
- 可串行化调度：可串行化调度是指与事务的某种串行执行等价的调度。这可以保证结果是正确的。
![[Pasted image 20231212195439.png]]


现在我们有一个问题，左边这种事务显然是错误的，但对于数据库来说，它只能看到右边的形式，那么数据库该如何判断交错调度是正确的？
![[Pasted image 20231212195626.png]]

## 冲突操作
冲突发生的条件：
- 两个操作来自不同的事务，但又作用于同一个对象
- 至少有一个操作是写操作

冲突变体：
- 读写冲突（不可重复读）：一个事务多次获取一个对象时无法获取相同的值（要让事务有一种独立运行的错觉）
- 写读冲突（脏读）：一个事务在提交其更改之前看到了另一个事务的写效果
- 写写冲突（丢失更新）：一个事务覆盖另一个并发事务的未提交事务

可串行化有多个级别：
1. 基于冲突的可串行化（Most DBMSs try to support this）
2. 基于观察的可串行化（No DBMS can do this）

## 基于冲突的可串行化
两个调度是冲突等效的定义：
- 涉及相同事务的相同操作
- 每一对冲突操作在两个调度中以相同的方式排序

Schedule S是conflict serializable（冲突可串行化）的：
- S冲突等效于某个串行调度
- 直觉：可以通过交换不同事务的**连续无冲突**操作，将S转换为串行调度。这种方式在面对多个事务时会变得过于昂贵

判断冲突可串行化的**例子如下**：
![[Pasted image 20231212202320.png]]
在T1和T2两个事务中，R(A)、R(B)和W(B)、W(A)是连续不冲突的（**冲突条件见前面**），因而可以将它们彼此交换，得到以下结果。从而看出该调度与串行调度是等效的，因而是冲突可串行化。
![[Pasted image 20231212202546.png]]

**反例如下**，这种情况无法将左边这种调度进行串行化，也就是不正确的。
![[Pasted image 20231212202809.png]]

当调度中只有两个事务时，交换操作很容易。当有很多事务时，这很麻烦。更好的方法是利用**依赖图**。

**依赖图**
- 在依赖图中，每个事务都是图中的一个节点。
- 如果来自 Ti 的操作 Oi 与来自 Tj 的操作 Oj 冲突，并且 Oi 发生在调度中的 Oj 之前，则存在从节点 Ti 到 Tj 的有向边。
- 一个调度是冲突可序列化的依赖图是**无环的**
![[Pasted image 20231212203519.png]]

**基于观察的可串行化**
难以实现，不做赘述

## 小结
在所有调度中，串行调度只占很小一部分，冲突可串行化调度扩展了一部分，基于观察的可串行化调度又扩展了一部分。
![[Pasted image 20231212205342.png]]

# ACID：Durability
所有提交事务的更改必须在崩溃或重启后是持久的。通过日志或影子分页都可以实现

# 总结
- 并发控制和恢复是DBMS提供的最重要的功能之一
- 并发控制是自动的
	- 系统自动插入锁/解锁请求，并调度不同txns的动作
	- 确保结果执行等同于以某种串行顺序一个接一个地执行txns

