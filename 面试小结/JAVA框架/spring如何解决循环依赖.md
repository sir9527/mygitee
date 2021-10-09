



### 1.参考文章：https://mp.weixin.qq.com/s?__biz=MzUxODkzNTQ3Nw==&mid=2247485600&idx=1&sn=0c49b94e7fbd35c88c4470e936023e3e&chksm=f9800e7acef7876ca05ab45ce9420ea140f188e84153f23d0af9d044f475458ad38d49a6546a&token=1641046204&lang=zh_CN&scene=21#wechat_redirect








### 2.spring循环依赖
spring通过三级缓存解决循环依赖

```java
@Service
publicclass TestService1 {

    @Autowired
    private TestService2 testService2;

    public void test1() {
    }
}
```

```java
@Service
publicclass TestService2 {

    @Autowired
    private TestService1 testService1;

    public void test2() {
    }
}
```



