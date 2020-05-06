# 1 k8s介绍

编排工具

+ 单机容器编排工具: docker-compose

+ 集群容器编排工具: docker swarm , kubernetes(k8s)



## 1) swarm 和 k8s

docker swarm 的manager节点和worker节点

<img src="https://images.gitee.com/uploads/images/2020/0506/153410_704cebd7_7530643.png" alt="image-20200506105035746" style="zoom:100%;" />



kubernetes 的master和node节点

<img src="https://images.gitee.com/uploads/images/2020/0506/153422_f4218af2_7530643.png" alt="image-20200506105202364" style="zoom:100%;" />





### k8s-master

<img src="https://images.gitee.com/uploads/images/2020/0506/153432_532b6bff_7530643.png" alt="image-20200506105519590" style="zoom:100%;" />



master模块：

#### API Server

+ 暴露给外界访问，通过 CLI 或者 UI 通过API server对集群进行通信交互访问

#### Scheduler

+ 调度模块，外界cli或者ui通过api接口部署应用，需要的容器部署在哪个节点上，通过scheduler调度算法将容器部署在哪个节点上

#### Controller

+ 集群管理/控制模块，比如控制某个web的横向扩展为2

#### etcd

+ 集群分布式的kv存储，存储k8s集群的状态和配置



### k8s-node

<img src="https://images.gitee.com/uploads/images/2020/0506/153442_2a3174f4_7530643.png" alt="image-20200506110320795" style="zoom:100%;" />



#### pod

+ 容器编排调度的最小单位，具有相同NetworkNameSpace的container组合，这个NameSpace包含了所有(UserNameSpace,NetworkNameSpace)，如果是多个container，他们是共享一个NetworkNameSpace

#### docker

+  底层依赖的容器技术

#### kubelent

+ 是k8s集群node的agent，接收master指令信息，对node中pod和volume进行管理

#### Kube-proxy

+ 对node中container暴露端口给外接访问，提供代理和转发，并且做负载均衡，实现服务发现

#### Fluentd

+ 日志的采集，存储，查询

#### Optional 插件

+ 提供DNS，UI，

#### Image Registry

+ 给集群提供image的仓库



### k8s调度

<img src="https://images.gitee.com/uploads/images/2020/0506/153451_893564e9_7530643.png" alt="image-20200506111540455" style="zoom:100%;" />





## 2 k8s集群搭建

### 搭建方式介绍

#### 1) kubernetes-the-hard-way最困难方式部署

  + https://github.com/kelseyhightower/kubernetes-the-hard-way

#### 2) minikube工具部署

  + https://github.com/kubernetes/minikube
  + 在本地创建1个node的k8s集群

#### 3) kubeadm 部署

  + https://github.com/kubernetes/kubeadm
  + 本地搭建多个节点的k8s集群

#### 4) kops 部署

  + https://github.com/kubernetes/kops
  + 在cloud部署k8s集群
    + 例如在aws的某个regin，搭建2，3个节点的k8s集群

#### 5) tectonic 部署

  + http://coreos.com/tectonic/
  + tectonic 工具小于10个节点免费，>10节点收费

#### 6) play-with-k8s 部署

  + https://labs.play-with-k8s.com/
  + 注册账号，在云端部署k8s，4小时集群会被销毁



### minikube部署

#### 1,安装minikube

+ https://minikube.sigs.k8s.io/docs/start/

```shell
# mac下安装
wangjunxiang@My_MacBook_pro_2018  ~/data/ISO/VirtualBox_VMs/chapter7  brew install minikube

# 如果 brew install 太慢 卡住 更换清华源
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

# 如果brew install还卡住 则
export HOMEBREW_NO_AUTO_UPDATE=true
```



#### 2,安装 kube-cli工具

+ https://kubernetes.io/docs/tasks/tools/install-kubectl/

#### 3,本地虚拟化工具 Vmware/VirtualBox

+ 

























