

[TOC]


* 异步编程CompletableFuture
    * 1.回顾Future
    * 2.CompletableFuture快速入门
    * 3.CompletableFuture的使用场景
        * 3.1 创建异步任务的2种方式
            * 3.1.1 supplyAsync：执行CompletableFuture任务，支持返回值
            * 3.1.2 runAsync：执行CompletableFuture任务，没有返回值
            * 3.1.3 supplyAsync和runAsync的使用
        * 3.2 简单任务异步回调
            * 3.2.1 thenRun/thenRunAsync
            * 3.2.2 thenAccept/thenAcceptAsync
            * 3.2.3 thenApply/thenApplyAsync
            * 3.2.4 异常处理：handle、whenComplete、exceptionally
        * 3.3 多个任务组合处理
            * 3.3.1 AND组合关系
            * 3.3.2 OR组合关系
            * 3.3.3 AllOf 和 AnyOf
            * 3.3.4 ThenCompose
    * 4.CompletableFuture使用有哪些注意点
        * 4.1 Future需要获取返回值，才能获取异常信息
        * 4.2 CompletableFuture的get()方法是阻塞的
        * 4.3 默认线程池的注意点（自定义线程池时，注意饱和策略）
    参考文档：https://mp.weixin.qq.com/s/nLMFS66kKSJ9NEjIay8b2w


# 异步编程CompletableFuture



```java
    // 接口1：查询用户基本信息
    public static String getUserInfo() {
        try {
            Thread.sleep(300); // 模拟调用耗时
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "用户信息：姓名 法外狂徒张三；性别 男"; // 一般是查数据库，或者远程调用返回的
    }


    // 接口2：查询用户勋章信息
    public static String getMedalInfo() {
        try {
            Thread.sleep(500); //模拟调用耗时
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "守护勋章";
    }
```





## 1.回顾Future

- Future接口：java5提供的接口

- CompletableFuture：java8提供的接口。CompletableFuture实现了Future接口



Future进行异步调用

```java
    public static void useFuture() throws Exception{
        ExecutorService executorService = Executors.newFixedThreadPool(10);

        long startTime = System.currentTimeMillis();
        // 1.调用业务1
        FutureTask<String> userFutureTask = new FutureTask<>(() -> {
            return getUserInfo();
        });
        executorService.submit(userFutureTask);

        Thread.sleep(300); //模拟主线程其它操作耗时

        // 2.调用业务2
        FutureTask<String> medalFutureTask = new FutureTask<>(() -> {
            return getMedalInfo();
        });
        executorService.submit(medalFutureTask);


        /**
         * Future对于结果的获取，不是很友好，只能通过阻塞或者轮询的方式得到任务的结果。
         * 1.Future.get() 就是阻塞调用，在线程获取结果之前get方法会一直阻塞。
         * 2.Future提供了一个isDone方法，可以在程序中轮询这个方法查询执行结果
         * 注：阻塞的方式和异步编程的设计理念相违背，而轮询的方式会耗费无谓的CPU资源
         * 注：CompletableFuture提供了一种观察者模式类似的机制，可以让任务执行完成后通知监听的一方
         */
        String userInfo = userFutureTask.get(); // 获取个人信息结果
        String medalInfo = medalFutureTask.get(); // 获取勋章信息结果

        System.out.println("总共用时" + (System.currentTimeMillis() - startTime) + "ms");

        executorService.shutdown();
    }
```



```java
    public static void main(String[] args) throws Exception{
        // 1."future + 线程池" 异步配合
        // 2.不实用异步需要时间 300+500+300=1100ms，使用异步则在1000ms内完成业务
        useFuture();
    }
```




## 2.CompletableFuture快速入门

```java
   public static void useCompletableFuture() throws Exception{
        long startTime = System.currentTimeMillis();
     
				// completableFuture使用了默认线程池是 "ForkJoinPool.commonPool"
     
        // 1.调用用户服务获取用户基本信息
        CompletableFuture<String> userCompletableFuture = CompletableFuture.supplyAsync(() -> getUserInfo());

        Thread.sleep(300); //模拟主线程其它操作耗时
			
     		// 2.调用获取用户勋章
        CompletableFuture<String> medalCompletableFuture = CompletableFuture.supplyAsync(() -> getMedalInfo());

        // 设置超时时间为2s 不设置userCompletableFuture的get方法会一直阻塞主线程
     		// String userInfo = userCompletableFuture.get();
        String userInfo = userCompletableFuture.get(2, TimeUnit.SECONDS); // 获取个人信息
        String medalInfo = medalCompletableFuture.get(2, TimeUnit.SECONDS); // 获取勋章信息结果
        System.out.println("总共用时" + (System.currentTimeMillis() - startTime) + "ms");
    }
```


## 3.CompletableFuture的使用场景

### 3.1 创建异步任务的2种方式

#### 3.1.1 supplyAsync：执行CompletableFuture任务，支持返回值

```java
//使用默认内置线程池ForkJoinPool.commonPool()，根据supplier构建执行任务
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
//自定义线程，根据supplier构建执行任务
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```


#### 3.1.2 runAsync：执行CompletableFuture任务，没有返回值

```java
//使用默认内置线程池ForkJoinPool.commonPool()，根据runnable构建执行任务
public static CompletableFuture<Void> runAsync(Runnable runnable) 
//自定义线程，根据runnable构建执行任务
public static CompletableFuture<Void> runAsync(Runnable runnable,  Executor executor)
```



#### 3.1.3 supplyAsync和runAsync的使用

```java
    public static void useAsync() {
        //可以自定义线程池
        ExecutorService executor = Executors.newCachedThreadPool();
        //runAsync的使用
        CompletableFuture<Void> runFuture = CompletableFuture.runAsync(() -> getUserInfo(), executor);
        //supplyAsync的使用
        CompletableFuture<String> supplyFuture = CompletableFuture.supplyAsync(() -> getUserInfo(), executor);

        /**
         * join()和get()方法都是用来获取CompletableFuture异步之后的返回值. 
         * 1.join()：方法抛出的是uncheck异常（即未经检查的异常),不会强制开发者抛出
         * 2.get()：抛出的是经过检查的异常，ExecutionException, InterruptedException 需要用户手动处理（抛出 或者 try catch）
         */
      
        // runAsync的future没有返回值，输出null
        System.out.println(runFuture.join());
        // supplyAsync的future，有返回值
        System.out.println(supplyFuture.join());
        executor.shutdown(); // 线程池需要关闭
    }
```


### 3.2 简单任务异步回调

#### 3.2.1 thenRun/thenRunAsync

```java
/**
  * CompletableFuture的thenRun：做完第一个任务后，再做第二个任务。某个任务执行完成后，执行回调方法；但是前后两个任务没有参数传递，第二个任务也没有返回值
*/

/**
  * 如果你执行第一个任务的时候，传入了一个自定义线程池
  * 1.调用thenRun方法执行第二个任务时，则第二个任务和第一个任务是"共用同一个线程池"
  * 2.调用thenRunAsync执行第二个任务时，则第一个任务使用的是你自己传入的线程池，第二个任务使用的是"ForkJoin线程池"
  * 注：thenAccept和thenAcceptAsync，thenApply和thenApplyAsync等同理
*/
public CompletableFuture<Void> thenRun(Runnable action);
public CompletableFuture<Void> thenRunAsync(Runnable action);
```



```java
		public static void thenCom(){
        CompletableFuture<Void> voidCompletableFuture = CompletableFuture.supplyAsync(() -> {
            return "执行任务1";
        }).thenRun(() -> {
            System.out.println("执行任务2");
        });

        // voidCompletableFuture没有返回值
    }
```



#### 3.2.2 thenAccept/thenAcceptAsync

```java
    /**
     * CompletableFuture的thenAccept：第一个任务执行完成后，执行第二个回调方法任务，会将该任务的执行结果，作为入参，传递到回调方法中，但是回调方法是没有返回值的。
     */
    public static void thenAcceptCom(){
        CompletableFuture<Void> thenAcceptFuture = CompletableFuture.supplyAsync(() -> "方法1参数").thenAccept((param1) -> {
            if ("方法1参数".equals(param1)) {
                System.out.println("方法2使用了方法1的参数：" + param1);
            }else {
                System.out.println("方法2没有使用方法1的参数");
            }
        });

        // thenAcceptFuture没有返回值
    }
```


#### 3.2.3 thenApply/thenApplyAsync

```java
    /**
     * CompletableFuture的thenApply：第一个任务执行完成后，执行第二个回调方法任务，会将该任务的执行结果，作为入参，传递到回调方法中，并且回调方法是有返回值的
     */
    public static void thenApplyCom() throws Exception {
        CompletableFuture<String> thenApplyFuture = CompletableFuture.supplyAsync(() -> "方法1参数").thenApply((param1) -> {
            if ("方法1参数".equals(param1)) {
                return "方法2使用了方法1的参数,并返回数据";
            } else {
                return "方法2没有使用方法1的参数";
            }
        });
        
        System.out.println(thenApplyFuture.get(2,TimeUnit.SECONDS));
    }
```



#### 3.2.4 异常处理：handle、whenComplete、exceptionally

```java
    /**
     * CompletableFuture的exceptionally：相当于catch捕获异常，异常时给一个默认值
     */
    public static void thenExeCom() throws Exception {
        String str = "test";
        CompletableFuture<String> data = CompletableFuture.supplyAsync(() -> {
            if ("test".equals(str)) {
                throw new RuntimeException("抛出异常");
            }
            return "正常数据：1.0";
        }).exceptionally(e -> {
            // e.printStackTrace();
            return "默认值：2.0";
        });
        System.out.println(data.get(2, TimeUnit.SECONDS));
        
        // String str = "test1";  输出：正常数据：1.0
        // String str = "test";  输出：正常数据：2.0
    }
```

### 3.3 多个任务组合处理

#### 3.3.1 AND组合关系

```java
   /**
     * thenCombine / thenAcceptBoth / runAfterBoth都表示：将两个CompletableFuture组合起来，只有这两个都正常执行完了，才会执行某个任务
     * 1.thenCombine：会将两个任务的执行结果作为方法入参，传递到指定方法中，且 有返回值
     * 2.thenAcceptBoth: 会将两个任务的执行结果作为方法入参，传递到指定方法中，且 无返回值
     * 3.runAfterBoth不会把执行结果当做方法入参，且没有返回值。
     */

    /**
    * AND组合是单线程阻塞，AllOf是多线程并发不阻塞
    * OR组合是单线程阻塞，AnyOf是多线程并发不阻塞
    */
    public static void thenAndCom() throws Exception {
        CompletableFuture<String> oneComFuture = CompletableFuture.supplyAsync(() -> "异步接口1");
        CompletableFuture<String> twoComFuture = CompletableFuture.supplyAsync(() -> "异步接口2");

        // 1.thenCombine
        CompletableFuture<String> strCombine = oneComFuture.thenCombine(twoComFuture, (e1, e2) -> "异步任务结果：" + e1 + ";" + e2);

        // 2.thenAcceptBoth
        oneComFuture.thenAcceptBoth(twoComFuture, (e1, e2) -> System.out.println("无返回值：" + e1 + "；" + e2));

        // 3.runAfterBoth
        oneComFuture.runAfterBoth(twoComFuture, () -> System.out.println("无参无返回值"));
    }
```



#### 3.3.2 OR组合关系

```java
    /**
     * applyToEither / acceptEither / runAfterEither：将两个CompletableFuture组合起来，只要其中一个执行完了,就会执行某个任务。
     * 1.applyToEither：会将已经执行完成的任务，作为方法入参，传递到指定方法中，且有返回值
     * 2.acceptEither: 会将已经执行完成的任务，作为方法入参，传递到指定方法中，且无返回值
     * 3.runAfterEither：不会把执行结果当做方法入参，且没有返回值
     */
    public static void thenOrCom() throws Exception {
        CompletableFuture<String> oneComFuture = CompletableFuture.supplyAsync(() -> "异步接口1");
        CompletableFuture<String> twoComFuture = CompletableFuture.supplyAsync(() -> "异步接口2");

        // 1.applyToEither
        CompletableFuture<String> applyToEither = oneComFuture.applyToEither(twoComFuture, s -> "有一个结果：" + s);
        // 2.acceptEither
        oneComFuture.acceptEither(twoComFuture, s -> {});
        // 3.runAfterEither
        oneComFuture.runAfterEither(twoComFuture, () -> {});
    }
```


#### 3.3.3 AllOf 和 AnyOf

```java
 /**
     * AllOf 和 AnyOf
     * 1.AllOf：所有任务都执行完成后，才执行 allOf返回的CompletableFuture。如果任意一个任务异常，allOf的CompletableFuture，执行get方法，会抛出异常
     * 2.AnyOf：任意一个任务执行完，就执行anyOf返回的CompletableFuture。如果执行的任务异常，anyOf的CompletableFuture，执行get方法，会抛出异常
     */
    public static void thenAllOfCom() throws Exception {
        CompletableFuture<String> oneComFuture = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("执行异步接口1");
            return "异步接口1";
        });
        CompletableFuture<String> twoComFuture = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("执行异步接口2");
            return "异步接口2";
        });

        // 1.输出：执行异步接口1  执行异步接口2
        // CompletableFuture<Void> voidCompletableFuture = CompletableFuture.allOf(oneComFuture, twoComFuture);
        // voidCompletableFuture.join();
        
        // 2.输出：执行异步接口2
        CompletableFuture<Object> anyOf = CompletableFuture.anyOf(oneComFuture, twoComFuture);
        anyOf.join();
    }
```


#### 3.3.4 ThenCompose

```java
    /**
     * thenCompose：在某个任务执行完成后，将该任务的执行结果,作为方法入参,去执行指定的方法。该方法会返回一个新的CompletableFuture实例
     */
    public static void thenCompose() throws Exception {
        CompletableFuture<String> oneComFuture = CompletableFuture.supplyAsync(() -> "异步接口1");

        CompletableFuture<String> newCompletableFuture = oneComFuture.thenCompose(data -> CompletableFuture.supplyAsync(() -> {
            return data + "&新的CompletableFuture";
        }));

        System.out.println(newCompletableFuture.join());;
    }
```


## 4.CompletableFuture使用有哪些注意点

### 4.1 Future需要获取返回值，才能获取异常信息

```java
1.在get()或者join()的时候才会获取异常
2.考虑是否加 try-catch 或者 使用exceptionally方法。
```



### 4.2 CompletableFuture的get()方法是阻塞的

```java
//反例
 CompletableFuture.get();
//正例
CompletableFuture.get(5, TimeUnit.SECONDS);
```



### 4.3 默认线程池的注意点（自定义线程池时，注意饱和策略）

```java
/**
* 
1.CompletableFuture代码中又使用了默认的线程池，处理的线程个数是电脑CPU核数-1。在大量请求过来的时候，处理逻辑复杂的话，响应会很慢。一般建议使用自定义线程池，优化线程池配置参数。

2.但是如果线程池拒绝策略是 DiscardPolicy或者 DiscardOldestPolicy，当线程池饱和时，会直接丢弃任务，不会抛弃异常。因此建议，CompletableFuture线程池策略最好使用AbortPolicy，然后耗时的异步线程，做好线程池隔离哈
*/
```



## 参考文档：https://mp.weixin.qq.com/s/nLMFS66kKSJ9NEjIay8b2w