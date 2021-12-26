



[TOC]



# Docker常用命令



**Docker常用命令**

![](./image/docker命令.png)





###  1.拉取镜像

```yacas
# 镜像名:版本名[:标签]  https://hub.docker.com/_/mysql?tab=tags
docker pull nginx           #下载最新版
docker pull nginx:1.20.1    #下载指定版本
docker pull redis:6.2.4     # 
```

### 2.查看镜像

```yacas
# 查看所有镜像（下载来的镜像都在本地）  
docker images  
```

### 3.删除镜像

```yacas
# 删除镜像 注：redis = redis:latest
docker rmi 镜像名:版本号/镜像id
```

### 4.启动容器

```yacas
# 启动容器 OPTIONS设置项 、 IMAGE镜像名 、 COMMAND命令
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# 查看帮助命令
docker run --help   
```

### 5.查看容器日志

```yacas
# 查看docker容器日志
docker logs 容器名/id 
# 动态打印日志
docker logs -f 容器id   
```
### 6.查看宿主机中容器

```yacas
# 查看正在运行的容器
docker ps

# 查看所有
docker ps -a
```

### 7.进入容器

```yacas
# 进入容器内部的系统，修改容器内容  -it以交互模式
docker exec -it 容器id  /bin/bash

# 退出容器交互模式
exit
```

### 8.容器和宿主机copy文件

```yacas
# 从容器内拷贝到主机上
docker cp 容器id 容器内路径 目的地的主机路径

#把容器指定位置的东西复制出来 
docker cp 容器id:/etc/nginx/nginx.conf  /data/conf/nginx.conf
#把外面的内容复制到容器里面
docker cp  /data/conf/nginx.conf  容器id:/etc/nginx/nginx.conf
```

### 9.停止/再次启动/设置开机自启/删除容器

```yacas
#停止容器
docker stop 容器id/名字

#再次启动
docker start 容器id/名字

# 删除停止的容器
docker rm  容器id/名字
docker rm -f 容器id/名字  # 强制删除正在运行中的

#应用开机自启
docker update 容器id/名字 --restart=always
```

### 10.修改容器并保存到本地镜像仓库

```yacas
# 拉去镜像
docker pull nginx:1.20.1
# 运行镜像  ro只读模式、rw读写模式
docker run --name=mynginx   \
-d  --restart=always \
-p  8777:80 -v /data/html:/usr/share/nginx/html:ro  \
youlinginx:v1.0
# 进入宿主机目录/data/html
cd /data/html
# 给文件添加内容
echo "youli" >index.html
# 外部访问nginx
http://101.34.59.162:8777/    
# docker commit提交容器  -a用户名、-m修改记录、CONTAINER容器id、"重命名:版本"
docker commit --help
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
docker commit -a "youli"  -m "首页变化" 287016a06473 youlinginx:v1.0
```

### 11.镜像传输 

#### 镜像传输方式1：scp传输

```yacas
# 将镜像保存成压缩包
docker save --help
docker save [OPTIONS] IMAGE [IMAGE...]
docker save -o youlinginx.tar youlinginx:v1.0

# 将本机linux下打包的文件传给其他服务器
scp youlinginx.tar root@121.4.88.61:/opt

# 别的机器加载这个镜像
docker load -i youlinginx.tar

# 查看传过来解压后的镜像
docker images
```

#### 镜像传输方式2：push到远程镜像仓库

```yacas
# 登录到docker hub
docker login  

# 把旧镜像的名字，改成仓库要求的新版名字"a995525192/youlinginx:v1.0"
docker tag local-image:tagname new-repo:tagname
docker tag youlinginx:v1.0 a995525192/youlinginx:v1.0

# 推送
docker push new-repo:tagname
docker push a995525192/youlinginx:v1.0

# 退出docker hub
docker logout（推送完成镜像后退出）

# 别的机器下载
docker pull a995525192/youlinginx:v1.0
```





