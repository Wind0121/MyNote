# OLTP & OLAP
![[Pasted image 20231107152011.png]]
OLTP主要用于大量小规模的读和写——重点在Transaction
OLAP主要用于大规模的查询操作——重点在Analytical

# 行存储
![[Pasted image 20231107155050.png]]
行存储的问题在于出现了很多无用数据，而这些数据是根本不需要的。

# 列存储
![[Pasted image 20231107155135.png]]
列存储在一个page种存储相同属性的值，这就便于我们进行查找。但随之而来的问题是如何将数据进行缝合：即多个列值缝合为一个tuple的值。

**列缝合**
![[Pasted image 20231107155230.png]]
大多数采用偏移量的方法来进行缝合，这样做的计算量较小。