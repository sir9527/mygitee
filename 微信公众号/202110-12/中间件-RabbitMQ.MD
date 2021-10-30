#### RabbitMQ



[TOC]



#### 1.RabbitMQ简介

```
	## bilibili视频链接：https://www.bilibili.com/video/BV1dX4y1V73G

	## rabbitmq官网：https://rabbitmq.com

	## rabbitmq配置：https://rabbitmq.com/configure.html
```



```


	MQ（消息队列）应用：
	1.异步处理（比异步线程更快，不需要等待结果）
      场景说明：用户注册后，需要发注册邮件和注册短信
   
    2.解耦：订单系统发消息，库存系统接收消息之后进行处理
     场景说明：用户下单后,订单系统需要通知库存系统
   
    3.流量控制（流量削峰）：根据秒杀系统能力处理消息
      场景说明：秒杀活动，流量过大，消息队列控制流量人数

```




##### 2.RabbitMQ的安装(Linux安装) 
```
	## 1.新建目录文件rabbitmq并上传erlang和rabbitmq的rpm
	mkdir -p /opt/rabbitmq
	cd /opt/rabbitmq
	
	## 2.安装erlang（rabbitmq是erlang语言写的，就像打开java代码需要jdk环境）
	rpm -Uvh erlang-solutions-2.0-1.noarch.rpm
	# 安装
	yum install -y erlang
	# 查看版本号
	erl -v
	
	## 3.安装rabbitmq
	# rabbitmq需要socat
	yum install -y socat
	rpm -Uvh rabbitmq-server-3.8.13-1.el8.noarch.rpm
	# 安装
	yum install rabbitmq-server -y
	# 启动
	systemctl start rabbitmq-server
	# 查看状态
	systemctl status rabbitmq-server
	# 开机自启动
	systemctl enable rabbitmq-server
	# 停止rabbitmq服务
	systemctl stop rabbitmq-server
	
	## 4.rabbitmq开启图形化界面
	rabbitmq-plugins enable rabbitmq_management
	# 添加用户admin/admin
	rabbitmqctl add_user admin admin
	# 给用户admin赋予权限级别超级管理员administrator
	rabbitmqctl set_user_tags admin administrator
	# 给用户设置权限 linux命令 或者 客户端去设置
	# 远程进入管理界面：http://主机IP:15672/     admin/admin
	
```

注：docker安装视频狂神的视频里也有说到，我懒得弄



##### 本机访问远端RabbitMQ图形化界面

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405124347.png)



#### 3.RabbitMQ的5种模型

##### 3.1  直连（Direct）（1个消费者消费一个生产者）

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405124725.png)注：生产者、队列、消费者



##### 3.2  任务模式（work quene）（多个消费者消费一个生产者）（1条消息不能多次消费）

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405124853.png)

注：生产者、队列、消费者



##### 3.3  广播模型（fanout）（交换机将消息推给绑定的队列）（1条消息可以多次消费）

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405124933.png)

注：生产者、交换机（X）、队列、消费者



##### 3.4  路由模型（Routing）

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210421094841.png)



注：生产者、交换机（X）、路由、队列、消费者



##### 3.5  订阅模型（Topic）

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405125041.png)

注：生产者、交换机（X）、路由、队列、消费者



```

	Fanout、Routing、Topic模型区别：
	  1.在Fanout模式中，一条消息，会被所有订阅的队列都消费
	  2.在Routing模式中，不同的消息被不同的队列消费。推送消息的时候给队列个唯一Routingkey
	  3.在Topic模式中，不同的消息被不同的队列消费。推送消息的时候给队列个可匹配多个的Routingkey

```



#### 4.RabbitMQ的运行流程

![](https://gitee.com/domineering_red_tide/image/raw/master/image/2af98dc9a25928f029464091e65a2d0.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/006f143287a74e4c088bcdb10adfbc1.png)



#### 5.死信队列

```
	
  什么样的消息会进入死信队列？
	1.消息被拒绝
	2.消息过期
	3.队列达到最大长度

```



#### 6.最终一致性分布式事务之RabbitMQ

```

分布式事务小结:
https://gitee.com/domineering_red_tide/weixin_technology/blob/master/weixin/src/main/java/com/baidu/weixin/_%E4%BA%8B%E5%8A%A1/_%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1.MD	

```



订单服务模块 和 配送中心模块

```

	1.配送中心模块
	https://gitee.com/domineering_red_tide/rabbitmq_distributed

```



```

	2.订单服务模块
	https://gitee.com/domineering_red_tide/rabbitmq_distributed

```



##### 6.1 存在分布式事务的问题

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405095509.png)



##### 6.2 "可靠生产问题 + 可靠消费问题"解决分布式事务问题

**借助MQ解决分布式事务**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405104431.png)



**可靠生产问题**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405105256.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405104717.png)



```

	MQ的publisher/Confirm机制 + 冗余表 + 定时任务
	
```



**可靠消费问题**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405114342.png)



```

	手动ack + try/catch+ + 死信队列处理 + 人工干预

```



#### 7.写在最后的话

```

	建议：
	1.尽量去避免分布式事务
	2.尽量将非核心业务做成异步

```





