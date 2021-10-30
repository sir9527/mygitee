

[TOC]



# 多线程

注：高并发三大利器：缓存、异步、限流



## 1.线程的状态

- 新建 New
- 就绪 Runnable   start()方法   等待获取CPU的使用权
- 运行 Running     线程获取到了CPU的使用权
- 阻塞 Blocked     sleep()、wait()、join()方法  放弃CPU的使用权，阻塞完后进入就绪状态
- 死亡 Dead



## 2.线程创建方式

3种方式：继承Thread、实现Runnable、实现Callable

```java
    public static String getData(String threadName){
        System.out.println(threadName + ": 123456");
        return "123456";
    }
```

```java
    // 使用new Thread调用多线程
	public static void main(String[] args) {
        for(int i=0;i<5;i++){
            new Thread(() -> {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                getData(Thread.currentThread().getName());
            },Integer.toString((int)(Math.random()*10))).start();
        }
    }
```

```java
	// 使用线程池执行任务   
	public static void main(String[] args) {
        ThreadPoolExecutor executorService = new ThreadPoolExecutor(
                2,      
                5, 
                3,    
                TimeUnit.SECONDS,    
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),     
                new ThreadPoolExecutor.AbortPolicy()  
        );

        try {
            for (int i=0;i<8;i++) {
                executorService.execute(() -> {
                    getData(Thread.currentThread().getName());
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            executorService.shutdown();//关闭线程池
        }
    }
```



## 3.线程池的7个参数

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210609125916.png)

```java

/** 线程池的7个参数：
 *  1.corePoolSize            线程池核心线程大小
 *  2.maximumPoolSize         线程池最大线程数量
 *  3.keepAliveTime           空闲线程存活时间
 *  4.unit                    空闲线程存活时间单位
 *  5.workQueue               工作队列
 *      ArrayBlockingQueue    ：FIFO 先进先出
 *      LinkedBlockingQuene   ：FIFO 先进先出
 *      SynchronousQuene      ：不缓存任务的阻塞队列
 *      PriorityBlockingQueue ：具有优先级的无界阻塞队列
 *  6.threadFactory           线程工厂（创建一个新线程时使用的工厂）
 *  7.handler                 拒绝策略
 */
 
/** 4大拒绝策略：
 * 1.new ThreadPoolExecutor.AbortPolicy();  默认拒绝策略
 *   直接丢弃任务，并抛出异常
 * 2.new ThreadPoolExecutor.DiscardOldestPolicy();
 *   抛弃进入队列最早的那个任务，然后尝试把这次拒绝的任务放入队列
 * 3.new ThreadPoolExecutor.CallerRunsPolicy();
 *   由调用线程处理该任务（一般是main线程处理）
 * 4.new ThreadPoolExecutor.DiscardPolicy();
 *   直接丢弃任务，不抛出异常
 * */
 
//最大线程到底该如何定义
//1. CPU 密集型：几核CPU就是几（例：我的电脑就是4核）
	//4 （获取个人PC的CPU数量）
	System.out.println(Runtime.getRuntime().availableProcessors());
//2. IO  密集型：判断你程序中十分耗 IO的线程

//程序：比如你现在有15个大型任务（IO流很耗资源） ---> 设置最大线程数为30（耗资源程序数量的2被）
ThreadPoolExecutor executorService = new ThreadPoolExecutor(
        2,      //核心线程池数（银行常开窗口数量）
        5,      //最大线程池数（人多时银行才会临时开一些线程池）
        3,      //超时就会释放线程池，使线程池保持在核心线程池数量上
        TimeUnit.SECONDS,    //超时时间单位
        new LinkedBlockingQueue<>(3),//阻塞队列
        Executors.defaultThreadFactory(),      //线程工厂（创建线程的，一般不用动）
        new ThreadPoolExecutor.AbortPolicy()   //拒绝策略（银行候客厅人已经满了，还有人进银行，这时候银行拒绝进入）
); 

```



```java

// 线程池工具类Executors
// 以下是Executors工具类的3个常用线程池方法（）
// 阿里巴巴开发手册不建议使用，因为最大线程池是int范围，即21亿
// 1.单个线程
//ExecutorService executorService = Executors.newSingleThreadExecutor();
// 2.创建一个固定的线程池的大小
/ExecutorService executorService = Executors.newFixedThreadPool(5);    
// 3.可伸缩的 遇强则强，遇弱则弱
//ExecutorService executorService = Executors.newCachedThreadPool();    

```



## 4.线程式怎么通信的

- volatile （vɒ lə taɪl）  内存可见
- 等待/通知机制     wait和notify
- join()
- JUC提供的工具类CountDownLatch（ lætʃ] ）、CyclicBarrier、Semaphore
- ThreadLocal  不同于上面的几个多线程间通信，这个是线程内通信



## 5.JAVA并发编程线程安全3要素

synchronized：1.具有原子性，2.有序性（ 无法禁止指令重排优化），3.可见性 

volatile：1.不能保证原子性，2.有序性（禁止指令重排序优化），3.可见性  



JAVA多线程并发编程线程安全需要保证3要素

- 原子性（**Synchronized, Lock**）

- 有序性（**Volatile，Synchronized, Lock**）

- 可见性（**Volatile，Synchronized,Lock**）

  