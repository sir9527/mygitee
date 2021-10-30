

[TOC]



#### 1.限流算法

##### 1.1 漏桶算法

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210604144408.png)



```

漏桶算法思路：水(请求)先进入到漏桶里,漏桶以一定的速度出水(接口有响应速率),当水流入速度过大会直接溢出(访问频率超过接口响应速率),然后就拒绝请求,可以看出漏桶算法能强行限制数据的传输速率


这里有两个变量：一个是桶的大小,支持流量突发增多时可以存多少的水,另一个是水桶漏洞的大小


注：Redis实现的漏桶算法是分布式限流
Redis可以实现漏斗算法：cl.throttle mylimit 15 30 60
其中 15 为漏斗的容量，30 / 60s 为漏斗的速率

注：Nginx 的控制速率其实使用的就是漏桶算法，属于分布式限流。控制IP多久访问1次

```



##### 1.2 令牌算法

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210604142342.png)





谷歌开源的guava（单机限流）

```

<!-- https://mvnrepository.com/artifact/com.google.guava/guava -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>28.2-jre</version>
</dependency>

```



```

/**
 * Guava 实现限流
 */
public class RateLimiterExample {
    public static void main(String[] args) {
        // 每秒产生 10 个令牌（每 100 ms 产生一个）
        RateLimiter rt = RateLimiter.create(10);
        for (int i = 0; i < 11; i++) {
            new Thread(() -> {
                // 获取 1 个令牌
                rt.acquire();
                System.out.println("正常执行方法，ts:" + Instant.now());
            }).start();
        }
    }
}

```



##### 1.3 漏桶算法和令牌算法的比较

```

1.漏桶：漏桶的出水速度是恒定的，那么意味着如果瞬时大流量的话，将有大部分请求被丢弃掉（也就是所谓的溢出）。
2.令牌桶：生成令牌的速度是恒定的，而请求去拿令牌是没有速度限制的。这意味，面对瞬时大流量，该算法可以在短时间内请求拿到大量令牌，而且拿令牌的过程并不是消耗很大的事情。

最后，不论是对于令牌桶拿不到令牌被拒绝，还是漏桶的水满了溢出，都是为了保证大部分流量的正常使用，而牺牲掉了少部分流量，这是合理的，如果因为极少部分流量需要保证的话，那么就可能导致系统达到极限而挂掉，得不偿失。

```



#### 2.单机限流

```

/*
* 信号量控制高并发：并发控制到10个
*/
public class DubboService {
 
    private final Semaphore permit = new Semaphore(10, true);
 
    public void process(){
        try{
            permit.acquire();
            // 业务逻辑处理
 			// ...
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            permit.release();
        }
    }
}

```





#### 3.分布式限流之sentinel

##### 3.1 启动sentinel的服务端

- 找到官方地址，下载jar，地址：https://github.com/alibaba/Sentinel/releases 

- 进入jar所在目录，在地址栏目输入 cmd，进入命令窗口，输入命令：java -jar sentinel-dashboard-1.8.0.jar 

- 访问web地址：[http://localhost:8080](http://localhost:8080/#/login)  默认用户名和密码都是sentinel/sentinel，然后进入首页 

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210604135314.png)





![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210604135458.png)



##### 3.2 搭建客户端

###### 3.2.1 搭建springCloud项目集成sentinel

**pom.xml文件**

```

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.0.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<groupId>com.baidu</groupId>
	<artifactId>mysentinel</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>mysentinel</name>
	<description>sentinel的简单使用</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<!-- web服务 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

        <!-- sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```



**application.properties文件**

```

server.port=8081
spring.application.name=frms

# sentinel 配置
spring.cloud.sentinel.transport.dashboard=localhost:8080
spring.cloud.sentinel.transport.heartbeat-interval-ms=500

```



**TestController文件**

```

@RestController
public class TestController {
    
    @GetMapping("mySentiel")
    public String mySentiel(){
        return "hello world";
    }
}

```



**访问web接口**

```

http://localhost:8081/mySentiel

```



###### 3.2.2 在sentinel服务端配置限流或者降级规则

配置限流规则（ 给mySentiel查询接口添加流控规则，它的QPS(每秒请求数)为1 ）

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210604135938.png)





配置降级规则

给mySentiel接口查询添加一个降级规则配置，如果QPS大于1，且平均响应时间大于20ms，则接口下来接口在2秒钟无法访问，之后自动恢复

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210604140143.png)






