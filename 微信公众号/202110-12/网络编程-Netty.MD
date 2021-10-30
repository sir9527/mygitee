



[TOC]



### Netty



**1.Netty简介**

```

	Netty(Java编写)
	1.异步框架（NIO框架）
	2.主要针对TCP协议
    3.协议：TCP/IP --> JDK原生 --> NIO --> Netty
    4.RPC框架（例如Dubbo）使用Netty通信
    
```



```

	Netty可以做的事情：
	  1.可以作为rpc通讯的框架、协议、库，实现远程过程调用，基于socket的方式,例如Dubbo等
	  2.可以作为长链接实现客户端与服务器的通讯，基于websocket,例如QQ 微信等
	  3.作为http服务器，类似于jetty、tomcat等等。
	    作为http服务器时，netty所采取的编程模型并不是遵循servlet的规范，而是自实现

```



**2.BIO NIO AIO**

```

	I/O模型
	1.I/O模型：数据发送和接受的通道
	2.JAVA支持3种网络编程模型I/O模型：BIO,NIO,AIO
	  2.1 BIO：同步并阻塞。一个连接一个线程
	  2.2 NIO：同步非阻塞。一个线程处理多个连接。和IO多路复用配合使用。
	  2.3 AIO：异步非阻塞。使用还未推广开
	  
```



```

	win系统连接客户端
	1.cmd调出dos窗口
	2.telnet 127.0.0.1 7000
	3.ctrl + ]
	4.send helloworld
	
```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/9bf6eac3fc2693e4680e68e2a67e608.png)



**BIO模型图**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210331231520.png)



**NIO模型图**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210403085634.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210403092410.png)



```
	
	NIO
	1.Channel（通道）：相当于BIO的Socket
	2.Buffer（缓冲区）
	3.Selector（选择器）
	
	Selector只有在 连接/通道 真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，
	并且不必为每个连接都创建一个线程，不用去维护多个线程。
	避免了多线程之间的上下文切换导致的开销

	Selector会和多个Channel关联。选择器监听通道的动作，从而达到非阻塞。
	NIO可以实现单个线程监听多个客户端通道。

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/fb14862d810f533167061f45da89eac.png)



**Buffer缓冲区的作用**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/7f3a7639a215cb24a0f52c0001d6fdd.png)



**Channel通道（类似流）**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/b8192d09e7b1dc9763aece1970c80c1.png)



**3.TCP拆包粘包问题(Netty)**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210404112248.png)



```

	假设客户端分别发送了两个数据包D1和D2给服务端，由于服务端一次读取到字节数是不确定的，
	故可能存在以下四种情况：
	1.服务端分两次读取到了两个独立的数据包，分别是D1和D2，没有粘包和拆包
	2.服务端一次接受到了两个数据包，D1和D2粘合在一起，称之为TCP粘包
	3.服务端分两次读取到了数据包，
	  第一次读取到了完整的D1包和D2包的部分内容，
	  第二次读取到了D2包的剩余内容，这称之为TCP拆包
	4.服务端分两次读取到了数据包，
	  第一次读取到了D1包的部分内容D1_1，
	  第二次读取到了D1包的剩余部分内容D1_2和完整的D2包。


	Netty解决TCP拆包粘包问题:
	  使用自定义协议 + 编解码器 来解决
	  (注:关键就是要解决 服务器端每次读取数据长度的问题, 从而避免的TCP粘包、拆包 )
	  
```



**4.RPC远程调用(Netty可以实现)**

```

	RPC远程调用示例：
	  1.阿里的Dubbo(底层是Netty)
	  2.Spring 旗下的 Spring Cloud(Fegin)
	  ...

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210404154827.png)





















