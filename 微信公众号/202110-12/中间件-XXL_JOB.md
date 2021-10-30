
## XXL-JOB

#### 1. [XXL-JOB码云地址](https://gitee.com/xuxueli0323/xxl-job?_from=gitee_search)



#### 2.版本选择 v2.2.0

**2.1 XXL-JOB源码结构**
![](https://gitee.com/domineering_red_tide/image/raw/master/image/xxl-job源码结构.png)



**2.2 执行表结构**
![](https://gitee.com/domineering_red_tide/image/raw/master/image/表结构.png)





**2.3 业务系统和XXL-JOB系统的关系**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/2bb4051ba013c6395578d1bc3b61878.png)



#### 3.启动并登录任务调度后台管理系统(XXL-JOB系统)

~~~java

3.1 启动后登录网址：http://localhost:8080/xxl-job-admin/

3.2 登录用户/密码 ：admin/123456

~~~

![](https://gitee.com/domineering_red_tide/image/raw/master/image/启动项目后端截图.png)




![](https://gitee.com/domineering_red_tide/image/raw/master/image/启动项目前端界面.png)





#### 4.启动业务系统（建议使用springboot框架）

**4.1 编写定时任务业务**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/编写任务业务.png)




~~~java

/**
 * xujiaqi、简单任务示例（Bean模式）
 */
@XxlJob("demoXuJiaQiHandler")
public ReturnT<String> demoXuJiaQiHandler(String param) throws Exception {
        XxlJobLogger.log("你好，我是叶秋");

        System.out.println("你好，我是叶秋");
        return ReturnT.SUCCESS;
}

~~~



**4.2 业务系统中配置文件**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/业务系统关于任务系统的配置.png)



**4.3 启动业务系统**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/768213ca277be431c5b59ee8ab809a1.png)



**5.测试XXL-JOB系统的定时任务**

**5.1 配置执行器（执行器管理）**


![](https://gitee.com/domineering_red_tide/image/raw/master/image/614e2059250872f7bb366aab2dfbfe8.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/2fa12b790c7431c9d1e0ad48ad111eb.png)



**5.2 配置定时任务（任务管理）**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/d1008d910176a9a0481b826dec118cb.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/252666a7d12034a804b58af9c53435f.png)



**5.3 执行定时任务并查看执行结果**

![](https://gitee.com/domineering_red_tide/image/raw/master/image/62a20615187129a0927602959bc9024.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/bd18ce52ca55813456a27ffe9bf75f1.png)








