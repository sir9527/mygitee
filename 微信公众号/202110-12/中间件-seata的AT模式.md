

```

TC事务协调器：zip包

```



#### 分布式事务Seata

分布式事务处理过程的 "ID + 三组件模型"
- Transaction ID全局唯一的事务ID
- 三组件概念
```

1.TC (Transaction Coordinator):事务协调者。管理全局事务和分支事务。
2.TM (Transaction Manager):事务管理器。全局事务。
3.RM (Resource Manager):资源管理器。分支事务，需要注册到TC中
 
TC管理全局事务和分支事务的状态 
TM管理全局事务，包括开启全局事务，提交/回滚全局事务
RM管理分支事务

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210525092804.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210525093238.png)



```

1.TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID
2.XID在微服务调用链路的上下文中传播;
3.RM向TC注册分支事务，将其纳入XID对应全局事务的管辖
4.TM向TC发起针对XID的全局提交或回滚决议;
5.TC调度XID下管辖的全部分支事务完成提交或回滚请求

```



##### 1.seata的AT模式使用

###### 1.1 seata的TC服务

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210530115138.png)





###### 1.2 配置注册中心"registry.conf"

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210530120246.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210530120343.png)





###### 1.3 配置文件"file.conf"不变

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210526093948.png)



###### 1.4 据库初始化

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210530120845.png)



```

-- seata（注：各个微服务的回滚表 undo_log也是seata提供的）

CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(96),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;


```



```

-- 订单数据库
CREATE DATABASE seata_order;

CREATE TABLE t_order(
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
    `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
    `count` BIGINT(11) DEFAULT NULL COMMENT '数量',
    `money` BIGINT(11) DEFAULT NULL COMMENT '金额',
    `status` BIGINT(11) DEFAULT NULL COMMENT '订单状态: 0创建中 1已完结'
) ENGINE = INNODB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;


-- seata提供的回滚表
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

```



```

-- 库存数据库
CREATE DATABASE seata_storage;

CREATE TABLE t_storage(
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
    `total` BIGINT(11) DEFAULT NULL COMMENT '总库存',
    `used` BIGINT(11) DEFAULT NULL COMMENT '已用库存',
    `residue` BIGINT(11) DEFAULT NULL COMMENT '剩余库存'
) ENGINE = INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
 
INSERT INTO t_storage( `id`,`product_id`, `total`,`used`,`residue`) values(1,1,100,0,100);

SELECT * FROM t_storage;


-- seata提供的回滚表
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

```



```

-- 支付数据库
CREATE DATABASE seata_account;

CREATE TABLE t_account(
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
    `total` BIGINT(11) DEFAULT NULL COMMENT '总额度',
    `used` BIGINT(11) DEFAULT NULL COMMENT '已用额度',
    `residue` BIGINT(11) DEFAULT NULL COMMENT '剩余额度'
) ENGINE = INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
 
INSERT INTO t_account( `id`,`user_id`, `total`,`used`,`residue`) values(1,1,1000,0,1000);

SELECT * FROM t_account;


-- seata提供的回滚表
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

```



###### 1.5 启动nacos和seata

```

1.nacos管理界面：http://127.0.0.1:8848/nacos/#/login

2.进入bin目录并执行"seata-server.bat"

```



##### 2.订单系统官网案例

```

三个服务
1.仓储服务：对于给定的商品扣除仓储数量。
2.订单服务：根据采购需求创建订单。
3.账户服务：从用户账户中扣除余额

用户购买商品的四个步骤：
1.用户下订单 创建订单
2.订单系统远程调用库存服务扣减下单商品的库存
3.订单系统再远程调用账户服务来扣减用户账户里面的余额
4.订单服务修改订单状态为已完成
注：该操作跨3个数据库，又2次远程调用，需要使用分布式事务

```



**微服务架构**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210530121349.png)





````

业务逻辑：下订单 --> 减库存 --> 减账户金额

注：seata在需要最开始的下订单用"@GlobalTransactional"全局事务控制跨库事物

````



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210530123741.png)





**需要注意各个微服务需要用自己的应用名字去到TC注册**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210530121733.png)











