

​      

[TOC]



### TCP/IP四层模型



​      **TCP/IP四层模型**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210603103032.png)



​      **四层模型间关系**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210603102950.png)



#### 1.物理层、链路层、网络层

**物理层、链路层、网络层**：通过IP（可以查到对应的MAC地址）将数据从一个主机搬到另一台主机。



##### 1.1 物理层之集线器（或网线）

**3太电脑用网线连接**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210602192730.png)



**集线器连接电脑**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210602192750.png)



**集线器**，它仅仅是无脑将电信号**转发到所有出口（广播）**，不做任何处理，你觉得它是没有智商的，因此把人家定性在了**物理层**。 



注：每台电脑都有一个唯一标识，**MAC地址** 

注：当A要给B发消息，会给集线器所有成员发，其他成员通过判断数据头是不是给自己的MAC地址进行丢弃或者接受。

注：缺点就是A只给B发消息，其他成员也要接受并判断消息。这样既不安全，又不节省网络资源。 



**A给B发消息**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210602192819.png)





##### 1.2 数据链路层之交换机（mac地址）

如果把这个集线器弄得更智能一些，**只发给目标 MAC 地址指向的那台电脑**，看起来似乎有智能了，你把这东西叫做**交换机**。也正因为这一点点智能，你把它放在了另一个层级，**数据链路层**。



**交换机**

![1622633320708](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\1622633320708.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210602193315.png)





 交换机内部维护一张 **MAC 地址表**，记录着每一个 MAC 地址的设备，连接在其哪一个端口上 

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210602192956.png)



 你给这个通过这样传输方式而组成的小范围的网络，叫做**以太网**。 



##### 1.3 网络层之路由器（IP地址）

**路由器，**它的功能就是，作为一台独立的拥有 MAC 地址的设备，并且可以帮我把数据包做一次转发**，**你把它定在了**网络层。** 



MAC地址也叫物理地址、硬件地址， 长度为 48 位 ， 是网络设备制造商生产时烧录在网卡的EPROM（一种闪存芯片，通常可以通过程序擦写）。MAC地址就如同身份证上的身份证号码，具有唯一性 。



**IP 地址**：现在每一台电脑，同时有自己的 MAC 地址，又有自己的 IP 地址，只不过 IP 地址是**软件层面**上的，可以随时修改，MAC是物理地址一般是无法修改的 



如果源 IP 与目的 IP 处于一个子网，直接将包通过交换机发出去；

如果源 IP 与目的 IP 不处于一个子网，就交给路由器去处理 



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210602194951.png)





例1：A向B发送数据。它们直接连着交换机，所以 A 直接通过mac地址借助交换机发出如下数据包即可，其实网络层（路由器）没有体现出作用。

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210602195156.png)



例2：A向C发送数据。A先通过mac地址将包数据发给交换机，交换机通过mac地址找到路由器。路由器通过arp 缓存表找到目标ip的mac地址，然后通过交换机用mac地址将包发给C。由于最底层的传输仍然需要依赖以太网，所以数据包是分成两段的。 

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210602195328.png)





````

1.交换机中有 MAC 地址表用于映射 MAC 地址和它的端口
2.路由器中有路由表用于映射 IP 地址(段)和它的端口
3.电脑和路由器中都有 arp 缓存表用于缓存 IP 和 MAC 地址的映射关系

````



```

小结：网络层（IP协议）本身没有传输包的功能，包的实际传输是委托给数据链路层（以太网中的交换机）来实现的。

```





#### 2.传输层协议的代表之TCP、UDP 

前三层协议（物理层、链路层、网络层） 只能把数据包从一个主机搬到另外一台主机 ，却不知道交给哪个程序（进程），这时候就需要给进程起名字，即端口号放入传输层头部，这就是UDP协议（进程与进程间通信）。



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210603094649.png)



**UDP**的问题：丢包、发包乱序、流量控制、连接问题等



**TCP**解决了UDP的一系列问题，例如3次握手4次挥手解决连接问题。TCP是面向连接、可靠 、基于字节流的传输层通信协议。

````

三次握手
A：我准备好了(SYN)
B：我知道了(ACK)，我也准备好了(SYN)
A：我知道了(ACK)

四次挥手
A：再见，我要关闭了(FIN)
B：我知道了(ACK)...给 B 一段时间把自己的事情处理完...
B：再见，我要关闭了(FIN)
A：我知道了(ACK)

````



**TCP数据包**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210603104616.png)





**Socket编程**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210603103523.png)

**TCP下的Socket编程**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210603103658.png)





