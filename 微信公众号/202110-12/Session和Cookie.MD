



1.分布式session(分布式session)

​    1.Tomcat配置文件server.xml共享session

​    2.Nginx配置绑定IP

​    3.springSession             @EnableRedisHttpSession  //将整个应用中使用session的数据全部交给redis处理

​    4.JWT                             Token替代Session



2.session 和 cookie区别

​     cookie数据存放在客户的浏览器上，session数据放在服务器上