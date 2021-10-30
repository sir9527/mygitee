



[TOC]



##### 错误结论：一台服务器最多连接65535个TCP连接，因为端口号不够用了。端口号范围0~65535。



##### 1.端口号限制

一直连同一个目标IP和同一个目标端口，最多连接 63977 个，因为你自己的端口号不够用了。
换一个目标IP或者目标端口可以继续连接服务器

```

[root]# cat /proc/sys/net/ipv4/ip_local_port_range 
1024 65000

```

​        

**Socket编程四元组**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210605130523.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210605131353.png)





##### 2.文件描述符

每建立一个TCP连接，就得分配给你一个文件描述符（默认从1-100000）

1台linux客户端连服务端后，TCP连接会返回给客户端一个 '文件描述符'

```

系统级：当前系统可打开的最大数量，通过 cat /proc/sys/fs/file-max 查看
用户级：指定用户可打开的最大数量，通过 cat /etc/security/limits.conf 查看
进程级：单个进程可打开的最大数量，通过 cat /proc/sys/fs/nr_open 查看

[root ~]# cat /proc/sys/fs/file-max
100000
[root ~]# cat /proc/sys/fs/nr_open
100000
[root ~]# cat /etc/security/limits.conf
...
* soft nproc 100000
* hard nproc 100000

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210605131447.png)





##### 3.线程

IO阻塞：每建一个TCP连接都需要消耗一个线程来为你服务，因为上下文切换，TCP建立太多线程上下文切换慢。

IO多路复用： 一个线程可以管理多个 TCP 连接的资源，可以用少量的线程来管理大量的 TCP 连接了 



**IO阻塞**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210605131743.png)



**IO多路复用**

[略]



##### 4.内存

TCP连接本身，以及这个连接所用到的缓冲区，都是需要占用一定内存的，连接太多TCP会内存溢出 



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210605131931.png)





##### 5.CPU

CPU是处理线程的能力。当TCP太多，线程也多，CPU 的占用率已经快到 100% 



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210605132029.png)





##### 6.小结

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210605132401.png)







