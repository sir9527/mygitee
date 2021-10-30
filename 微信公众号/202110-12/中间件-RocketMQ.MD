

[TOC]




#### 1.MQ简述

**消息队列**是一种“先进先出”的数据结构



```

	MQ（消息队列）应用：
	1.异步处理（比异步线程更快，不需要等待结果）
      场景说明：用户注册后，需要发注册邮件和注册短信
   
    2.解耦：订单系统发消息，库存系统接收消息之后进行处理
      场景说明：用户下单后,订单系统需要通知库存系统
   
    3.流量控制（流量削峰）：根据秒杀系统能力处理消息
      场景说明：秒杀活动，流量过大，消息队列控制流量人数

```



 **MQ间比较**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210606151354.png)



#### 2.win环境安装RocketMQ

##### 2.1 配置rocketmq的环境

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210520225753.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210520225827.png)





##### 2.2 启动NameServer( .\mqnamesrv )(相当于注册中心)

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210520231307.png)



```

.\mqnamesrv

```



##### 2.3 启动Broker( .\mqbroker -n 127.0.0.1:9876 )(连接注册中心)

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210520231322.png)



```

.\mqbroker -n 127.0.0.1:9876

```



##### 2.4 启动可视化界面
 ![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210520232000.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210520232012.png)

```

http://127.0.0.1:18080

```



#### 3.RocketMQ集群搭建

##### 3.1 RocketMQ中组件

* Producer：消息的发送者；举例：发信者
* Consumer：消息接收者；举例：收信者
* Broker：暂存和传输消息；举例：邮局
* NameServer：管理Broker；举例：各个邮局的管理机构
* Topic：区分消息的种类；一个发送者可以发送消息给一个或者多个Topic；一个消息的接收者可以订阅一个或者多个Topic消息
* Message Queue：相当于是Topic的分区；用于并行发送和接收消息



**RocketMQ集群**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210606151918.png)



```

1.NameServer是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。

2.Broker部署相对复杂，Broker分为Master与Slave，一个Master可以对应多个Slave，但是一个Slave只能对应一个Master，Master与Slave的对应关系通过指定相同的BrokerName，不同的BrokerId来定义，BrokerId为0表示Master，非0表示Slave。Master也可以部署多个。每个Broker与NameServer集群中的所有节点建立长连接，定时注册Topic信息到所有NameServer。

3.Producer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer取Topic路由信息，并向提供Topic服务的Master建立长连接，且定时向Master发送心跳。Producer完全无状态，可集群部署。

4.Consumer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer取Topic路由信息，并向提供Topic服务的Master、Slave建立长连接，且定时向Master、Slave发送心跳。Consumer既可以从Master订阅消息，也可以从Slave订阅消息，订阅规则由Broker配置决定。

```



##### 3.2 集群模式

###### 3.2.1 单Master模式

```

roker重启或者宕机时，会导致整个服务不可用

```



###### 3.2.2 多Master模式

```

1.异步刷盘丢失少量消息，同步刷盘一条不丢
2.单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会受到影响

```



###### 3.2.3 多Master多Slave模式（异步）

```

1.每个Master配置一个Slave，有多对Master-Slave
2.异步复制方式，主备有短暂消息延迟（毫秒级）
3.Master宕机，磁盘损坏情况下会丢失少量消息。

```



###### 3.2.4 多Master多Slave模式（同步）

```

1.每个Master配置一个Slave，有多对Master-Slave
2.同步双写方式，即只有主备都写成功，才向应用返回成功，不丢数据
3.性能比异步复制模式略低（大约低10%左右）

```



##### 3.3 搭建双主双从（同步）

###### 3.3.1 架构图

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210606153202.png)



```

1. 启动NameServer，NameServer起来后监听端口，等待Broker、Producer、Consumer连上来，相当于一个路由控制中心。

2. Broker启动，跟所有的NameServer保持长连接，定时发送心跳包。心跳包中包含当前Broker信息(IP+端口等)以及存储所有Topic信息。注册成功后，NameServer集群中就有Topic跟Broker的映射关系。

3. 收发消息前，先创建Topic，创建Topic时需要指定该Topic要存储在哪些Broker上，也可以在发送消息时自动创建Topic。

4. Producer发送消息，启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取当前发送的Topic存在哪些Broker上，轮询从队列列表中选择一个队列，然后与队列所在的Broker建立长连接从而向Broker发消息。

5. Consumer跟Producer类似，跟其中一台NameServer建立长连接，获取当前订阅Topic存在哪些Broker上，然后直接跟Broker建立连接通道，开始消费消息

```



###### 3.3.2 默认端口号

RocketMQ默认使用3个端口：9876 、10911 、11011

* `nameserver` 默认使用 9876 端口
* `master` 默认使用 10911 端口
* `slave` 默认使用11011 端口



###### 3.3.3 修改配置文件

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210606153418.png)



**`master1、master2、slave1、slave2`**配置如下：

**master1**

```
vi /usr/soft/rocketmq/conf/2m-2s-sync/broker-a.properties
```

```

#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master，>0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128

```



**slave1**

```
vi /usr/soft/rocketmq/conf/2m-2s-sync/broker-a-s.properties
```

```
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master，>0 表示 Slave
brokerId=1
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=11011
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128

```



**master2**

```
vi /usr/soft/rocketmq/conf/2m-2s-sync/broker-b.properties
```

```

#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示 Master，>0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128

```



**slave2**

```
vi /usr/soft/rocketmq/conf/2m-2s-sync/broker-b-s.properties
```

```

#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示 Master，>0 表示 Slave
brokerId=1
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=11011
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128

```



##### 3.4 Rocket消息类型

###### 3.4.1 消息发送

- 发送同步消息：可靠性同步地发送方式使用的比较广泛，比如：重要的消息通知，短信通知。
- 发送异步消息：对响应时间敏感的业务场景，即发送端不能容忍长时间地等待Broker的响应。
- 单向发送消息：不特别关心发送结果的场景，例如日志发送。

###### 3.4.2 消息消费

- 负载均衡模式：消费者采用负载均衡方式消费消息，多个消费者共同消费队列消息，每个消费者处理的消息不同
- 广播模式：消费者采用广播的方式消费消息，每个消费者消费的消息都是相同的



```

消息的高级特性使用：顺序消息,延时消息,批量消息,过滤消息,事务消息

1.顺序消息
RocketMQ可以严格的保证消息有序，可以分为 分区有序 或者 全局有序。

- 当发送和消费参与的queue只有一个，则是全局有序(一般没有必要)；
- 如果多个queue参与，则为分区有序，即相对每个queue，消息都是有序的(同一个订单id放入一个队列)。

2.延时消息

- 提交了一个订单就可以发送一个延时消息，1h后去检查这个订单的状态，如果还是未付款就取消订单释放库存
- 延时发送时间设置

注：延时消息的等级 "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";

3.批量消息
4.过滤消息：通过消费者订阅特定标签进行消费
5.事务消息：正常事务消息的发送及提交、事务消息的补偿流程
  注：事务消息不支持延时消息和批量消息。
  
```



**事务消息**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210606161836.png)

```

1.事务消息发送及提交
(1) 发送消息（half消息）。
(2) 服务端响应消息写入结果。
(3) 根据发送结果执行本地事务（如果写入失败，此时half消息对业务不可见，本地逻辑不执行）。
(4) 根据本地事务状态执行Commit或者Rollback（Commit操作生成消息索引，消息对消费者可见）


2.事务补偿
(1) 对没有Commit/Rollback的事务消息（pending状态的消息），从服务端发起一次“回查”
(2) Producer收到回查消息，检查回查消息对应的本地事务的状态
(3) 根据本地事务状态，重新Commit或者Rollback
其中，补偿阶段用于解决消息Commit或者Rollback发生超时或者失败的情况。


3.事务消息状态
事务消息共有三种状态，提交状态、回滚状态、中间状态：
- TransactionStatus.CommitTransaction: 提交事务，它允许消费者消费此消息。
- TransactionStatus.RollbackTransaction: 回滚事务，它代表该消息将被删除，不允许被消费。
- TransactionStatus.Unknown: 中间状态，它代表需要检查消息队列来确定状态。

```



#### 4.RocketMQ实现分布式事务



#### 5.RocketMQ高级特性

##### 5.1 消息存储

分布式队列因为有高可靠性的要求，所以数据要进行持久化存储。

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210607102839.png)



```

存储介质
1.关系型数据库DB。例如：mysql
2.文件系统
目前业界较为常用的几款产品（RocketMQ/Kafka/RabbitMQ）均采用的是消息刷盘至所部署虚拟机/物理机的文件系统来做持久化（刷盘一般可以分为异步刷盘和同步刷盘两种模式）。消息刷盘为消息存储提供了一种高效率、高可靠性和高性能的数据持久化方式。除非部署MQ机器本身或是本地磁盘挂了，否则一般是不会出现无法持久化的故障问题。
注：推荐文件系统

```



###### 5.1.1 消息存储和发送

- 消息存储（提高消息存盘）
  高性能磁盘，顺序写速度可以达到600MB/s，但是磁盘随机写的速度只有大概100KB/s，和顺序写的性能相差6000倍！RocketMQ的消息用顺序写,保证了消息存储的速度。

- 消息发送（提高网络发送）
  Linux操作系统分为【用户态】和【内核态】
  **内核态**：cpu可以访问内存的所有数据，包括外围设备，例如硬盘，网卡等
  **用户态**：应用程序。当需要做一些内核态的事情, 例如从硬盘读取数据, 或者从键盘获取输入等. 这时用户态因为权限不够而要切换到内核态。

 



**RocketMQ的零拷贝技术**

一台服务器把本机磁盘文件的内容发送到客户端，分为两个步骤：
1.read：读取本地文件内容 ;
2:write：将读取的内容通过网络发送出去。



这两个看似简单的操作，实际进行了4 次数据复制，分别是：
1.从磁盘复制数据到内核态内存；
2.从内核态内存复 制到用户态内存；
3.然后从用户态 内存复制到网络驱动的内核态内存；
4.最后是从网络驱动的内核态内存复 制到网卡中进行传输



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210607105400.png)



通过使用mmap（内存共享技术）的方式，可以省去向用户态的内存复制，提高速度。这就是"**零拷贝技术**"。



###### 5.1.2 刷盘机制

RocketMQ的消息是存储到磁盘上的，这样既能保证断电后恢复， 又可以让存储的消息量超出内存的限制。RocketMQ为了提高性能，会尽可能地保证磁盘的顺序写。消息在通过Producer写入RocketMQ的时 候，有两种写磁盘方式，分布式 **同步刷盘** 和 **异步刷盘**

- 同步刷盘
  在返回写成功状态时，消息已经被写入磁盘。具体流程是，消息写入内存的PAGECACHE后，立刻通知刷盘线程刷盘， 然后等待刷盘完成，刷盘线程执行完成后唤醒等待的线程，返回消息写成功的状态。

- 异步刷盘
  在返回写成功状态时，消息可能只是被写入了内存的PAGECACHE，写操作的返回快，吞吐量大；当内存里的消息量积累到一定程度时，统一触发写磁盘动作，快速写入  



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210607110210.png)





##### 5.2 高可用机制

RocketMQ分布式集群是通过Master和Slave的配合达到高可用性的。

Master角色的Broker支持读和写，Slave角色的Broker仅支持读。也就是 Producer只能和Master角色的Broker连接写入消息；Consumer可以连接 Master角色的Broker，也可以连接Slave角色的Broker来读取消息。



**消息主从复制**
如果一个Broker组有Master和Slave，消息需要从Master复制到Slave 上，有同步和异步两种复制方式

- 同步复制：同步复制方式是等Master和Slave均写 成功后才反馈给客户端写成功状态；
- 异步复制：异步复制方式是只要Master写成功 即可反馈给客户端写成功状态。



##### 5.3 死信队列

一条消息初次消费失败，消息队列 RocketMQ 会自动进行消息重试；达到最大重试次数后，若消费依然失败，消息会被放入死信队列。



##### 5.4 消费幂等

消息队列 RocketMQ 消费者在接收到消息以后，有必要根据业务上的唯一 Key 对消息做幂等处理的必要性



**生产方**

```
Message message = new Message();
message.setKey("ORDERID_100");
SendResult sendResult = producer.send(message);
```



**消费方**

````
consumer.subscribe("ons_test", "*", new MessageListener() {
    public Action consume(Message message, ConsumeContext context) {
        String key = message.getKey()
        // 根据业务唯一标识的 key 做幂等处理
    }
});
````













