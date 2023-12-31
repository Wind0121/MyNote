# 概述
- 由于 SQL 是声明性的，因此查询只告诉 DBMS 要计算什么，而不告诉它如何计算
- DBMS 需要 将 SQL 语句转换为可执行的查询计划
- DBMS 优化器的工作是为任何给定的查询选择最优计划

查询优化有两种策略：
- 启发式/基于规则：将查询的部分内容与已知的模式进行匹配，以组合出一个计划
- 代价估计：使用基于成本的搜索来读取数据，并估计执行等效计划的成本

数据库体系：
![[Pasted image 20231206151207.png]]
我们的工作集中在优化器部分，前面的跟编译原理相关

# Logical Query Optimization
常见规则如下：
- 拆分复杂谓词
- 尽早执行过滤器（谓词下推）
- 对谓词进行重新排序，以便 DBMS 首先应用选择性最强的谓词
- 拆分连接谓词
- 投影下推
![[Pasted image 20231206152644.png]]

投影优化：
- 投影下推，尽早执行投影
- 去除不需要的属性

谓词优化：
- 删除不可能的谓词
- 合并不必要的谓词
![[Pasted image 20231206153237.png]]

嵌套查询优化：
- 重写子查询
![[Pasted image 20231206153409.png]]
- 分解子查询
![[Pasted image 20231206153425.png]]

# Cost Estimations
- DBMS 使用成本模型来预测数据库特定状态下的查询计划行为
- 运行所有可能的计划来确定这些信息的成本太高，因此DBMS需要一种方法来获取这些信息

代价估计选择：
![[Pasted image 20231206154425.png]]

为了估算查询的成本，DBMS 在其内部编目中维护有关表、属性和索引的内部统计信息。大多数系统试图通过维护一个内部统计表来避免实时计算。

# Plan Enumeration
执行基于规则的重写后，DBMS 将列举查询的不同计划，并估算其成本
- Single relation
- Multiple relations
- Nested sub-queries
在查询完所有计划或超时后，它会为查询选择最佳计划。

## Single-Relation Query Plans
对于单关系查询计划，最大的障碍是选择最佳的访问方法(即顺序扫描、二分查找、索引扫描等)

## Multi-Relation Query Plans
对于多关系查询计划，随着连接数量的增加，备选计划的数量也会迅速增长。因此，重要的是要限制搜索空间，以便能够在合理的时间内找到最优计划。

有两种方法来解决这个搜索问题：
- 自下而上：从零开始，然后制定计划，达到你想要的结果
- 自上而下：从你想要的结果开始，然后沿着树向下工作，找到能让你达到目标的最佳计 划

