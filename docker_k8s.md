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

### 2.1 搭建方式介绍

#### 1) minikube工具部署

+ 用途：一般用来 快速本地 开发 测试 用

  + https://github.com/kubernetes/minikube
  + 在本地创建1个node的k8s集群

#### 2) kubeadm 部署

+ 用途：本地搭建 多个节点 的k8s集群

  + https://github.com/kubernetes/kubeadm

#### 3) kops 部署

  + https://github.com/kubernetes/kops
  + 在cloud部署k8s集群
    + 例如在aws的某个regin，搭建2，3个节点的k8s集群



### 2.2 minikube部署单节点k8s

https://minikube.sigs.k8s.io/docs/start/

mac为例：

#### 1) 下载minikube

##### brew安装minikube 或者 curl安装

+ 可以找到阿里云的minikube文件安装



###### 方式1：挂vpn国外安装...(二选一)

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

###### 方式2：阿里云国内安装(后来才知道)

阿里云发布的minikube地址：https://github.com/AliyunContainerService/minikube

```shell
# 从release目录下载最新的minikube版本
chmod +x minikube
mv minikube /usr/local/bin
```

---------

在安装完 minikube 之后，由于高版本，会自带kubectl，如果kubectl在安装minikube的时候一并安装好了

##### 准备本地虚拟化工具 Vmware/VirtualBox

+ 安装 VirtualBox



#### 2) 启动minikube

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



##### 查看minikube集群状态

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  minikube status
m01
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

##### 停止minikube集群

```shell
minikube stop
```

##### 删除minikube集群

```shell
minikube delete
```





#### 3) 进入minikube

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





#### 4) 通过 kubectl 连接k8s集群

下载kubectl忽略，因为mac在安装好minikube后会顺带安装kubectl

##### 查看kubectl 版本信息

```shell
wangjunxiang@My_MacBook_pro_2018  /tmp  kubectl version

Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-26T06:16:15Z", GoVersion:"go1.14", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:50:46Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
```

##### 查看集群核心组件：所有namespace

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



#### 5) 通过web访问k8s集群



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







### 2.3 kubeadm部署多节点k8s

搭建3台机器的k8s集群

#### 安装

##### 1) 环境准备

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



###### Vagrantfile

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

###### setup

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



##### 2) 安装部署

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



##### 3) 验证查看k8s版本

```shell
which kubeadm && which kubelet && which kubectl
```

```shell
[root@k8s-master vagrant]# kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2", GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean", BuildDate:"2020-04-16T11:56:40Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2", GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean", BuildDate:"2020-04-16T11:48:36Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```





#### 部署k8s集群

##### master节点初始化

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



##### 新增worker节点

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







































