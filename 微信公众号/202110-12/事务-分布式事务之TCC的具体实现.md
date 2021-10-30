

[TOC]

# hmily_TCC

### 介绍

TCC：try-confirm-cancel
重点：hmily官网用TCC实现分布式事务的下订单实战项目

#### 1.TCC的实际场景使用

##### 1.1 A和B转账场景TCC的实现

###### 实现1：



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210529183819.png)





![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210529195741.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210529200906.png)





###### 实现2：

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210529183913.png)





###### 实现的架构图

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210529184510.png)





##### 1.2 下订单 --> 减库存 --> 扣钱 (TCC官网实战项目)

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210529201055.png)



###### 微服务模块设计

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210530094527.png)



```

- eureka      注册中心eureka     
- order       订单系统           hmilyTransactionBootstrap
- inventory   库存系统           hmilyTransactionBootstrap
- account     支付系统           hmilyTransactionBootstrap
 

注：swagger界面 http://127.0.0.1:8884/swagger-ui.html#
   eureka界面  http://localhost:8768
   
```



###### 数据库设计

````

-- 数据库准备
-- 数据库1：tcc         --> hmily会在运行的时候为各个注册的微服务自动生成一张表记录信息
-- 数据库2：tcc_account --> account支付表
-- 数据库3：tcc_stock   --> inventory库存表
-- 数据库4：tcc_order   --> order订单表

````



```

/*
SQLyog Ultimate v12.2.6 (64 bit)
MySQL - 5.7.19-0ubuntu0.16.04.1 : Database - account
*********************************************************************
*/
CREATE DATABASE `tcc` ;

CREATE DATABASE /*!32312 IF NOT EXISTS*/`tcc_account` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin */;

USE `tcc_account`;

/*Table structure for table `account` */

DROP TABLE IF EXISTS `account`;

CREATE TABLE `account` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` varchar(128) NOT NULL,
  `balance` decimal(10,0) NOT NULL COMMENT '用户余额',
  `freeze_amount` decimal(10,0) NOT NULL COMMENT '冻结金额，扣款暂存余额',
  `create_time` datetime NOT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

/*Data for the table `account` */

insert  into `account`(`id`,`user_id`,`balance`,`freeze_amount`,`create_time`,`update_time`) values

(1,'10000',10000,0,'2017-09-18 14:54:22',NULL);



CREATE DATABASE /*!32312 IF NOT EXISTS*/`tcc_stock` /*!40100 DEFAULT CHARACTER SET utf8mb4 */;

USE `tcc_stock`;

/*Table structure for table `inventory` */

DROP TABLE IF EXISTS `inventory`;

CREATE TABLE `inventory` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `product_id` VARCHAR(128) NOT NULL,
  `total_inventory` int(10) NOT NULL COMMENT '总库存',
  `lock_inventory` int(10) NOT NULL COMMENT '锁定库存',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

/*Data for the table `inventory` */

insert  into `inventory`(`id`,`product_id`,`total_inventory`,`lock_inventory`) values

(1,'1',1000,0);


CREATE DATABASE /*!32312 IF NOT EXISTS*/`tcc_order` /*!40100 DEFAULT CHARACTER SET utf8mb4 */;

USE `tcc_order`;
DROP TABLE IF EXISTS `order`;

CREATE TABLE `order` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `create_time` datetime NOT NULL,
  `number` varchar(20) COLLATE utf8mb4_bin NOT NULL,
  `status` tinyint(4) NOT NULL,
  `product_id` varchar(128) NOT NULL,
  `total_amount` decimal(10,0) NOT NULL,
  `count` int(4) NOT NULL,
  `user_id` varchar(128) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

```



数据库创建结构图

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210530091954.png)





###### Hmily的TCC实际使用

```

-- service层TCC的具体实现

@Hmily(confirmMethod = "confirmOrderStatus", cancelMethod = "cancelOrderStatus")
public void makePayment(Order order) {
	注：跨库的远程feign调用接口上也需要加上@Hmily注解(生产者和消费者都需要加)
	...
}


public void confirmOrderStatus(Order order) {
	...
}


public void cancelOrderStatus(Order order) {
	... 
}

```




