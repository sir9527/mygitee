

[TOC]



DockerCompose容器编排，相比k8s编排能力还是弱了点



### 1.下载DockerCompose

```yacas
## 1.下载 Compose 只执行文件到 usr/local/bin/ 目录
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

## 2.对 Compose 可执行文件添加运行权限
sudo chmod +x /usr/local/bin/docker-compose

## 3.输入下面命令查看帮助，测试安装是否成功
docker-compose -h
```



### 2.DockerCompose编排容器命令

```yacas
1.version     # 版本，不要超过4.0
2.services    # 服务
3.tomcat      # 服务id 名字随便起
  - build          # 打包镜像  类似"docker build -t yygh-hopital:v1.0"
    - context      # 指定上下文目录 即dockerfile所在目录
    - dockerfile   # 名字一般都是Dockerfile，尽量不要自定义
  - container_name # 容器名字 可以用来替代域名进行通信 类似"docker run的--name
  - ports          # 暴露端口号 类似docker run 的 -p
  - networks # 类似docker run的--network。所属网路 随便起个名字，多个容器要用同一个 
  - depends_on     # 服务启动前依赖的容器
  - image          # 服务使用的镜像
  - environment    # 配置参数
  - command        # 执行命令
  - volumes        # 类似docker run 的 -v
  - healthcheck（不重要）    # 容器健康心跳检测
  - sysctls（不重要）        # 修改容器内系统参数 
  - ulimits（不重要）        # 修改容器中系统内部进程数限制 不必须
4.volumes   使用到的volumes必须声明
5.networks  使用到的networks必须声明
```

```yacas
version: "3.0"
services:
  tomcat:
    container_name: mytomcat
    image: tomcat:8.0-jre8
    ports:
      - "8080:8080"
    volumes:
      - "tomcatwebapps:/usr/local/tomcat/webapps"
    networks:
      - some_network
    depends_on:
      - mysql
      - redis
  mysql:
    container_name: mysql
    image: mysql:5.7.32
    ports:
      - "3306:3306"
    volumes:
      - "mysqldata:/var/lib/mysql"
      - "mysqlconf:/etc/mysql"
    environment:
      - MYSQL_ROOT_PASSWORD=1234
    networks:
      some_network:
  redis:
    container_name: redis
    image: redis:5.0.10
    ports:
      - "6379:6379"
    volumes:
      - "redisdata:/data"
    command: "redis-server --appendonly yes"
    networks:
      some_network:
volumes:
  tomcatwebapps: 
  mysqldata:
  mysqlconf:
  redisdata: 
networks:
  some_network:
```



### 3.DockerCompose编排容器命令

启动

```yacas
# 后台启动 yaml 定义的所有容器
docker-compose up -d

# 仅启动 mysql 这个service
docker-compose up mysql 
```
查看
```yacas
# 查看各个 service 容器内运行的进程情况
docker-compose top

# 列出当前 yaml 中定义的容器的信息
docker-compose ps
```
进入容器
```yacas
# 进入 redis 这个 service 使用 exit 退出
docker-compose exec redis bash
```
停止/删除
```yacas
# 停止容器并移除自动创建的网桥
docker-compose down 

# 删除当前 yaml 中定义的容器，需要先 stop，后面可以指定上某个具体的 service
docker-compose rm
```
暂停/恢复
```yacas
# 暂停 和 恢复
docker-compose pause
docker-compose unpause
```
重启
```yacas
# 重启所有 service 后面可以指定上某个具体的 service
docker-compose restart
```

查看容器日志
```yacas
# 查看日志默认查看 yaml 所有的，可以跟上具体 service
docker-compose logs
docker-compose -f logs
```



### 4.我的实战项目

```yacas
1.按照下面目录新建，并上传jar包
2.在gateway目录、user目录、order目录下执行"docker build -t yygh-hopital:v1.0"将jar包打成镜像
3.docker image 查看镜像
4.touch docker-compose.yml 
5.docker目录下执行"docker-compose up"启动所有容器 或者 "docker-compose up -d"后台启动所有容器
6.docker ps 查看启动的容器
7.docker logs -f 容器id 查看容器日志
```

```yacas
- docker目录
  - docker-compose.yml文件
  - gateway目录
    - yygh_gateway_80.jar
    - Dockerfile文件
  - user目录
    - yygh_user_8160.jar
    - Dockerfile文件
  - order目录
    - yygh_order_8206.jar
	- Dockerfile文件
```

```yacas
# 基础镜像
FROM openjdk:8-jdk-slim
# 暴露端口
EXPOSE 80
# 复制jar包
COPY target/yygh_gateway_80.jar /yygh_gateway.jar
# 启动命令
ENTRYPOINT ["java","-jar","/yygh_gateway.jar"]
```

```yacas
其他的Dockerfile...
```

```yacas
version: "3.0"
services:
  yygh_gateway:
    container_name: yygh_gateway
    image: yygh-gateway:v2.0
    ports:
      - "80:80"
  yygh_user:
    container_name: yygh_user
    image: yygh-user:v2.0
    ports:
      - "8160:8160"
  yygh_order:
    container_name: yygh_order
    image: yygh-order:v2.0
    ports:
      - "8206:8206"
```



### 5.其他

```yacas
1.根据Dockerfile打包镜像
2.编写touch docker-compose.yml
3.启动容器docker-compose up
```

```yacas
- docker文件夹
  - docker-compose.yml
  - employee文件夹
    - Dockerfile文件
    - ems-employees-1.0.jar
  - department文件夹
    - Dockerfile文件
	- ems-department-1.0.jar
  - gateway文件夹
    - Dockerfile文件
	- ems-gateway-1.0.jar
```
```yacas
FROM openjdk:8-jre
ENV APP_HOME=/apps
WORKDIR $APP_HOME
COPY ./ems-gateway-1.0.jar ./ems-gateway.jar
ENTRYPOINT ["java","-jar"]
CMD ["gateway.jar"]
```

```yacas
FROM openjdk:8-jre
ENV APP_HOME=/apps
WORKDIR $APP_HOME
COPY ./ems-department-1.0.jar ./ems-dept.jar
ENTRYPOINT ["java","-jar"]
CMD ["dept.jar"]
```

```yacas
FROM openjdk:8-jre
ENV APP_HOME=/apps
WORKDIR $APP_HOME
COPY ./ems-employees-1.0.jar ./ems-employees.jar
ENTRYPOINT ["java","-jar"]
CMD ["employees.jar"]
```
```yacas
version: "3.8"
services:
  employee:
    build:
      context: ./employee
      dockerfile: Dockerfile
    ports:
      - "8081:8081"
    networks:
      - ems
  department:
    build:
      context: ./department
      dockerfile: Dockerfile
    ports:
      - "8082:8082"
    networks:
      - ems
  gateway:
    build:
      context: ./gateway
      dockerfile: Dockerfile
    ports:
      - "8888:8888"
    networks:
      - ems
  nacos:
    container_name: yygh_nacos
    image: nacos/nacos-server:1.3.1
    ports:
      - "8848:8848"
    environment:
      - "MODE=standalone"
    networks:
      - ems
  mysql:
    container_name: yygh_mysql
    image: mysql:5.7
    ports:
      - "3306:3306"
    environment: 
      - "MYSQL_ROOT_PASSWORD=root"
      - "MYSQL_DATABASE=ems"
    volumes:
      - mysqldata:/var/lib/mysql
      - ./ems.sql:/docker-entrypoint-initdb.d/ems.sql
    networks:
      - ems
  redis:
    container_name: yygh_redis
    image: redis:5.0.10
    ports:
      - "6379:6379"
    volumes:
      - "redisdata:/data"
    command: "redis-server --appendonly yes"
    networks:
      - ems
volumes: 
  mysqldata:
  redisdata: 
networks: 
  ems:
```



