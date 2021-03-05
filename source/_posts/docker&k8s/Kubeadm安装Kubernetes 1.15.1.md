---
title: Kubeadm安装Kubernetes 1.15.1
date: 2018-08-07 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: 安装Kubernetes 1.15.1
categories: docker&k8s
tags:
  - dokcer
  - k8s
  - linux
---

# Kubeadm安装Kubernetes 1.15.1

## 一、实验环境准备
1. 服务器虚拟机准备

IP | CPU | 内存 | hostname
:-: | :-: | :-: | :-:
192.168.198.200 | >=2c | >=2G | master
192.168.198.201 | >=2c | >=2G | node1
192.168.198.202 | >=2c | >=2G | node2

本实验我这里用的VM是vmware workstation创建的，我的机器配置较低，所以master给了2G 2C，node每个给了1G 2C，大家根据自己的资源情况，按照上面给的建议最低值创建即可。
注意：hostname不能有大写字母，比如Master这样的。

2. 软件版本
    系统：CentOS7.5.1804
    Kubernetes：1.15.1
    docker版本：18.06.1-ce
3. 环境初始化操作
   3.1 配置hostname
      hostnamectl set-hostname master
      hostnamectl set-hostname node1
      hostnamectl set-hostname node2
    3.2 配置/etc/hosts
      echo "192.168.198.200 master" >> /etc/hosts
      echo "192.168.198.201 node1" >> /etc/hosts
      echo "192.168.198.202 node2" >> /etc/hosts
    3.3 关闭防火墙、selinux、swap
      ```bash
      //停防火墙 
      systemctl stop firewalld
      systemctl disable firewalld
      ```
      ```bash
      //关闭Selinux
      setenforce 0
      sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux
      sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
      sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/sysconfig/selinux
      sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config
      ```

      ```bash
      //关闭Swap
      swapoff –a
      注释掉/etc/fstab的swap行
      # /dev/mapper/centos-swap swap                    swap    defaults        0 0
      ```
      ```bash
      //加载br_netfilter
      modprobe br_netfilter
      ```
      &nbsp;
        3.4 配置内核参数
      ```   bash
      //配置sysctl内核参数 
      cat > /etc/sysctl.d/k8s.conf <<EOF
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      EOF
      //生效文件
      sysctl -p /etc/sysctl.d/k8s.conf
      //修改Linux 资源配置文件，调高ulimit最大打开数和systemctl管理的服务文件最大打开数
      echo "* soft nofile 655360" >> /etc/security/limits.conf 
      echo "* hard nofile 655360" >> /etc/security/limits.conf
      echo "* soft nproc 655360" >> /etc/security/limits.conf
      echo "* hard nproc 655360" >> /etc/security/limits.conf
      echo "* soft memlock unlimited" >> /etc/security/limits.conf
      echo "* hard memlock unlimited" >> /etc/security/limits.conf
      echo "DefaultLimitNOFILE=1024000" >> /etc/systemd/system.conf
      echo "DefaultLimitNPROC=1024000" >> /etc/systemd/system.conf
      ```

4. 配置CentOS YUM源
    ```bash
    //配置国内tencent yum源地址、epel源地址、Kubernetes源地址
    mkdir -p /etc/yum.repo.d/repo.bak
    mv /etc/yum.repo.d/*.repo  /etc/yum.repo.d/repo.bak
    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo
    wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo
    yum clean all && yum makecache

    //配置国内Kubernetes源地址
    cat <<EOF > /etc/yum.repos.d/kubernetes.repo 
        [kubernetes]
        name=Kubernetes baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
        enabled=1
        gpgcheck=1
        repo_gpgcheck=1
        gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg 
        https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF
    ```


5. 安装一些依赖软件包
`yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp bash-completion yum-utils device-mapper-persistent-data lvm2 net-tools conntrack-tools vim libtool-ltdl`



6. 时间同步配置
    ```bash
    yum install chrony –y
    systemctl enable chronyd.service && systemctl start chronyd.service && systemctl status chronyd.service 
    chronyc sources
    ```
    运行date命令看下系统时间，过一会儿时间就会同步。

7. 配置节点间ssh互信
    配置ssh互信，那么节点之间就能无密访问，方便以后操作 
    ```bash
    ssh-keygen  //每台机器执行这个命令， 一路回车即可
    ssh-copy-id node  //到master上拷贝公钥到其他节点，这里需要输入 yes和密码
    ```

8. 以上操作后，全部重启一下。

&nbsp;
&nbsp;

## 二、docker安装
1. 设置docker yum源
`yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

2. 列出docker版本
`yum list docker-ce --showduplicates | sort -r`

3. 安装docker 指定18.06.1
` yum install -y docker-ce-18.06.1.ce-3.el7`
`systemctl start docker`

4. 配置镜像加速器和docker数据存放路径
    ```bash
    tee /etc/docker/daemon.json << EOF
    {
        "registry-mirrors": ["https://q2hy3fzi.mirror.aliyuncs.com"],
        "graph": "/tol/docker-data"
    }
    EOF
    ```
5. 启动docker
`systemctl daemon-reload && systemctl restart docker && systemctl enable docker`

&nbsp;
&nbsp;

## 三、安装kubeadm、kubelet、kubectl（所有节点）
1. 工具说明
 - kubeadm: 部署集群用的命令 
 - kubelet: 在集群中每台机器上都要运行的组件，负责管理pod、容器的生命周期 
 - kubectl: 集群管理工具
2. yum 安装
    ```bash
    //安装工具
    yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
    //启动kubelet
    systemctl enable kubelet && systemctl start kubelet
    ```
    注意：kubelet 服务会暂时启动不了，先不用管它。

&nbsp;
&nbsp;

## 四、镜像下载准备
1. 初始化获取要下载的镜像列表
  使用kubeadm来搭建Kubernetes，那么就需要下载得到Kubernetes运行的对应基础镜像，比如：kube- proxy、kube-apiserver、kube-controller-manager等等 。那么有什么方法可以得知要下载哪些镜像 呢？从kubeadm v1.11+版本开始，增加了一个kubeadm config print-default 命令，可以让我们方便 的将kubeadm的默认配置输出到文件中，这个文件里就包含了搭建K8S对应版本需要的基础配置环境。另 外，我们也可以执行 kubeadm config images list 命令查看依赖需要安装的镜像列表。

    ```bash
    [root@master ]# kubeadm config images list
    W0806 17:29:06.709181  130077 version.go:98] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
    W0806 17:29:06.709254  130077 version.go:99] falling back to the local client version: v1.15.1
    k8s.gcr.io/kube-apiserver:v1.15.1
    k8s.gcr.io/kube-controller-manager:v1.15.1
    k8s.gcr.io/kube-scheduler:v1.15.1
    k8s.gcr.io/kube-proxy:v1.15.1
    k8s.gcr.io/pause:3.1
    k8s.gcr.io/etcd:3.3.10
    k8s.gcr.io/coredns:1.3.1
    ```
  2. 生成默认kubeadm.conf文件
      ```bash
      //执行这个命令就生成了一个kubeadm.conf文件
      kubeadm config print init-defaults > kubeadm.conf
      ```
  3. 绕过墙下载镜像方法
    这个配置文件默认会从google的镜像仓库地址k8s.gcr.io下载镜像，下载不了。因此，我们通过下面的方法把地址改成国内的，比如用阿里的：
      ```bash
      vim kubeadm.conf
      ...
      imageRepository: registry.aliyuncs.com/google_containers   //地址
      kind: ClusterConfiguration
      kubernetesVersion: v1.15.1   //版本
      ...
      ```

  4. 下载需要用到的镜像
    kubeadm.conf修改好后，我们执行下面命令就可以自动从国内下载需要用到的镜像了
    `kubeadm config images pull --config kubeadm.conf`

  5. docker tag 镜像
  镜像下载好后，还需要tag下载好的镜像，让下载好的镜像都是带有 k8s.gcr.io 标识的，如果不打tag变成k8s.gcr.io，那么后面用kubeadm安装会出现问题，因为kubeadm里面只认 google自身的模式。打tag后删除带有 registry.aliyuncs.com 标识的镜像。下面把操作写在脚本里。
      ```bash
      #/bin/bash

      # 打tag
      docker tag registry.aliyuncs.com/google_containers/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1
      docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.15.1 k8s.gcr.io/kube-proxy:v1.15.1
      docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.15.1 k8s.gcr.io/kube-apiserver:v1.15.1
      docker tag registry.aliyuncs.com/google_containers/kube-controller-manager:v1.15.1 k8s.gcr.io/kube-controller-manager:v1.15.1
      docker tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.15.1 k8s.gcr.io/kube-scheduler:v1.15.1
      docker tag registry.aliyuncs.com/google_containers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
      docker tag registry.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1

      # 删除带有 registry.aliyuncs.com 标识的镜像
      docker rmi registry.aliyuncs.com/google_containers/coredns:1.3.1
      docker rmi registry.aliyuncs.com/google_containers/kube-proxy:v1.15.1
      docker rmi registry.aliyuncs.com/google_containers/kube-apiserver:v1.15.1
      docker rmi registry.aliyuncs.com/google_containers/kube-controller-manager:v1.15.1
      docker rmi registry.aliyuncs.com/google_containers/kube-scheduler:v1.15.1
      docker rmi registry.aliyuncs.com/google_containers/etcd:3.3.10
      docker rmi registry.aliyuncs.com/google_containers/pause:3.1
      ```
  6. 查看下载的镜像列表
      ```bash
      [root@master ~]# docker image ls
      REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
      k8s.gcr.io/kube-controller-manager   v1.15.1             d75082f1d121        2 weeks ago         159MB
      k8s.gcr.io/kube-apiserver            v1.15.1             68c3eb07bfc3        2 weeks ago         207MB
      k8s.gcr.io/kube-proxy                v1.15.1             89a062da739d        2 weeks ago         82.4MB
      k8s.gcr.io/kube-scheduler            v1.15.1             b0b3c4c404da        2 weeks ago         81.1MB
      k8s.gcr.io/coredns                   1.3.1               eb516548c180        6 months ago        40.3MB
      k8s.gcr.io/etcd                      3.3.10              2c4adeb21b4f        8 months ago        258MB
      k8s.gcr.io/pause                     3.1                 da86e6ba6ca1        19 months ago       742kB
      ```

&nbsp;
&nbsp;

## 五、部署master节点
1. kubeadm init 初始化master节点
  kubeadm init --kubernetes-version=v1.15.1 --pod-network-cidr=172.22.0.0/16 --apiserver-advertise-address=192.168.198.200
  这里我们定义POD的网段为: 172.22.0.0/16 ，api server就是master本机IP地址。

2. 初始化成功后，最后会显示如下  
    ```
    Your Kubernetes control-plane has initialized successfully!

    To start using your cluster, you need to run the following as a regular user:

      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config

    You should now deploy a pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
      https://kubernetes.io/docs/concepts/cluster-administration/addons/

    Then you can join any number of worker nodes by running the following on each as root:

    kubeadm join 192.168.198.200:6443 --token 81i5bj.qwo2gfiqafmr6g6s \
        --discovery-token-ca-cert-hash sha256:aef745fb87e366993ad20c0586c2828eca9590c29738ef.... 
    ```
    `kubeadm join 192.168.198.200:6443 --token 81i5bj.qwo2gfiqafmr6g6s \
        --discovery-token-ca-cert-hash sha256:aef745fb87e366993ad20c0586c2828eca9590c29738ef.... `
    最后这个记录下，到时候添加node的时候要用到。
    
    同时/etc/kubernetes/会生成下面的文件
    ```bash
    [root@master ~]# ll /etc/kubernetes/
    总用量 36
    -rw------- 1 root root 5451 8月   5 15:12 admin.conf
    -rw------- 1 root root 5491 8月   5 15:12 controller-manager.conf
    -rw------- 1 root root 5459 8月   5 15:12 kubelet.conf
    drwxr-xr-x 2 root root  113 8月   5 15:12 manifests
    drwxr-xr-x 3 root root 4096 8月   5 15:12 pki
    -rw------- 1 root root 5435 8月   5 15:12 scheduler.conf
    ```
3. 验证测试
    配置kubectl命令
    `mkdir -p /root/.kube`
    `cp /etc/kubernetes/admin.conf /root/.kube/config`
    执行获取pods列表命令，查看相关状态
    ```bash
    [root@master ~]# kubectl get pods --all-namespaces
    NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
    kube-system   coredns-5c98db65d4-rrpgm                0/1     Pending   0          5d18h
    kube-system   coredns-5c98db65d4-xg5cc                0/1     Pending   0          5d18h
    kube-system   etcd-master                             1/1     Running   0          5d18h
    kube-system   kube-apiserver-master                   1/1     Running   0          5d18h
    kube-system   kube-controller-manager-master          1/1     Running   0          5d18h
    kube-system   kube-proxy-8vf84                        1/1     Running   0          5d18h
    kube-system   kube-scheduler-master                   1/1     Running   0          5d18h
    ```
    其中coredns pod处于Pending状态，这个先不管
    也可以执行 kubectl get cs 查看集群的健康状态
    ```
    [root@master ~]# kubectl get cs
    NAME                 STATUS    MESSAGE             ERROR
    scheduler            Healthy   ok                  
    controller-manager   Healthy   ok                  
    etcd-0               Healthy   {"health":"true"}   
    ```
    &nbsp;
    &nbsp;
## 六、部署calico网络 （在master上执行）    
  calico介绍：Calico是一个纯三层的方案，其好处是它整合了各种云原生平台(Docker、Mesos与OpenStack等)每个Kubernetes节点上通过Linux Kernel有的L3 forwarding功能来实现vRouter功能。
1. 下载calico 官方镜像
    这里要下载三个镜像，分别是calico-node:v3.1.4、calico-cni:v3.1.4、calico-typha:v3.1.4
    直接运行 docker pull 下载即可
    ```bash
    docker pull calico/node:v3.1.4
    docker pull calico/cni:v3.1.4
    docker pull calico/typha:v3.1.4
    ```

2. tag 这三个calico镜像
    ```bash
    docker tag calico/node:v3.1.4 quay.io/calico/node:v3.1.4
    docker tag calico/cni:v3.1.4 quay.io/calico/cni:v3.1.4
    docker tag calico/typha:v3.1.4 quay.io/calico/typha:v3.1.4
    ```

3. 删除原有镜像
    ```bash
    docker rmi calico/node:v3.1.4
    docker rmi calico/cni:v3.1.4
    docker rmi calico/typha:v3.1.4
    ```
  
4. 部署calico
    4.1 下载执行rbac-kdd.yaml文件
    `curl https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml -O`
    `kubectl apply -f rbac-kdd.yaml`

    4.2 下载、配置calico.yaml文件
    `curl https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/policy-only/1.7/calico.yaml -O`

    把ConfigMap 下的 typha_service_name 值由none变成 calico-typha
    ```bash
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: calico-config
      namespace: kube-system
    data:
      # To enable Typha, set this to "calico-typha" *and* set a non-zero value for Typha replicas
      # below.  We recommend using Typha if you have more than 50 nodes. Above 100 nodes it is
      # essential.
      #typha_service_name: "none"   #before
      typha_service_name: "calico-typha"   #after
    ```
    设置 Deployment 类目的 spec 下的replicas值为1
    ```bash
    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: calico-typha
      namespace: kube-system
      labels:
        k8s-app: calico-typha
    spec:
      # Number of Typha replicas.  To enable Typha, set this to a non-zero value *and* set the
      # typha_service_name variable in the calico-config ConfigMap above.
      #
      # We recommend using Typha if you have more than 50 nodes.  Above 100 nodes it is essential
      # (when using the Kubernetes datastore).  Use one replica for every 100-200 nodes.  In
      # production, we recommend running at least 3 replicas to reduce the impact of rolling upgrade.
      #replicas: 0  #before
      replicas: 1   #after
      revisionHistoryLimit: 2
      template:
        metadata:
    ```
    4.3 定义POD网段
    我们找到CALICO_IPV4POOL_CIDR，然后值修改成之前定义好的POD网段，我这里是172.22.0.0/16
    ```bash
    - name: CALICO_IPV4POOL_CIDR
      value: "172.22.0.0/16"
    ```
    4.4 开启bird模式
    把 CALICO_NETWORKING_BACKEND 值设置为 bird ，这个值是设置BGP网络后端模式
    ```bash
    - name: CALICO_NETWORKING_BACKEND
      #value: "none"
      value: "bird"
    ```
    4.5 部署calico.yaml文件
    上面参数设置调优完毕，我们执行下面命令彻底部署calico
    `kubectl apply -f calico.yaml`
&nbsp;
&nbsp;
## 七、部署node节点
1. 下载安装镜像（在node上执行）
  node上也是需要下载安装一些镜像的，需要下载的镜像为：kube-proxy:v1.13、pause:3.1、calico-node:v3.1.4、calico-cni:v3.1.4、calico-typha:v3.1.4
  1.1 下载镜像，打tag，删除原镜像
    ```bash
    docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.13.0
    docker pull registry.aliyuncs.com/google_containers/pause:3.1
    docker pull calico/node:v3.1.4
    docker pull calico/cni:v3.1.4
    docker pull calico/typha:v3.1.4

    docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.13.0 k8s.gcr.io/kube-proxy:v1.13.0
    docker tag registry.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
    docker tag calico/node:v3.1.4 quay.io/calico/node:v3.1.4
    docker tag calico/cni:v3.1.4 quay.io/calico/cni:v3.1.4
    docker tag calico/typha:v3.1.4 quay.io/calico/typha:v3.1.4

    docker rmi registry.aliyuncs.com/google_containers/kube-proxy:v1.13.0
    docker rmi registry.aliyuncs.com/google_containers/pause:3.1
    docker rmi calico/node:v3.1.4
    docker rmi calico/cni:v3.1.4
    docker rmi calico/typha:v3.1.4
    ```
    1.2 把node加入集群里
      在node上运行
        `kubeadm join 192.168.198.200:6443 --token 81i5bj.qwo2gfiqafmr6g6s \ --discovery-token-ca-cert-hash sha256:aef745fb87e366993ad20c0586c2828eca9590c29738ef....`
      运行完后，我们在master节点上运行 kubectl get nodes 命令查看node是否正常
      ```bash
      [root@master ~]# kubectl get nodes
      NAME     STATUS   ROLES    AGE     VERSION
      master   Ready    master   5d19h   v1.15.1
      node1    Ready    <none>   5d18h   v1.15.1
      node2    Ready    <none>   5d18h   v1.15.1
      ```

## 总结
  到此，k8s就部署完成了。
