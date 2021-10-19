

[TOC]



# 1.K8s

大规模容器编排系统



## 1.1 K8s组成
![k8s04](../../../../云原生课件笔记/云原生笔记/image/k8s04.png)

### 主节点master

- kube-apiserver

  - 对外暴露k8s的api接口，是外界进行资源操作的唯一入口
  - 提供认证、授权、访问控制、API注册和发现等机制

- etcd

  - 是兼具一致性和高可用性的键值数据库，可以作为保存Kubernetes所有集群数据的后台数据库
  - Kubernetes集群的数据库通常需要有个备份计划
  - 注：数据库。存数据

- kube-scheduler

  - 主节点上的组件，该组件监视那些新创建的未指定运行节点的Pod，并选择节点让Pod在上面运行
  - 所有对k8s的集群操作，都必须经过主节点进行调度

  - 注：任务调度。外界操作k8s，会先保存到etcd数据库中，scheduler从etcd中读取数据调度操作节点node，从而操作k8s

- kube-controller-manager

  - 在主节点上运行控制器的组件
  
  - 这些控制器包括
    - 节点控制器(NodeController):负责在节点出现故障时进行通知和响应
    
    - 副本控制器(ReplicationController)：负责为系统中的每个副本控制器对象维护正确数量的Pod
    
    - 端点控制器（EndpointsController)：填充端点（Endpoints）对象（即加入service与Pod)
    
    - 服务帐户和令牌控制器(Service Account&Token Controllers)．为新的命名空间创建默认帐户和API访可令牌
    
  - 注：真正操作k8s的组件      



### Node节点（运行服务）

- kubelet
  - 一个在集群中每个Node节点上运行的**代理**。它保证容器都运行在Pod中
  - 负责维护容器的生命周期，同时也负责Volume(CSI)和网络(CNI)的管理
  - 注：Pod是一个Pod包含一个或多个容器，提供一个完整的API功能。**Pod是k8s中最小的部署单元**。
- kube-proxy
  - 负责为提供cluster内部的服务发现和负载均衡
  - 注：网络代理，可以理解为路由器、网卡这种
- 容器运行环境(Container Runtime)
  - 容器运行环境是负责运行容器的软件
  - Kubernetes支持多个容器运行环境．Docker、containerd、cri-o、rktlet以及任何实现Kubernetes CRI（容器运行环境接口）。默认使用Docker。
- fluentd
  - 是一个守护进程，它有助于提供集群层面日志集群层面的日志



### 下面，以部署一个nginx服务来说明kubernetes系统各个组件调用关系

1.首先要明确，一旦kubernetes环境启动之后，master和node都会将自身的信息存储到etcd数据库中

2.一个nginx服务的安装请求会首先被发送到master节点的apiServer组件

3.apiServer组件会调用scheduler组件来决定到底应该把这个服务安装到哪个node节点上

4.在此时，它会从etcd中读取各个node节点的信息，然后按照一定的算法进行选择，并将结果告知apiServer

5.apiServer调用controller-manager去调度Node节点安装nginx服务

6.kubelet接收到指令后，会通知docker，然后由docker来启动一个nginx的pod

7.pod是kubernetes的最小操作单元，容器必须跑在pod中至此，

8.一个nginx服务就运行了，如果需要访问nginx，就需要通过kube-proxy来对pod产生访问的代理



### 重要定义

Master：集群控制节点，每个集群需要至少一个master节点负责集群的管控

Node：工作负载节点，由master分配容器到这些node工作节点上，然后node节点上的docker负责容器的运行

Container：容器。docker镜像启动的容器。

Pod：kubernetes的最小控制单元，容器都是运行在pod中的，一个pod中可以有1个或者多个容器

Volume：挂载文件

Controller：控制器，通过它来实现对pod的管理，比如启动pod、停止pod、伸缩pod的数量等等（控制器在k8s中是一类概念，有很多控制器）

Service：pod对外服务的统一入口，下面可以维护者同一类的多个pod。如通过service访问多个tomcat。

Label：标签，用于对pod进行分类，同一类pod会拥有相同的标签（service怎么判断应该给哪个pod，如下面3个都是tomcat，只有一个tom，所以没选它）

NameSpace：命名空间，用来隔离pod的运行环境（同个命名空间里的pod可以相互访问）



## 1.2 搭建K8s

### 1.2.1 K8s架构

![k8s06.png](../../../../云原生课件笔记/云原生笔记/image/k8s06.png)

3台机器（1个master，2个node）

- Docker
- kubelet  每个节点的管理者
- kubectl 和 kubeadm
  - kubectl  帮助程序员管理k8s         注：不要也可以
  - kubeadm 帮助程序员搭建k8s      注：不要也可以





### 1.2.2 搭建K8s步骤

1.创建私有网络（VPC）

- 一个VPC下的一个中集群机器组内互信，可以相互访问（私网IP）

2.3台云服务器

- 1台master服务器
- 2台Node服务器
- 注：服务器性能 2核4G
- 注：3台机器用私网IP **Ping一下**

3.3台服务器基础环境

- 安装Docker环境（参考Docker笔记）

- 关闭交换分区swap

- 安装3大件：kubelet、kubeadm、kubectl

  - kubelet：管理节点。注：各节点的厂长
  - kubeadm：程序员操作k8s集群工具
    - kubeadm token create --print-join-command  # 新令牌
    - kubeadm init  # 初始化主节点
    - kubeadm join cluster-endpoint:6443 --token hums8f.vyx71prsg74ofce7 \
          --discovery-token-ca-cert-hash sha256:a394d059dd51d68bb007a532a037d0a477131480ae95f75840c461e85e2c6ae3 # 将节点node加入到主节点master中
  - kubectl：程序员管理k8s集群工具。**常用命令**

    - kubectl get nodes   \# 查看集群所有节点
    - kubectl get pods -A   # 相当于 docker ps。查看集群部署了哪些应用。**运行中的应用在docker里面叫容器，在k8s里面叫Pod** 
    - kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
    - kubectl apply -f recommended.yaml  \#  根据配置文件，给集群创建资源

4.初始化主节点

5.初始化2个节点

6.部署k8s可视化工具dashboard

- master节点执行一行命令即可
- 可视化界面：https://集群任意IP:端口      https://139.198.165.238:32759

**7.k8s中的资源**

- Namespace：名称空间。隔离资源，不隔离网络
- Pod：运行中的一组容器，Pod是kubernetes中应用的最小单位.
- 工作负载
  - **Deployment**：无状态应用部署，比如微服务，提供多副本等功能。控制Pod，使Pod拥有多副本，自愈，扩缩容等能力。底层运行的是一个个的Pod。**用的多**
  - StatefulSet： 有状态应用部署，比如redis，提供稳定的存储、网络等功能。底层运行的是一个个的Pod。**用的多**
  - DaemonSet： 守护型应用部署，比如日志收集组件，在每个机器都运行一份
  - Job/CronJob：定时任务部署， 比如垃圾清理组件，可以在指定时间运行
- Service：Pod的服务发现与负载均衡。将一组 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 公开为网络服务的抽象方法。暴露一组pod服务。这组中的Pod是相同服务功能，为了负载均衡。
- Ingress：Service的统一网关入口。相当于nginx
- NFS：挂载。一个节点会备份其他节点数据。
  - 方式1：原生挂载。将节点文件夹和NFS的文件夹映射挂载
  - 方式2：PVC持久卷申明/PV持久卷 对应 Pod。挂载目录，**推荐使用**
  - 注：配置文件挂载ConfigMap（实际在k8s的etcd中）
  - **注：PVC挂载目录、ConfigMap挂载配置文件、Secret挂载敏感信息。**

- Secret
  - Secret保存敏感信息，例如密码、OAuth 令牌和 SSH 密钥。将这些信息放在 secret 中比放在 **Pod **的定义或者 **容器镜像** 中来说更加安全和灵活。
  - 注：区别于ConfigMap

8.k8s资源创建方式

- 命令行
  - kubectl create ns hello # 直接使用命令创建资源
- yaml
  - kubectl apply -f recommended.yaml  \#  根据配置文件，给集群创建资源





# 2.KubeSphere（简化K8s）

## 2.1 KubeSphere作用

云平台上的一站式能力：自动化运维部署、应用监控、日志收集、系统告警...



## 2.2 安装KubeSphere

### 2.2.1 3种安装方式

#### 1.安装方式1：安装Kubernetes后再安装KubeSphere

- 3台服务器
  - 4核8G（master）
  - 8核16G（node1）
  - 8核16G（node2）

- 安装Docker
  - 看之前的Docker笔记
- 安装Kubernetes
  - 看之前的k8s笔记（安装1个master节点、2各node节点）
- 安装KubeSphere前置环境
  - 之前用的PVC静态的储存文件，现在是要动态的PVC储存文件
  - 注：静态意思是提前分区硬盘，动态是存文件时在动态分配硬盘
- 安装KubeSphere
  - yaml配置文件方式安装KubeSphere



#### 2.安装方式2：Linux单节点直接部署KubeSphere

- 1台服务器
  - 4核8G

- 下载KubeKey --> 使用KubeKey引导安装集群 --> 安装后开启功能




#### 3.安装方式3：Linux多节点直接部署KubeSphere

- 3台服务器
  - 4核8G（master）
  - 8核16G（node1）
  - 8核16G（node2）

- 参考文档中心安装



### 2.2.2 KubeSphere实战

#### 项目1：多租户系统实战

- 用户 - 角色 - 权限

  

#### 项目2：中间件部署实战

- 方式1：手动使用docker镜像部署（不方便）
- 方式2：KubeSphere中的应用商店一键式部署（只有17款中间件）
- 方式3：使用应用商店Helm（第三方仓库 k8s中间件远程仓库）
- 

#### 项目3：RuoYi-Cloud项目部署（手动部署）

- 思路
  - 中间件：有状态、数据导入、上云后一般使用的都是内网IP或者内网域名通信
  - 微服务：无状态、制作镜像
  - 网络：各种访问地址
  - 配置：生产配置分离、URL

- 部署步骤

  - 部署中间件
    - mysql
    - nacos
    - redis
    - nginx

  - 部署微服务
    - 项目 "微服务jar包 / vue包 + Dockerfile文件"  制作镜像并上云
    - 注：k8s可以使用存活探针检测nacos是否启动，做一个健康检查机制。没启动会一直重试。



#### 项目4：尚医通DevOps部署（KubeSphere开启jenkins）

- 部署步骤
  - 部署中间件
    - mysql
    - redis
    - ...
  - 
  - jenkins可视化流水线（前后端部署2条流水线）
    - 设置Git云代码地址
    - maven打包项目    mvn clean package -Dmaven.test.skip=true
    - 构建镜像（多微服务可以并行构建）
    - 推送远程镜像仓库（多微服务可以并行推送）
    - 部署到k8s集群（多微服务可以并行部署）
    - 注：后端服务需要对外暴露
