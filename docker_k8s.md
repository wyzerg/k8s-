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





# 2 k8s集群搭建

## 2.1 搭建方式介绍

### 1) minikube工具部署

+ 用途：一般用来 快速本地 开发 测试 用

  + https://github.com/kubernetes/minikube
  + 在本地创建1个node的k8s集群

### 2) kubeadm 部署

+ 用途：本地搭建 多个节点 的k8s集群

  + https://github.com/kubernetes/kubeadm

### 3) kops 部署

  + https://github.com/kubernetes/kops
  + 在cloud部署k8s集群
    + 例如在aws的某个regin，搭建2，3个节点的k8s集群



## 2.2 minikube部署单节点k8s

https://minikube.sigs.k8s.io/docs/start/

mac为例：

### 1) 下载minikube

#### brew安装minikube 或者 curl安装

+ 可以找到阿里云的minikube文件安装



##### 方式1：挂vpn国外安装...(二选一)

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

# --> mac环境安装minikube上述都不行，直接下载可执行文件
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

##### 方式2：阿里云国内安装(后来才知道)

阿里云发布的minikube地址：https://github.com/AliyunContainerService/minikube

```shell
# 从release目录下载最新的minikube版本
chmod +x minikube
mv minikube /usr/local/bin
```

---------

在安装完 minikube 之后，由于高版本，会自带kubectl，如果kubectl在安装minikube的时候一并安装好了

#### 准备本地虚拟化工具 Vmware/VirtualBox

+ 安装 VirtualBox



### 2) 启动minikube

如果指定virtualbox启动

+ minikube start --vm-driver=virtualbox



```
wangjunxiang@My_MacBook_pro_2018  /tmp  minikube start --vm-driver=virtualbox --registry-mirror=https://registry.docker-cn.com

😄  Darwin 10.15.4 上的 minikube v1.9.0
✨  根据现有的配置文件使用 virtualbox 驱动程序
🏃  Updating the running virtualbox "minikube" VM ...
🐳  正在 Docker 19.03.8 中准备 Kubernetes v1.18.0…
🌟  Enabling addons: default-storageclass, storage-provisioner
🏄  完成！kubectl 已经配置至 "minikube"
```



#### 查看minikube集群状态

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  minikube status
m01
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

#### 停止minikube集群

```shell
minikube stop
```

#### 删除minikube集群

```shell
minikube delete
```





### 3) minikube进入k8s集群

**通过查看docker ps发现启动了大量的容器，这些容器就是组成k8s服务的组件**

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)


$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES
0e816cc5c523        4689081edb10           "/storage-provisioner"   51 minutes ago      Up 51 minutes                           k8s_storage-provisioner_storage-provisioner_kube-system_011ef743-aba8-48dd-a952-e735048bfb20_1
ba8feb8a7d35        67da37a9a360           "/coredns -conf /etc…"   51 minutes ago      Up 51 minutes                           k8s_coredns_coredns-66bff467f8-tn7xn_kube-system_5fa0b3a0-38e8-4a21-814f-c71841a456d7_0
d32961d1e895        67da37a9a360           "/coredns -conf /etc…"   51 minutes ago      Up 51 minutes                           k8s_coredns_coredns-66bff467f8-mbsh6_kube-system_32626d95-47ce-4edb-b67c-ad35a761d421_0
a680164c9d53        43940c34f24f           "/usr/local/bin/kube…"   51 minutes ago      Up 51 minutes                           k8s_kube-proxy_kube-proxy-j6j7c_kube-system_73eaa163-d71f-4819-9714-71061057b38d_0
904bf34aee57        k8s.gcr.io/pause:3.2   "/pause"                 51 minutes ago      Up 51 minutes                           k8s_POD_coredns-66bff467f8-tn7xn_kube-system_5fa0b3a0-38e8-4a21-814f-c71841a456d7_0
81b868651a72        k8s.gcr.io/pause:3.2   "/pause"                 51 minutes ago      Up 51 minutes                           k8s_POD_coredns-66bff467f8-mbsh6_kube-system_32626d95-47ce-4edb-b67c-ad35a761d421_0
2d4948acec5c        k8s.gcr.io/pause:3.2   "/pause"                 51 minutes ago      Up 51 minutes                           k8s_POD_storage-provisioner_kube-system_011ef743-aba8-48dd-a952-e735048bfb20_0
c93da269c370        k8s.gcr.io/pause:3.2   "/pause"                 51 minutes ago      Up 51 minutes                           k8s_POD_kube-proxy-j6j7c_kube-system_73eaa163-d71f-4819-9714-71061057b38d_0
db1ab3437ac5        a31f78c7c8ce           "kube-scheduler --au…"   51 minutes ago      Up 51 minutes                           k8s_kube-scheduler_kube-scheduler-minikube_kube-system_5795d0c442cb997ff93c49feeb9f6386_0
fac5e7687946        d3e55153f52f           "kube-controller-man…"   51 minutes ago      Up 51 minutes                           k8s_kube-controller-manager_kube-controller-manager-minikube_kube-system_3016593d20758bbfe68aba26604a8e3d_0
28145c3e8bd0        303ce5db0e90           "etcd --advertise-cl…"   51 minutes ago      Up 51 minutes                           k8s_etcd_etcd-minikube_kube-system_3d83851205ef38fb7489910db37ce6db_0
a215654f4bfa        74060cea7f70           "kube-apiserver --ad…"   51 minutes ago      Up 51 minutes                           k8s_kube-apiserver_kube-apiserver-minikube_kube-system_59a25aa23d3a8b3a2bc6157139d12f6c_0
0416f66abb86        k8s.gcr.io/pause:3.2   "/pause"                 51 minutes ago      Up 51 minutes                           k8s_POD_kube-scheduler-minikube_kube-system_5795d0c442cb997ff93c49feeb9f6386_0
3135821f640a        k8s.gcr.io/pause:3.2   "/pause"                 51 minutes ago      Up 51 minutes                           k8s_POD_kube-controller-manager-minikube_kube-system_3016593d20758bbfe68aba26604a8e3d_0
b77a331398c5        k8s.gcr.io/pause:3.2   "/pause"                 51 minutes ago      Up 51 minutes                           k8s_POD_kube-apiserver-minikube_kube-system_59a25aa23d3a8b3a2bc6157139d12f6c_0
f3de3a849022        k8s.gcr.io/pause:3.2   "/pause"                 51 minutes ago      Up 51 minutes                           k8s_POD_etcd-minikube_kube-system_3d83851205ef38fb7489910db37ce6db_0

$ exit
logout
```





### 4) 通过 kubectl 连接k8s集群

下载kubectl忽略，因为mac在安装好minikube后会顺带安装kubectl

#### 查看kubectl 版本信息

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl version

Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-26T06:16:15Z", GoVersion:"go1.14", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:50:46Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
```

#### 查看集群核心组件：所有namespace

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl get pod --all-namespaces
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-mbsh6           1/1     Running   0          62m
kube-system   coredns-66bff467f8-tn7xn           1/1     Running   0          62m
kube-system   etcd-minikube                      1/1     Running   0          62m
kube-system   kube-apiserver-minikube            1/1     Running   0          62m
kube-system   kube-controller-manager-minikube   1/1     Running   0          62m
kube-system   kube-proxy-j6j7c                   1/1     Running   0          62m
kube-system   kube-scheduler-minikube            1/1     Running   0          62m
kube-system   storage-provisioner                1/1     Running   1          62m
```



### 5) 通过web访问k8s集群



 通过minikube启动dashboard，自动弹出web页面

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  minikube dashboard
🔌  正在开启 dashboard ...
🤔  正在验证 dashboard 运行情况 ...
🚀  Launching proxy ...
🤔  正在验证 proxy 运行状况 ...
🎉  Opening http://127.0.0.1:65078/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...

```

<img src="https://images.gitee.com/uploads/images/2020/0506/185657_3adc9fab_7530643.png" style="zoom:67%;" />







## 2.3 kubeadm部署多节点k8s

搭建3台机器的k8s集群

### 安装

#### 1) 环境准备

系统：centos7

网络：桥接模式

3台host

+ k8s-master
  + ip: 192.168.50.100
  + cpu >= 2  (master节点cpu至少2个)
  + mem >= 2G

+ k8s-node1
  + ip: 192.168.50.101
  + cpu >= 1
  + mem >=2G
+ k8s-node2
  + ip: 192.168.50.102
  + cpu >= 1
  + mem >=2G



Vagrantfile 和 setup.sh同目录

```shell
wangjunxiang@My_MacBook_pro_2018  ~/data/ISO/VirtualBox_VMs/k8s-1  ll
total 16
-rw-r--r--  1 wangjunxiang  staff  1084  5  6 19:26 Vagrantfile
-rw-r--r--  1 wangjunxiang  staff  1499  5  6 19:28 setup.sh
```



##### Vagrantfile

Vagrantfile初始化文件，通过virtualbox接口自动生成三台配置好的虚拟机，并安装好centos7之后，挨个手动安装

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.0"

boxes = [
    {
        :name => "k8s-master",
        :eth1 => "192.168.50.100",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-node1",
        :eth1 => "192.168.50.101",
        :mem => "2048",
        :cpu => "1"
    },
    {
        :name => "k8s-node2",
        :eth1 => "192.168.50.102",
        :mem => "2048",
        :cpu => "1"
    }

]

Vagrant.configure(2) do |config|

  config.vm.box = "centos/7"
  boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.hostname = opts[:name]
      config.vm.provider "vmware_fusion" do |v|
        v.vmx["memsize"] = opts[:mem]
        v.vmx["numvcpus"] = opts[:cpu]
      end
      config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--memory", opts[:mem]]
        v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
      end
      config.vm.network :public_network, ip: opts[:eth1]
    end
  end
  #config.vm.provision "shell", privileged: true, path: "./setup.sh"
end
```

##### setup

root下yum安装docker，yum安装k8s

```shell
#/bin/sh

# 关闭防火墙和selinux，install docker ,some tools
systemctl stop firewalld.service && systemctl disable firewalld.service && sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config && setenforce 0 && yum -y install wget curl && curl -o /etc/yum.repos.d/Centos-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo && curl -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo && yum install -y yum-utils device-mapper-persistent-data lvm2 vim telnet bind-utils && yum-config-manager  --add-repo  https://download.docker.com/linux/centos/docker-ce.repo && yum install -y docker-ce docker-ce-cli containerd.io && systemctl enable docker && systemctl daemon-reload && systemctl restart docker  && mkdir -p /etc/docker

[ ! $(getent group docker) ] && sudo groupadd docker && sudo gpasswd -a $USER docker || echo "docker user group already exists" 

cat <<EOF >  /etc/docker/daemon.json
{
  "registry-mirrors" : [
    "https://8xpk5wnt.mirror.aliyuncs.com",
    "https://dockerhub.azk8s.cn",
    "https://registry.docker-cn.com",
    "https://ot2k4d59.mirror.aliyuncs.com/"
  ]
}
EOF

systemctl restart docker



# open password auth for backup if ssh key doesn't work, bydefault, username=vagrant password=vagrant
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config && systemctl restart sshd

#阿里云的k8s源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF


# install kubeadm, kubectl, and kubelet.
sudo yum install -y kubelet kubeadm kubectl

# 配置网路
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
EOF

sysctl --system

#关闭swap
swapoff -a

systemctl enable kubelet.service
```



#### 2) 安装部署

```shell
wangjunxiang@My_MacBook_pro_2018  ~/data/ISO/VirtualBox_VMs/k8s-1  vagrant up
............
.......
==> k8s-master: Available bridged network interfaces:
1) en0: Wi-Fi (Wireless)
2) en5: USB Ethernet(?)
3) p2p0
4) awdl0
5) llw0
6) en2: 雷雳2
7) en1: 雷雳1
8) en3: 雷雳3
9) en4: 雷雳4
10) bridge0
==> k8s-master: When choosing an interface, it is usually the one that is
==> k8s-master: being used to connect to the internet.
==> k8s-master:
    k8s-master: Which interface should the network bridge to? 1    # 选择每一台机器都是连接 en0的网桥 1
```



#### 3) 验证查看k8s版本

```shell
which kubeadm && which kubelet && which kubectl
```

```shell
[root@k8s-master vagrant]# kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2", GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean", BuildDate:"2020-04-16T11:56:40Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2", GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean", BuildDate:"2020-04-16T11:48:36Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```





### 部署k8s集群

#### master节点初始化(kubeadm)

`--pod-network-cidr`

+ 集群中创建pod使用的网段

`--apiserver-advertise-address`

+ 对外宣告APIserver地址，一般是master的IP地址

`--image-repository`

+ 指定阿里云镜像地址  registry.aliyuncs.com/google_containers

```shell
[root@k8s-master ~]# kubeadm init --pod-network-cidr 172.100.0.0/16 --apiserver-advertise-address 192.168.50.100 --image-repository registry.aliyuncs.com/google_containers
............
.........
.....
..
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

####################### 先执行安装成功提示命令 #######################
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

####################### 再执行下条 #######################
# 在work节点运行，将节点加入k8s集群成为worker
kubeadm join 192.168.50.100:6443 --token g6oc9q.7tr2lqmpa1m4yn6n  --discovery-token-ca-cert-hash sha256:4f6aa49affd476924360e056d274d65d70ba988d3cd482cdc68e0103dabd7721
```

先执行

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

查看master起的pod，2个pending状态，保证下面5个组件running

```shell
[root@k8s-master ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
# 由于网络插件没安装所以是pending状态
kube-system   coredns-7ff77c879f-fvlm7             0/1     Pending   0          6m47s
kube-system   coredns-7ff77c879f-szws6             0/1     Pending   0          6m47s
# 保证下面5个组件是running
kube-system   etcd-k8s-master                      1/1     Running   0          7m1s
kube-system   kube-apiserver-k8s-master            1/1     Running   0          7m1s
kube-system   kube-controller-manager-k8s-master   1/1     Running   0          7m1s
kube-system   kube-proxy-vm7db                     1/1     Running   0          6m47s
kube-system   kube-scheduler-k8s-master            1/1     Running   0          7m1s
```

安装网络插件

```shell
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

网络插件安装后，此时查看master起的pod

```shell
[root@k8s-master vagrant]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
# 网络插件安装后dns组件准备好
kube-system   coredns-7ff77c879f-fvlm7             1/1     Running   0          11m
kube-system   coredns-7ff77c879f-szws6             1/1     Running   0          11m
#
kube-system   etcd-k8s-master                      1/1     Running   0          11m
kube-system   kube-apiserver-k8s-master            1/1     Running   0          11m
kube-system   kube-controller-manager-k8s-master   1/1     Running   0          11m
kube-system   kube-proxy-vm7db                     1/1     Running   0          11m
kube-system   kube-scheduler-k8s-master            1/1     Running   0          11m
# 网络插件安装后新增weave-net的pod
kube-system   weave-net-qb6ns                      2/2     Running   0          52s
```



#### 新增worker节点(kubeadm)

在worker1和2上运行

```shell
sudo kubeadm join 192.168.50.100:6443 --token g6oc9q.7tr2lqmpa1m4yn6n  --discovery-token-ca-cert-hash sha256:4f6aa49affd476924360e056d274d65d70ba988d3cd482cdc68e0103dabd7721
```



在master节点验证，worker节点会逐渐从 NotReady 变为 Ready

```shell
[root@k8s-master vagrant]# kubectl get nodes
NAME         STATUS     ROLES    AGE   VERSION
k8s-master   Ready      master   15m   v1.18.2
k8s-node1    NotReady   <none>   61s   v1.18.2
k8s-node2    Ready      <none>   68s   v1.18.2

[root@k8s-master vagrant]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-fvlm7             1/1     Running   0          16m
kube-system   coredns-7ff77c879f-szws6             1/1     Running   0          16m
kube-system   etcd-k8s-master                      1/1     Running   0          16m
kube-system   kube-apiserver-k8s-master            1/1     Running   0          16m
kube-system   kube-controller-manager-k8s-master   1/1     Running   0          16m
# 新增worker节点，集群增加了更多的proxy
kube-system   kube-proxy-4g78p                     1/1     Running   0          105s
kube-system   kube-proxy-cf996                     1/1     Running   0          112s
kube-system   kube-proxy-vm7db                     1/1     Running   0          16m
kube-system   kube-scheduler-k8s-master            1/1     Running   0          16m
# 新增worker节点，集群增加了更多的weave-net
kube-system   weave-net-bns2b                      2/2     Running   1          105s
kube-system   weave-net-k7xph                      2/2     Running   0          112s
kube-system   weave-net-qb6ns                      2/2     Running   0          6m6s
```



<img src="https://images.gitee.com/uploads/images/2020/0506/211039_e7f91552_7530643.png" style="zoom:80%;" />







# 3 k8s基本概念和操作

## 1) kubectl 配置

环境：之前通过kubeadm搭建3节点集群环境

master上查看集群节点状态

```shell
[root@k8s-master ~]# kubectl get node
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   34m   v1.18.2
k8s-node1    Ready    <none>   19m   v1.18.2
k8s-node2    Ready    <none>   19m   v1.18.2
```

### 1.1 kubectl通过mac访问k8s集群

#### 查看mac的minikube配置文件

+ 当mac生成minikube单节点k8s集群的时候，会在mac系统~/.kube/config生成配置文件

+ 这样kubectl可以在mac直接访问minikube的k8s集群

```yaml
wangjunxiang@My_MacBook_pro_2018  ~  less ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/wangjunxiang/.minikube/ca.crt
    server: https://192.168.99.101:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/wangjunxiang/.minikube/profiles/minikube/client.crt
    client-key: /Users/wangjunxiang/.minikube/profiles/minikube/client.key
```

+ 如果让mac系统不ssh虚拟机访问多节点k8s集群，需要再master节点的~/.kube/config 配置文件 和 mac的~/.kube/config 结合

#### 查看k8s-master节点的配置文件

查看

```shell
[root@k8s-master ~]# cat .kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EVXdOakV5TkRjeU5sb1hEVE13TURVd05ERXlORGN5Tmxvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTHVBCmNnVWlJcGp2NUZIdFUzWFVRbzVGNjA1bmxlN0NkOVNWS2dnSndUSkZPRWlPY1F1RFR2SzMzL3oxZWlyTXBCbC8KZlhTc1p4Q0xCQkh3RG5jQ0JvWXZiQW9QanpvOGthV3VFa3JEWm10ZkhXOC9rN1VJNDR6Nk1Oak52NFFEdmF2QwprVWNra051Z3FQV1dGWFBFcmF6dUw0bTZnWFVkVVROdjZXVkg0RkJPT1ZUc29Cczl2bGV5a3ZBdTJoT2tDdklQCnRmY1d1MUxzdjhRMmMyVmdJbjRRMThmTjNFNnh3ajRkUTdZTkNINW8wbU5HWEdIYkRSSHpmSVA5WnlPMmRWY2wKRWdXMm5PanZMeGNoaytKTTh5aVVJTGc1K1BmM1hhQ1ZVNkdqalRoa2d0OUh4SkkxbEdEc3k4SXBRUkhacmVaZwpQOFVMcXhlbFBUTU9XU0ZKZ24wQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFLTFRETVdmUlcrSXp0K1pXa2lxYktHdGszNW0KdGtUODY4cGFEaTlsYlVJRXYwVDEvMUVzODVzV2pLQ3gwMUREeEJTRFhxdHdNWnhnSzJDODQ2eXZjZzluWHFvQwpZV2pKd3ZVVm8wM0c1NUFLQWhrbXUwTy83alFNenl5MmFydzgwOGxzY0VOT1dZelVUblJuOExzQ0V0d0JTWkZMCi8xbW5GeUJVcnBhMUU1MW55M251TDY3eGVuQVVWeW9jbFk5WFUveUtUNytFRkZudHBIWEdsa2YxRUpObzhwclgKTUpUTWp3bkNWVlI4RW0vOUttRTdNMG5IYWxzVVpWZDVjckV3NVpxOTdBVmVCcUpyT0Jmb0dXcXZnNFRRUWRhVApTNVM2WC9Lb3dUSHdkZTFQNThnMzhEYjY4MjFrdjZwU2IzRmtBeXJSbENHcGdlRzdQWFVtemcrTnQ5ST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://192.168.50.100:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQWRxZ0F3SUJBZ0lJRzdSNVBmYnE4Tll3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMU1EWXhNalEzTWpaYUZ3MHlNVEExTURZeE1qUTNNamhhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXZxU3cyS0xESDVTelVTa1YKS3pybkFCa1BxdlZPbGtKOVJkRnREUjhBbk16L1lCL0MzU3lIY2FaOWF3SGNkUlpDNVZBWVZ5M1FCV29XUWphegpiMy95eW1aUDE0dC9jNVFBS0xUVHFPczBrZGszK3RrQUtqTUZTVTl4NHVyaHF1QldOYithdU9HZnppcUkxYVRHCkpIL0hqLzZtd2dRbzk2ejRyMGE5bnE0dG5FY1RuamZHU2tPQUhqd2xNaHVLRjJxMjJFQ1oxZUcrKzVncFVWSTQKZUZKcUhWMkNMWUQzbGdOMWZnSkt1Y0YvN1NTaU1uVmxBUndLQWpKQU1vRGg0NDcrNUM3WEs4bGh5amd5WDF5OApwY2hLUEVxWlhVNS9EcFBFa1h4MDRwN3R1eWtVbklPK21LNU15eXY4Q3VTd1BDdlE2dXVSUDBvcm9JY2E4V0VXClFkZDJId0lEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFHOUFQRUsxbHRDK1UyaWVrdStITFZ4V0kxK2RKcU1uSTJ5MQpqUkJ3OHdsR1A4a0h0N1I4TjdjblBuakVsTGhaOEdDUm1iTE5ZMU5DOWIrcFhxT0hRL0Z6YlF4YTJlOHBjMWNLCktrN3BsT3h5aEIyT2ZSOVNVQXM3RHZPWXhWRHBzYjhMeCswWDk5R0JaT0J6a2hwSXZYTC9LeVZCbGtrOWt6NVYKSmJPT3VvNWJCWDQ4bEdCWlBxbjlscjlDeHlCWndoU2hnQTVIOEU1alJFbit3cGNPVGVPSHc3YU9UQVNjOGpyQgo1MmdRZWQ0OFAzeXo5dldKdlZmR2dWZFlNZUloN0dkS0E1b3p0ZXcwaWVGNk1WeGluTkFxbElJeFhVNXAwNW5lCi9EUko3cFhZNE0zRnVGQ0poOVlKU21RMTVBVTFkdjFwNTgrUkpJZWEwb2lENnRCODJEWT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBdnFTdzJLTERINVN6VVNrVkt6cm5BQmtQcXZWT2xrSjlSZEZ0RFI4QW5Nei9ZQi9DCjNTeUhjYVo5YXdIY2RSWkM1VkFZVnkzUUJXb1dRamF6YjMveXltWlAxNHQvYzVRQUtMVFRxT3Mwa2RrMyt0a0EKS2pNRlNVOXg0dXJocXVCV05iK2F1T0dmemlxSTFhVEdKSC9Iai82bXdnUW85Nno0cjBhOW5xNHRuRWNUbmpmRwpTa09BSGp3bE1odUtGMnEyMkVDWjFlRysrNWdwVVZJNGVGSnFIVjJDTFlEM2xnTjFmZ0pLdWNGLzdTU2lNblZsCkFSd0tBakpBTW9EaDQ0Nys1QzdYSzhsaHlqZ3lYMXk4cGNoS1BFcVpYVTUvRHBQRWtYeDA0cDd0dXlrVW5JTysKbUs1TXl5djhDdVN3UEN2UTZ1dVJQMG9yb0ljYThXRVdRZGQySHdJREFRQUJBb0lCQVFDTkdxS1dSYW44WnZodQpHdVZETVA2bkVPV0syTFFJL1Q5eGZMZWxYWXY3Z3JPRjl4d296ZnVWLysrV1V6TlVLbHpyRTJSZ3FsVHNuUC9LCmxHZ3RIOXVaT1M3aFQ2dk81UDFWSTdvQnJjMGtJazJQeWl1WUVGbGFVSVh2dVgrOEZQMWFITzRzNXpwN3d2bXkKZjVuMGkrc3VSZ0l4V2hqa2NNOUVGQ3puRk81SFd3QU5hVGZDcm53MHVKWTc0KzRnaG1wSVNCeVV3QnB5RUtJVQpXQkROeXpQN2VWSFFpbmJZd2dzazA5QkFKaFpxMTd4WlpLZjk5bWlub2pyRXYrYmtXaHhmRjFITnlDT2IyYkhhCmpnRTBValBlazhZME92T2Z3Q3pvVk5OUFV1WmlVWm5xVG4rdkJ4ckV2Yk91cjV5TDQwVWtPTDVKOE9PckJhTWwKSGpNZGtKQlpBb0dCQVBDbW5CaThSbWhBdDF6WExaWkk4SWpUaENZZW5zK0ZGZjZaM0FOMFU1N1BNWnluVEtNMQpFZHJlM3RkcmxjOGZxejgrVW1TVkNnczBZaVR3MWdKRkNrMXBzN2JoUnJXZzJ0MzQybTdaRmJaVjJjRUJ0cDArCnQ0YmlObDdkY0swOTNXTEtsWWJZSFRmOXlBK0hCQzlWZkV2R1Z3YXdKMUc2cEpKK3Z4OWs5ajlsQW9HQkFNck4KakxHbjVucTJwU0p1T1RkSjVzQlhYQ2hwSWpxcGt2R014L2Q2eXpUNFBDSmF2Zk9FbHRSNXFaeVExSUhWSENyZgprZWNpU3ZLamprMnBzaDNDOEYrVnhJbTNNK0w0S2xBVHBzOXREMnVKejNrMVJ2VFFGcHVNbDBNemJWVEpMLzhnCmJBT3lVdGxZaHJUTUx3Ulk3amtVWmxhcWtCVmZJVytKOG41cDVMRXpBb0dCQU40TGZRQTl5R0V0UllMK2NHdTQKaFdoYWNoYVNMa3FnSzdrdDBobkYzZG9zcDBRNkFiYWRvd25tbG9zQ0U4cDNHQlZVdGNWazEwMmJXZXRuNUs5WApjTGdaRGQ5eVlVSDVWN2wwZ21mQkdnMlJqVWhQQW1aNGxmSjVDMTNneUxTdzNuTG5KYXl6LzlISDhpNlJqOFQxCkJha05Ld05heWd6WlFEeURnbW0vU0k4WkFvR0FZWWRhV3ZGdzBLRFhaMmgwa1pjenBsb2MwcjVFbk11Q0JESDgKMUpJeitVUkx2d1craGNiRXRtZlAzcUJ6NGdBM3JKS1oydnJONmtRbm9rZXloY2VDeTZUOXdIRGZQZzYwWXBBZwp6MjRXVlZRUDk2MWNjMDFESWdrSmtXTEErRjVNZTdmKzJnUSsvWkVxTHZabXdnTjJoUEsvaUh3OEVGc1FmRkJ0CjBzZGdHS3NDZ1lFQXRRU2JXN2c4NFNWOTN3bVNMYjJEd0J5REYweXAvN0hLNS9RQkhJb0g2WHp2U05OTUVSS0cKdzJNRVZjZFV3Q21KT3Q1cmNvampnMndZYllaUTNqbUl2am5XV01UdGlFOStEaUF2ZGFUVkJObUwzWjRyUzc3agpXMHFOaW5YZWE1cmhRTGYrdVFIWHdtNEVSSWxOREZxNkNLem9FNmR6NTBMNmhNcWhLclFjQy9RPQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```



#### 修改mac本地配置文件~/.kube/config

将minikube k8s集群和多节点k8s集群都配置在本机mac的配置文件中

+ 多节点k8s集群的集群名将kubernetes改为了kubeadm

```shell
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/wangjunxiang/.minikube/ca.crt
    server: https://192.168.99.101:8443
  name: minikube
# 将k8s的master集群 cluster信息复制到这里
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EVXdOakV5TkRjeU5sb1hEVE13TURVd05ERXlORGN5Tmxvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTHVBCmNnVWlJcGp2NUZIdFUzWFVRbzVGNjA1bmxlN0NkOVNWS2dnSndUSkZPRWlPY1F1RFR2SzMzL3oxZWlyTXBCbC8KZlhTc1p4Q0xCQkh3RG5jQ0JvWXZiQW9QanpvOGthV3VFa3JEWm10ZkhXOC9rN1VJNDR6Nk1Oak52NFFEdmF2QwprVWNra051Z3FQV1dGWFBFcmF6dUw0bTZnWFVkVVROdjZXVkg0RkJPT1ZUc29Cczl2bGV5a3ZBdTJoT2tDdklQCnRmY1d1MUxzdjhRMmMyVmdJbjRRMThmTjNFNnh3ajRkUTdZTkNINW8wbU5HWEdIYkRSSHpmSVA5WnlPMmRWY2wKRWdXMm5PanZMeGNoaytKTTh5aVVJTGc1K1BmM1hhQ1ZVNkdqalRoa2d0OUh4SkkxbEdEc3k4SXBRUkhacmVaZwpQOFVMcXhlbFBUTU9XU0ZKZ24wQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFLTFRETVdmUlcrSXp0K1pXa2lxYktHdGszNW0KdGtUODY4cGFEaTlsYlVJRXYwVDEvMUVzODVzV2pLQ3gwMUREeEJTRFhxdHdNWnhnSzJDODQ2eXZjZzluWHFvQwpZV2pKd3ZVVm8wM0c1NUFLQWhrbXUwTy83alFNenl5MmFydzgwOGxzY0VOT1dZelVUblJuOExzQ0V0d0JTWkZMCi8xbW5GeUJVcnBhMUU1MW55M251TDY3eGVuQVVWeW9jbFk5WFUveUtUNytFRkZudHBIWEdsa2YxRUpObzhwclgKTUpUTWp3bkNWVlI4RW0vOUttRTdNMG5IYWxzVVpWZDVjckV3NVpxOTdBVmVCcUpyT0Jmb0dXcXZnNFRRUWRhVApTNVM2WC9Lb3dUSHdkZTFQNThnMzhEYjY4MjFrdjZwU2IzRmtBeXJSbENHcGdlRzdQWFVtemcrTnQ5ST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://192.168.50.100:6443
  # 将kubernetes改为kubeadm
  name: kubeadm  
#################
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
# 将k8s的master集群 context信息复制到这里
- context:
    # 将kubernetes改为kubeadm
    cluster: kubeadm
    user: kubernetes-admin
    # kubernetes-admin@xxx 改为kubeadm
  name: kubeadm
###############
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/wangjunxiang/.minikube/profiles/minikube/client.crt
    client-key: /Users/wangjunxiang/.minikube/profiles/minikube/client.key
# 将k8s的master集群 user信息复制到这里
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQWRxZ0F3SUJBZ0lJRzdSNVBmYnE4Tll3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMU1EWXhNalEzTWpaYUZ3MHlNVEExTURZeE1qUTNNamhhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXZxU3cyS0xESDVTelVTa1YKS3pybkFCa1BxdlZPbGtKOVJkRnREUjhBbk16L1lCL0MzU3lIY2FaOWF3SGNkUlpDNVZBWVZ5M1FCV29XUWphegpiMy95eW1aUDE0dC9jNVFBS0xUVHFPczBrZGszK3RrQUtqTUZTVTl4NHVyaHF1QldOYithdU9HZnppcUkxYVRHCkpIL0hqLzZtd2dRbzk2ejRyMGE5bnE0dG5FY1RuamZHU2tPQUhqd2xNaHVLRjJxMjJFQ1oxZUcrKzVncFVWSTQKZUZKcUhWMkNMWUQzbGdOMWZnSkt1Y0YvN1NTaU1uVmxBUndLQWpKQU1vRGg0NDcrNUM3WEs4bGh5amd5WDF5OApwY2hLUEVxWlhVNS9EcFBFa1h4MDRwN3R1eWtVbklPK21LNU15eXY4Q3VTd1BDdlE2dXVSUDBvcm9JY2E4V0VXClFkZDJId0lEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFHOUFQRUsxbHRDK1UyaWVrdStITFZ4V0kxK2RKcU1uSTJ5MQpqUkJ3OHdsR1A4a0h0N1I4TjdjblBuakVsTGhaOEdDUm1iTE5ZMU5DOWIrcFhxT0hRL0Z6YlF4YTJlOHBjMWNLCktrN3BsT3h5aEIyT2ZSOVNVQXM3RHZPWXhWRHBzYjhMeCswWDk5R0JaT0J6a2hwSXZYTC9LeVZCbGtrOWt6NVYKSmJPT3VvNWJCWDQ4bEdCWlBxbjlscjlDeHlCWndoU2hnQTVIOEU1alJFbit3cGNPVGVPSHc3YU9UQVNjOGpyQgo1MmdRZWQ0OFAzeXo5dldKdlZmR2dWZFlNZUloN0dkS0E1b3p0ZXcwaWVGNk1WeGluTkFxbElJeFhVNXAwNW5lCi9EUko3cFhZNE0zRnVGQ0poOVlKU21RMTVBVTFkdjFwNTgrUkpJZWEwb2lENnRCODJEWT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBdnFTdzJLTERINVN6VVNrVkt6cm5BQmtQcXZWT2xrSjlSZEZ0RFI4QW5Nei9ZQi9DCjNTeUhjYVo5YXdIY2RSWkM1VkFZVnkzUUJXb1dRamF6YjMveXltWlAxNHQvYzVRQUtMVFRxT3Mwa2RrMyt0a0EKS2pNRlNVOXg0dXJocXVCV05iK2F1T0dmemlxSTFhVEdKSC9Iai82bXdnUW85Nno0cjBhOW5xNHRuRWNUbmpmRwpTa09BSGp3bE1odUtGMnEyMkVDWjFlRysrNWdwVVZJNGVGSnFIVjJDTFlEM2xnTjFmZ0pLdWNGLzdTU2lNblZsCkFSd0tBakpBTW9EaDQ0Nys1QzdYSzhsaHlqZ3lYMXk4cGNoS1BFcVpYVTUvRHBQRWtYeDA0cDd0dXlrVW5JTysKbUs1TXl5djhDdVN3UEN2UTZ1dVJQMG9yb0ljYThXRVdRZGQySHdJREFRQUJBb0lCQVFDTkdxS1dSYW44WnZodQpHdVZETVA2bkVPV0syTFFJL1Q5eGZMZWxYWXY3Z3JPRjl4d296ZnVWLysrV1V6TlVLbHpyRTJSZ3FsVHNuUC9LCmxHZ3RIOXVaT1M3aFQ2dk81UDFWSTdvQnJjMGtJazJQeWl1WUVGbGFVSVh2dVgrOEZQMWFITzRzNXpwN3d2bXkKZjVuMGkrc3VSZ0l4V2hqa2NNOUVGQ3puRk81SFd3QU5hVGZDcm53MHVKWTc0KzRnaG1wSVNCeVV3QnB5RUtJVQpXQkROeXpQN2VWSFFpbmJZd2dzazA5QkFKaFpxMTd4WlpLZjk5bWlub2pyRXYrYmtXaHhmRjFITnlDT2IyYkhhCmpnRTBValBlazhZME92T2Z3Q3pvVk5OUFV1WmlVWm5xVG4rdkJ4ckV2Yk91cjV5TDQwVWtPTDVKOE9PckJhTWwKSGpNZGtKQlpBb0dCQVBDbW5CaThSbWhBdDF6WExaWkk4SWpUaENZZW5zK0ZGZjZaM0FOMFU1N1BNWnluVEtNMQpFZHJlM3RkcmxjOGZxejgrVW1TVkNnczBZaVR3MWdKRkNrMXBzN2JoUnJXZzJ0MzQybTdaRmJaVjJjRUJ0cDArCnQ0YmlObDdkY0swOTNXTEtsWWJZSFRmOXlBK0hCQzlWZkV2R1Z3YXdKMUc2cEpKK3Z4OWs5ajlsQW9HQkFNck4KakxHbjVucTJwU0p1T1RkSjVzQlhYQ2hwSWpxcGt2R014L2Q2eXpUNFBDSmF2Zk9FbHRSNXFaeVExSUhWSENyZgprZWNpU3ZLamprMnBzaDNDOEYrVnhJbTNNK0w0S2xBVHBzOXREMnVKejNrMVJ2VFFGcHVNbDBNemJWVEpMLzhnCmJBT3lVdGxZaHJUTUx3Ulk3amtVWmxhcWtCVmZJVytKOG41cDVMRXpBb0dCQU40TGZRQTl5R0V0UllMK2NHdTQKaFdoYWNoYVNMa3FnSzdrdDBobkYzZG9zcDBRNkFiYWRvd25tbG9zQ0U4cDNHQlZVdGNWazEwMmJXZXRuNUs5WApjTGdaRGQ5eVlVSDVWN2wwZ21mQkdnMlJqVWhQQW1aNGxmSjVDMTNneUxTdzNuTG5KYXl6LzlISDhpNlJqOFQxCkJha05Ld05heWd6WlFEeURnbW0vU0k4WkFvR0FZWWRhV3ZGdzBLRFhaMmgwa1pjenBsb2MwcjVFbk11Q0JESDgKMUpJeitVUkx2d1craGNiRXRtZlAzcUJ6NGdBM3JKS1oydnJONmtRbm9rZXloY2VDeTZUOXdIRGZQZzYwWXBBZwp6MjRXVlZRUDk2MWNjMDFESWdrSmtXTEErRjVNZTdmKzJnUSsvWkVxTHZabXdnTjJoUEsvaUh3OEVGc1FmRkJ0CjBzZGdHS3NDZ1lFQXRRU2JXN2c4NFNWOTN3bVNMYjJEd0J5REYweXAvN0hLNS9RQkhJb0g2WHp2U05OTUVSS0cKdzJNRVZjZFV3Q21KT3Q1cmNvampnMndZYllaUTNqbUl2am5XV01UdGlFOStEaUF2ZGFUVkJObUwzWjRyUzc3agpXMHFOaW5YZWE1cmhRTGYrdVFIWHdtNEVSSWxOREZxNkNLem9FNmR6NTBMNmhNcWhLclFjQy9RPQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
##############
```

#### mac验证配置文件是否加载

+ 已经获取到2个集群

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  cd ~/.kube
wangjunxiang@My_MacBook_pro_2018  ~/.kube  kubectl config get-contexts
CURRENT   NAME                 CLUSTER    AUTHINFO           NAMESPACE
          kubeadm@kubernetes   kubeadm    kubernetes-admin
*         minikube             minikube   minikube
```



### 1.2 kubectl mac下使用

#### kubectl切换集群

+ 配置文件下加载这minikube和kubeadm2个集群



`kubectl config get-contexts`  查看集群 *号在minikube代表当前连接minikube集群

```shell
wangjunxiang@My_MacBook_pro_2018  ~/.kube  kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO           NAMESPACE
          kubeadm    kubeadm    kubernetes-admin
*         minikube   minikube   minikube
```

`kubectl get node` 查看当前连接的集群 minikube信息

```shell
wangjunxiang@My_MacBook_pro_2018  ~/.kube  kubectl get node
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   28m   v1.18.0
```

`kubectl config use-context kubeadm` 切换连接的集群 minikube 为 kubeadm

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl config use-context kubeadm
Switched to context "kubeadm".
```

切换到多节点集群kubeadm之后，就可以直接在mac用命令查看虚拟机3节点k8s信息了

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl get node
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   82m   v1.18.2
k8s-node1    Ready    <none>   67m   v1.18.2
k8s-node2    Ready    <none>   67m   v1.18.2
```



## 2) k8s节点和标签



`kubectl describe node k8s-master` 查看单个节点 k8s-master 的详细信息

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl describe node k8s-master
Name:               k8s-master
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s-master
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 06 May 2020 20:47:48 +0800
Taints:             node-role.kubernetes.io/master:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  k8s-master
  AcquireTime:     <unset>
  RenewTime:       Wed, 06 May 2020 22:13:35 +0800
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Wed, 06 May 2020 20:58:45 +0800   Wed, 06 May 2020 20:58:45 +0800   WeaveIsUp                    Weave pod has set this
  MemoryPressure       False   Wed, 06 May 2020 22:09:40 +0800   Wed, 06 May 2020 20:47:44 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Wed, 06 May 2020 22:09:40 +0800   Wed, 06 May 2020 20:47:44 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Wed, 06 May 2020 22:09:40 +0800   Wed, 06 May 2020 20:47:44 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Wed, 06 May 2020 22:09:40 +0800   Wed, 06 May 2020 20:58:54 +0800   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  10.0.2.15
  Hostname:    k8s-master
Capacity:
  cpu:                2
  ephemeral-storage:  41921540Ki
  hugepages-2Mi:      0
  memory:             1882144Ki
  pods:               110
Allocatable:
  cpu:                2
  ephemeral-storage:  38634891201
  hugepages-2Mi:      0
  memory:             1779744Ki
  pods:               110
System Info:
  Machine ID:                 3045291a94e3084fb518d21e9d0b3131
  System UUID:                3045291A-94E3-084F-B518-D21E9D0B3131
  Boot ID:                    01e2dc8f-0836-40c3-9e9e-35c316ae1970
  Kernel Version:             3.10.0-957.12.2.el7.x86_64
  OS Image:                   CentOS Linux 7 (Core)
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://19.3.8
  Kubelet Version:            v1.18.2
  Kube-Proxy Version:         v1.18.2
PodCIDR:                      172.100.0.0/24
PodCIDRs:                     172.100.0.0/24
Non-terminated Pods:          (8 in total)
  Namespace                   Name                                  CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                   ----                                  ------------  ----------  ---------------  -------------  ---
  kube-system                 coredns-7ff77c879f-fvlm7              100m (5%)     0 (0%)      70Mi (4%)        170Mi (9%)     16h
  kube-system                 coredns-7ff77c879f-szws6              100m (5%)     0 (0%)      70Mi (4%)        170Mi (9%)     16h
  kube-system                 etcd-k8s-master                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         16h
  kube-system                 kube-apiserver-k8s-master             250m (12%)    0 (0%)      0 (0%)           0 (0%)         16h
  kube-system                 kube-controller-manager-k8s-master    200m (10%)    0 (0%)      0 (0%)           0 (0%)         16h
  kube-system                 kube-proxy-vm7db                      0 (0%)        0 (0%)      0 (0%)           0 (0%)         16h
  kube-system                 kube-scheduler-k8s-master             100m (5%)     0 (0%)      0 (0%)           0 (0%)         16h
  kube-system                 weave-net-qb6ns                       20m (1%)      0 (0%)      0 (0%)           0 (0%)         16h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                770m (38%)  0 (0%)
  memory             140Mi (8%)  340Mi (19%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:              <none>

```



kubectl get node -o wide	

+ -o yaml	输出yaml格式信息
+ -o json     json格式

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl get node -o wide
NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
k8s-master   Ready    master   89m   v1.18.2   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-957.12.2.el7.x86_64   docker://19.3.8
k8s-node1    Ready    <none>   75m   v1.18.2   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-957.12.2.el7.x86_64   docker://19.3.8
k8s-node2    Ready    <none>   75m   v1.18.2   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-957.12.2.el7.x86_64   docker://19.3.8
```



### 设置label标签



给`k8s-master`节点设置标签（key=value格式）

`kubectl label node k8s-master env=test`

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl label node k8s-master env=test
node/k8s-master labeled
```



查看各个节点label标签 ，查看设置的env=test标签是否生效 

`kubectl get node --show-labels` 

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl get node --show-labels
NAME         STATUS   ROLES    AGE   VERSION   LABELS
k8s-master   Ready    master   96m   v1.18.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,env=test,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/master=
k8s-node1    Ready    <none>   81m   v1.18.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux
k8s-node2    Ready    <none>   81m   v1.18.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node2,kubernetes.io/os=linux

```



删除`k8s-master`节点的标签 (删除key-) 带'-'符号

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl label node k8s-master env-
node/k8s-master labeled
```



### 设置label节点ROLES

查看节点roles

```
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl get node
NAME         STATUS   ROLES    AGE    VERSION
k8s-master   Ready    master   100m   v1.18.2
k8s-node1    Ready    <none>   85m    v1.18.2
k8s-node2    Ready    <none>   85m    v1.18.2

wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl get node --show-labels
NAME         STATUS   ROLES    AGE    VERSION   LABELS
k8s-master   Ready    master   101m   v1.18.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/master=
k8s-node1    Ready    <none>   86m    v1.18.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux
k8s-node2    Ready    <none>   86m    v1.18.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node2,kubernetes.io/os=linux

```

node1 和node2的roles都为空，roles是特殊的label

设置roles

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl label nodes k8s-node1 node-role.kubernetes.io/worker=
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl label nodes k8s-node2 node-role.kubernetes.io/worker=
```

查看，node1和2的roles变为了worker

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl get node
NAME         STATUS   ROLES    AGE    VERSION
k8s-master   Ready    master   102m   v1.18.2
k8s-node1    Ready    worker   88m    v1.18.2
k8s-node2    Ready    worker   88m    v1.18.2
```



## 3) k8s调度最小单位:Pod

### POD

<img src="https://images.gitee.com/uploads/images/2020/0507/132129_1d9d738c_7530643.png" style="zoom:50%;" />

+ 在同一个命名空间或者网络空间的1个或多个容器
  + 相同IP地址
+ 共享资源(volume)的1个或多个容器
+ 在k8s中最小调度单位



### 定义POD

#### 定义pod

+ kind 定义pod
+ spec指明几个containers
+ metadata
  + name 定义这个pod的名字 nginx-busybox

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-busybox
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
  - name: busybox
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
```

#### 创建pod

```shell
# 指定yml文件
kubectl create -f nginx_busybox.yml
```

会指定某个k8s节点下载对应镜像，比较慢

### 对pod操作命令

#### 查询pods

```shell
kubectl get pods
# 获取跟多信息，比如ip地址
kubectl get pods nginx-busybox -o wide
```

#### 获取pod详细信息

```shell
kubectl describe pod nginx-busybox
```

#### 进入pod的指定的容器

```shell
# pod中有2个容器，默认进入第一个容器，-c 指定进入nginx的容器中
kubectl exec nginx-busybox -c nginx -it sh
```

#### 指定pod下某个容器执行命令

+ 根据-c 容器名
+ -- 命令

```shell
kubectl exec  nginx-busybox -c busybox -- date
Wed May  6 16:56:23 UTC 2020
```

```shell
kubectl exec  nginx-busybox -c nginx -- date
Wed May  6 16:56:33 UTC 2020
```

#### 删除pod

通过yml文件

```shell
kubectl delete -f nginx_busybox.yml
```



------------

### 解决网络问题引起的报错：

pod does not exist

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp/1k8s  kubectl exec  nginx-busybox -- date
Defaulting container name to nginx.
Use 'kubectl describe pod/nginx-busybox -n default' to see all of the containers in this pod.
error: unable to upgrade connection: pod does not exist
```

#### 原因分析：

k8s集群3个节点ip都是10网段

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp/1k8s  kubectl get node -o wide
NAME         STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
k8s-master   Ready    master   3h43m   v1.18.2   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-957.12.2.el7.x86_64   docker://19.3.8
k8s-node1    Ready    worker   3h28m   v1.18.2   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-957.12.2.el7.x86_64   docker://19.3.8
k8s-node2    Ready    worker   3h29m   v1.18.2   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-957.12.2.el7.x86_64   docker://19.3.8
```

但是ssh到机器上看到实际还有eth1网段192.168.50.0网段 是和mac客户端同网段，但是调用的是10网段，需要将k8s集群3个网段都改为192.168.50.0网段

<img src="https://images.gitee.com/uploads/images/2020/0507/152110_1511197a_7530643.png" style="zoom:100%;" />



#### 解决办法：

登录k8s3个节点分别吧自己eth1网段(192.168.50.xx)地址，修改各自配置文件/etc/sysconfig/kubelet，并重启3个节点的kubelet服务

master

192.168.50.100

```shell
[root@k8s-master ~]# cat  /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--node-ip=192.168.50.100"
[root@k8s-master ~]# systemctl restart kubelet
```

node1

192.168.50.101

```shell
[root@k8s-node1 ~]# cat  /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--node-ip=192.168.50.101"
[root@k8s-master ~]# systemctl restart kubelet
```

node2

192.168.50.102

```shell
[root@k8s-node2 ~]# cat  /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--node-ip=192.168.50.102"
[root@k8s-master ~]# systemctl restart kubelet
```

此时，在mac客户端查看，解决。

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp/1k8s  kubectl get node -o wide
NAME         STATUS   ROLES    AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
k8s-master   Ready    master   3h47m   v1.18.2   192.168.50.100   <none>        CentOS Linux 7 (Core)   3.10.0-957.12.2.el7.x86_64   docker://19.3.8
k8s-node1    Ready    worker   3h32m   v1.18.2   192.168.50.101   <none>        CentOS Linux 7 (Core)   3.10.0-957.12.2.el7.x86_64   docker://19.3.8
k8s-node2    Ready    worker   3h32m   v1.18.2   192.168.50.102   <none>        CentOS Linux 7 (Core)   3.10.0-957.12.2.el7.x86_64   docker://19.3.8
```































