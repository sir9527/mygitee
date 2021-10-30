

#### Sharding - JDBC

[TOC]

#### 1.ShardingSphere的三部曲

```

	ShardingSphere的三部曲
	1.ShardingSphere - JDBC  ( 可理解为增强版的 JDBC 驱动 )
	2.ShardingSphere - Proxy( sharding-proxy 相当于 mycat，单独一个服务)
	3.ShardingSphere - Sidecar(TODO)
	
```



##### 1.1 ShardingSphere - JDBC

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405151509.png)



##### 1.2 ShardingSphere - Proxy

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405151726.png)





#### 2.搭建MySQL读写分离

```

	## 1.配置mysql扩展源
	rpm -ivh http://repo.mysql.com/yum/mysql-5.7-community/el/7/x86_64/mysql57-community-release-el7-10.noarch.rpm
	
	## 2.yum安装mysql
	yum install mysql-community-server -y
	
	## 3.启动mysql，并加入开机自启
	systemctl start mysqld
	systemctl stop mysqld
	systemctl enable mysqld      -- 设置mysql服务开机自启动
	systemctl status mysqld      -- 查看mysql服务启动状态

	## 4.使用msql初始密码登录数据库
	mysql -uroot -p$(awk '/temporary password/{print $NF}' /var/log/mysqld.log)
	
	## 5.修改密码
	-- 降低mysql设置密码的难度(不更改密码需要包含大小写等字符)
	set global validate_password_policy=0;
	set global validate_password_length=1;
	-- 修改密码
	ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
	-- 退出mysql服务
	quit;                          
	-- 命令行到登录mysql
	mysql -uroot -proot           
	-- 授权远程登录
	grant all on *.* to root@'%' identified by 'root';
	-- 涮新
	flush privileges;
	
	## 6.本地远程连接云服务器
	## 本地远程连接linux服务器

	注：
	  主从复制 --> 从数据库层面说的
	  读写分离 --> 代码业务层面说的
	
```



##### 2.1 Master节点修改配置(master节点的 "/etc/my.cnf")

**①需改配置文件保存二进制文件**

```

	>vim /etc/my.cnf

	## 找到"[mysqld]"下面复制下面的配置

	[mysqlId]
	## 唯一id
	server-id=100
	## 开启二进制日志功能，名字可以随便取
	log-bin=mysql-bin
	## 复制过滤：不需要备份的数据库（mysql的mysql库一般不需要同步）
	binlog-ignore-db=mysql
	## 下面的这些配置可以不需要写
	## 为每个session分配的内存，在事务过程中用来存储二进制日志的缓存
	binlog_cache_size=1M
	## 主从复制的格式（mixed，statement,row,默认格式是statement）
	binlog_format=mixed

```

注：配置完需要重启



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405153318.png)



**②mater服务器给slave服务器授权**

```

	mysql -uroot -proot
	## 授予slave服务器可以同步master服务
	grant replication slave,replication client on *.* to 'root'@'slave服务的ip' identified by 'slave服务器的密码'; 
	## 刷新
	flush privileges;

	## 查看mysql现在有哪些用户及对应的ip权限（可以不执行，只是一个查看）
	select user,host from mysql.user;

```



**③查询master服务器的binlog文件名和位置**

```

	show master status;

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405154233.png)



##### 2.2 Slave节点修改配置(slave节点的 "/etc/my.cnf")

**①需改配置文件保存二进制文件**

```

	>vim /etc/my.cnf

	## 找到"[mysqld]"下面复制下面的配置
	
	[mysqlId]
	## 唯一id
	server-id=102
	## 开启二进制日志功能，以备slave作为其他slave的master时使用
	log-bin=mysql-slave-bin
	## relay_log配置中继日志
	relay_log=edu-mysql-relay-bin
	## 复制过滤：不需要备份的数据库（mysql的mysql库一般不需要同步）
	binlog-ignore-db=mysql
	## 如果需要同步函数或者存储过程
	log_bin_trust_function_creators=true
	## 为每个session分配的内存，在事务过程中用来存储二进制日志的缓存
	binlog_cache_size=1M
	## 主从复制的格式（mixed，statement,row,默认格式是statement）
	binlog_format=mixed
	## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断
	## 如：1062错误是指一些主键重复。1032错误是因为主从数据库数据不一致
	slave_skip_errors=1062

```

注：配置完需要重启



**②slave服务器进行关联master服务器**

```

	mysql>mysql -uroot -proot

	-- "master_log_file"和"master_log_pos" 在master机上"show master status;"
	change master to master_host='master服务器ip',master_user='root',master_password='master密码',master_port=3306,master_log_file='master查到的',master_log_pos=master查到的;

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405154413.png)



**③在slave节点查看主从同步状态**

```

	## 启动主从复制
	start slave;
	## 再查看主从同步状态
	show slave status\G;

	## 停止复制
	stop slave;

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405154854.png)



##### 2.3 主从复制报错

**①从服务器没有同步主服务器的数据**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210410094427.png)



**②输入主服务器的密码错误**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210410095805.png)



```

	## 原因1：复制位置发生改变（这种可能性比较大）

	## 操作从服务器
	## 停止复制
	stop slave;
	## "master_log_file"和"master_log_pos" 在master机上"show master status;"
	change master to master_host='master服务器ip',master_user='root',master_password='master密码',master_port=3306,master_log_file='master查到的',master_log_pos=master查到的;
	## 启动主从复制
	start slave;

```



```

	## 原因2：更改服务器了
	
	## 操作从服务器
	## 停止复制
	stop slave;
	set global sql_slave_skip_counter=1;
	## 启动主从复制
	start slave;
	## 再查看主从同步状态
	show slave status\G;

```



##### 2.3 读写分离,主从复制(1主2从)

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405152423.png)



**主从复制原理（数据层面的东西）**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405152624.png)



```

主从复制流程
1.在备库 B 上通过 change master 命令，设置主库 A 的 IP、端口、用户名、密码，以及要从哪个位置开始请求 binlog，这个位置包含文件名和日志偏移量。

2.在备库 B 上执行 start slave 命令，这时候备库会启动两个线程，就是图中的 io_thread 和 sql_thread。其中 io_thread 负责与主库建立连接。

3.主库 A 校验完用户名、密码后，开始按照备库 B 传过来的位置，从本地读取 binlog，发给 B。

4.备库 B 拿到 binlog 后，写到本地文件，称为中转日志（relay log）。

5.sql_thread 读取中转日志，解析出日志里的命令，并执行

```







**读写分离（shardingJDBC：业务层面的东西）**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405152423.png)





#### 3.分库分表



**水平分表,垂直分表**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210405161523.png)







#### 4.读写分离 分库分表 代码实战

##### 4.1 读写分离

**mysql的主从两个服务器**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210411215917.png)





**读只会使用2台从服务器("http://localhost:8085/user/listuser")**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210411205415.png)



**写只会在主服务器("http://localhost:8085/user/save")**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210411205748.png)





##### 4.3 分库分表



**分库分表查询语句会查询所有库和所有表的数据("http://localhost:8085/user/listuser")**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210411215635.png)





**分库分表带来的问题**

```

	分库分表后，本地事务已经不能保证事务，需要分布式事务
	
	## 本地事务
	@Transactional（rollbackFor = Exception.class）

	## Sharding提供的分布式事务
	## 扣钱-->强一致性 ； 订单 --> 弱一致性，柔性
	@Transactional（rollbackFor = Exception.class）
	@ShardingTransactionType(TransactionType.XA)

```



