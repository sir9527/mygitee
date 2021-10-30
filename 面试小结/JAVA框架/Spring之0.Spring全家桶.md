



[TOC]



## 1.Spring

Spring是轻量级的 **控制反转（IOC）**和 **面向切面（AOP）**的容器（框架）

- ioc：控制反转（通过依赖注入实现）

- aop：切面编程。找到切入点（pointCut），在切入点中的切面（一个类是一个切面）前后进行织入编程。

  

注：AOP底层实现原理是动态代理。
       动态代理有2种，cglib实现 和  JDK实现（默认）。
       JDK实现是默认实现，是基于接口的动态代理；cglib实现是基于类的动态代理



注：@Autowired先按 byType 找，找不到就按照 byName 找 
        @Resource 先按 byName 找，找不到就按照 byType 找 



注：过滤器、拦截器、AOP：拦截器本质就是AOP



**过滤器和拦截器的区别**

- 过滤（Servlet规范）：实现 Filter接口 或者 web.xml文件配置

        实际应用自定义过滤器：解决get和post请求 全部乱码的过滤器

- 拦截器（SpringMVC框架自己的）：实现 HandlerInterceptor接口

       拦截器是AOP思想的具体应用；拦截器只拦截控制方法；实际应用：用户登录验证信息



## 2.SpringMVC

SpringMVC：通过DispatcherServlet 调用 "数据处理器 + 视图解析器" 返回一个视图界面



## 3.Mybatis

Mybatis的缓存机制

- 一级缓存：一次会话（SqlSession）

- 二级缓存：基于namespace级别的缓存（Mapper）

- 第三方缓存库：memcached, ehcache 等



## 4.SpringBoot

springBoot自动装配原理

1.启动类的@SpringBootApplication注解是个复合注解，里面有@EnableAutoConfiguration注解用来开启自动配置的功能

2.@EnableAutoConfiguration 注解也是复合注解，里面有个AutoConfigurationImportSelector

3.AutoConfigurationImportSelector点进去会看到会扫描

    spring-boot-autoconfigure包里 "META-INF" --> "spring.factories"文件，里面全是各种AutoConfiguration，项目启动会注册这些bean。这些AutoConfiguration点进去有个Properties注解，点进去就看到bean的参数的配置信息，这个就是咋没呢yum文件配置bean的信息。





**spring-boot-autoconfigure包里 "META-INF"的"spring.factories"文件**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210611104623.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210611115000.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210611115115.png)





## 5.SpringCloud

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210611120623.png)



