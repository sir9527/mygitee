







[TOC]



# 不停机扩容数据库
## 1.初始化状态

刚开始用户表根据用户id取模存储到2张表中

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210621114154.png)





## 2.新增数据库

新增原来2倍的数据库，新增的数据库先做一个数据库同步(例如可以借助中间件canal)

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210621114338.png)





## 3.扩容数据库

数据同步完成后，删除同步配置，重启并清除数据库多余的数据

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210621114608.png)



```java

// 例如用户id：0 1 2 3 4 5 6 7 8 9 10
// 1.初始化2个数据库
// 桶1：0 2 4 6 8 10    id%2 = 0
// 桶2：1 3 5 7 9       id%2 = 1

// 2.扩容同步数据后为4个数据库 需要清除部分数据
// 0 2 4 6 8 10
// 桶1：0 2 4 6 8 10      2 6 10     去掉id%4 = 2
// 桶2：0 2 4 6 8 10      0 4 8      去掉id%4 = 0

// 1 3 5 7 9
// 桶3：1 3 5 7 9         1 5 9      去掉id%4 = 3
// 桶4：1 3 5 7 9         3  7       去掉id%4 = 1

```

















