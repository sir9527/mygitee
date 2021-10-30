



[TOC]





## 0.JVM基础知识

编译成class文件 -> 类加载器 -> 运行时数据区

类加载器：双亲委派机制（引导类加载器、扩展类加载器、系统类加载器）。从上到下加载。

运行时数据区：栈、堆、方法区

堆内存：新生代（伊甸园:from:to = 8:1:1） + 老年代 = 1: 2。

​                新生代minor GC15次会进入老年代。大对象会直接进入老年代。

垃圾回收算法：新生代from/to区（复制算法）、老年代（标记清除、标记整理）





## 1.JVM基础命令

- jps -l 查看虚拟机进程（**重要**）
- jstat 查看虚拟机的某个进程的情况。可以预测是否会OOM（**重要**）
- jinfo 实时查看或者修改JVM配置参数
- jmap 手动dump堆快照。实际生产配置JVM的参数自动转存OOM内存文件（**重要**）
- jhat dump文件分析工具
- jstack JVM线程快照（**重要**）
- jcmd 上面命令的一个总和命令
- jstatd 远程主机信息收集



## 2.死锁怎么看

- 1.jps -l 查看进程号

- 2.jstack -l 进程号





## 3.服务器cpu过载

- 1.TOP 查看cpu占比高的进程
- 2.ps -mp pid -o THREAD,tid,time  查看进程中线程占比高的线程号
- 3.将占比高的线程号TID转成16进制    Printf “%x\n” number
- 4.jstack 进程号 > jstack.txt   导出线程快照  匹配CPU占比高的线程信息分析（一般到这里就结束了）
- 5.觉得可能是GC导致，可是 jstat -gcutil 10206 2000 50 查看虚拟机gc情况
- 6.如果确定某些对象不能回收导致频繁GC占用CPU，则  jmap -dump:file=du.dump 13339 手动dump堆快照
- 7.把导出的du.dump用JProfiler打开（需要把dump后缀改成hprof才能用JProfiler打开）
- 8.注：可以对比异常和正常的dump文件。大对象占比小，可以考虑是不是重复小对象多，即ThreadLocal内存泄漏引起的。

参考文章：https://www.it610.com/article/1283049019639087104.htm