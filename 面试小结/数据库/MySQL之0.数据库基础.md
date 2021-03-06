



[TOC]



# MySQL基础



## 1.MySQL的日志

- 重做日志（redolog）：确保事务的持久性（粉板）。InnoDB引擎特有的
- 回滚日志（undolog）：事务回滚日志。InnoDB引擎特有的
- 二进制日志（binlog）：实现主从同步（账本）。MySQL的Server层实现的，所有引擎都可以使用





## 2.索引

- 主键索引
- 唯一索引（幂等性）
- 一般索引
- 覆盖索引（select中查索引字段，二级索引不回表查询）

注：主键索引是聚簇索引，其他索引都是非聚簇索引（二级索引）



  

**主键索引**

- 非叶子节点存储主键和页号
- 叶子节点存储完整的数据
- 叶子节点之间有双向链表链接，便于范围查询
- 叶子节点内部有页目录，内部记录是单链表链接，通过页目录二分再遍历链表即可得到对应记录。
- B+ 树只能帮助快速定位到的是页，而不是记录。
- 页大小默认16k，是按照主键大小排序的，所以无序的记录插入因为排序会插入到页中间，又因为容量有限会导致页分裂存储，性能比较差，所以主键要求有序



**非主键索引**

- 和主键索引的差别就在于叶子节点存储索引列和主键，没有完整的数据 





## 3.MySQL索引优化

**Explain索引执行计划：type 至少range级别；key 实际用到的索引**



索引失效

-  最佳左前缀法则 
-  使用不等于(!=  或者<>)的时候，该索引会失效
-  like模糊匹配如果%在前面会失效
-  不要在索引列上做任何计算 （如果非要计算可以创建索引时也计算）
-  索引列上范围查询之后的字段会失效
- or会导致索引失效







