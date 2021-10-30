





[TOC]



# 数据库优化

## 1. 为什么需要数据库优化

```java
单表数据量过大，查询sql语句过慢
```



## 2.数据库优化方案

```

	数据库优化方案很多，主要分为两大类：软件层面、硬件层面。
	软件层面：SQL调优、表结构优化、读写分离、数据库集群、分库分表等。
	硬件层面：主要是增加机器性能，性能瓶颈就是CPU、内存、磁盘、网络这些。

```



## 3. 软件层面优化

### 3.1 SQL调优
**主要目的：让SQL执行尽量命中索引**

```markdown

# 参数1：slow_query_log        慢查询开启状态
# 参数2：slow_query_log_file   慢查询日志存放的位置
# 参数3：long_query_time       查询超过多少秒才记录

```



```markdown

# 1.mysql数据库中查看3个参数的key-value值
	  show variables like 'slow_query%';
	  show variables like 'long_query_time';
# 2.更改参数并查看慢sql
	  -- 开启慢查询：
	  set global slow_query_log='ON'; 
	  -- 日志位置：
	  set global slow_query_log_file='/usr/local/mysql/data/slow_query.log';
	  -- 慢查询1s就记录：
	  set global long_query_time=1;
	  -- 查看日志
	  ls /usr/local/mysql/data/slow_query.log  
# 3.调优工具(explain命令)
      -- 利用explain分析sql查询情况
      EXPLAIN SELECT * FROM pms_sku_sale_attr_value LIMIT 10;
      
      返回有一列叫“type”，常见取值有：
        ALL、index、range、 ref、eq_ref、const、system、NULL（从左到右，性能从差到好）
        ALL 代表这条 SQL 语句全表扫描了，需要优化。一般来说需要达到range 级别及以上。
      
```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210408093242.png)



### 3.2 表结构优化

```markdown

	表结构优化又叫数据库表冗余字段，为了减少join联表查询
	
	注：冗余字段更新的时候会涉及多张表，所以尽量选择不经常更新的字段。

```



### 3.3 架构优化

```markdown

# 集群
当单台数据库实例扛不住，可以增加实例组成集群对外服务;
# 读写分离
当发现读请求明显多于写请求时，我们可以让主实例负责写，从实例对外提供读的能力；
# 缓存
读实例压力依然很大，可以在数据库前面加入缓存如 redis，让请求优先从缓存取数据减少数据库访问
# 分库分表
缓存分担了部分压力后，数据库依然是瓶颈，这个时候就可以考虑分库分表的方案了

```



### 分库分表

````markdown

# 分库分表定义
分库：由单个数据库实例拆分成多个数据库实例，将数据分布到多个数据库实例中。
分表：由单张表拆分成多张表，将数据划分到多张表内

# 分库分表目的
分库：目的是减轻单台MySQL实例存储压力及可扩展性
分表：是解决单张表数据过大以后查询的瓶颈问题

````





#### 3.3.1 单应用单数据库

```markdown
	
# 单体架构应用
在早期创业阶段想做一个商城系统，基本就是一个系统包含多个基础功能模块，最后打包成一个 war 包部署

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210409092119.png)



#### 3.3.2 多应用单数据库

```markdown

# 多应用单数据库
多个服务共享一个数据库

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210409092457.png)



#### 3.3.3 多应用多数据库

```markdown

# 多应用多数据库
	1.分库：每个服务相关的表拆出来单独建立一个数据库，这其实就是“分库”了。
	  
	2.分表(单表500万数据就要考虑分表了)
	  分表有几个维度，一是水平切分和垂直切分，二是单库内分表和多库内分表。
	  	
	  分表小结：分表主要是为了减少单张表的大小，解决单表数据量带来的性能问题
	  	
```



```markdown

# 分库分表带来的问题
## 1.跨库关联查询
		在单库未拆分表之前，使用 join 操作关联多张表查询数据，
		但是经过分库分表后两张表可能都不在一个数据库中，无法使用join关键词
		解决方案：
		①字段冗余：把需要关联的字段放入主表中，避免 join 操作；
	    ②数据抽象：通过ETL等将数据汇合聚集，生成新的表；
		③全局表：比如一些基础表可以在每个数据库中都放一份；
		④应用层组装：将基础数据查出来，通过应用程序计算组装；	
## 2.分布式事务
		单数据库可以用本地事务搞定，使用多数据库就只能通过分布式事务解决了。
		常用解决方案有：基于可靠消息（MQ）的解决方案、两阶段事务提交、柔性事务(TCC)等。
## 3.排序、分页、函数计算问题
    	在使用 SQL 时 order by， limit 等关键字需要特殊处理，一般来说采用分片的思路：
		先在每个分片上执行相应的函数，然后将各个分片的结果集进行汇总和再次计算，最终得到结果。
## 4.分布式 ID
        Mysql 数据库在单库单表可以使用 id 自增作为主键，分库分表了之后就不行了，会出现id 重复
        常用的分布式 ID 解决方案有
          ①UUID
	      ②基于数据库自增单独维护一张 ID表
	      ③号段模式
	      ④Redis 缓存
	      ⑤雪花算法（Snowflake）
	      ⑥百度uid-generator
	      ⑦美团Leaf
	      ⑧滴滴Tinyid
## 5.多数据源 
	分库分表之后可能会面临从多个数据库或多个子表中获取数据，一般的解决思路有：客户端适配和代理层适配
	 	
	 业界常用的中间件有
	  ①shardingsphere（前身 sharding-jdbc）
	  ②Mycat
		
```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210409092712.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210409093354.png)







## 4.硬件层面优化

```

	前期升级硬件数据库性能可以得到较大提升；后期，升级硬件得到的收益就不那么明显了

```





















