---
title: Kubernetes学习笔记
description: 
date: 2023-01-01 00:00:00+0000
image: 
categories:
    - 2023
tags:
    - Linux
weight: 1
---

汇总k8s学习过程中的笔记

<!--more-->

## 基本概念

### 名词解释

**Cluster**

Cluster 是计算、存储和网络资源的集合，Kubernetes利用这些资源运行各种基于容器的应用。

**Master**

Master 是 Cluster 的大脑，它的主要职责是调度，即决定将应用放在哪里运行。Master 运行 Linux 操作系统，可以是物理机或者虚拟机。为了实现高可用，可以运行多个 Master。

**Node**

Node 的职责是运行容器应用。Node 由 Master 管理，Node 负责监控并汇报容器的状态，并根据 Master 的要求管理容器的生命周期。Node 运行在 Linux 操作系统，可以是物理机或者是虚拟机。

**Pod**

Pod 是 Kubernetes 的最小工作单元。每个 Pod 包含一个或多个容器。Pod 中的容器会作为一个整体被 Master 调度到一个 Node 上运行。

Kubernetes 引入 Pod 主要基于下面两个目的：

1. 可管理性

有些容器天生就是需要紧密联系，一起工作。Pod 提供了比容器更高层次的抽象，将它们封装到一个部署单元中。Kubernetes 以 Pod 为最小单位进行调度、扩展、共享资源、管理生命周期。

2. 通信和资源共享。

Pod 中的所有容器使用同一个网络 namespace，即相同的 IP 地址和 Port 空间。它们可以直接用 localhost 通信。同样的，这些容器可以共享存储，当 Kubernetes 挂载 volume 到 Pod，本质上是将 volume 挂载到 Pod 中的每一个容器。

Pods 有两种使用方式：

1. 运行单一容器。
   one-container-per-Pod 是 Kubernetes 最常见的模型，这种情况下，只是将单个容器简单封装成 Pod。即便是只有一个容器，Kubernetes 管理的也是 Pod 而不是直接管理容器。
2. 运行多个容器。
   但问题在于：哪些容器应该放到一个 Pod 中？
   答案是：这些容器联系必须 非常紧密，而且需要 直接共享资源。

**Controller**

Kubernetes 通常不会直接创建 Pod，而是通过 Controller 来管理 Pod 的。Controller 中定义了 Pod 的部署特性，比如有几个副本，在什么样的 Node 上运行等。为了满足不同的业务场景，Kubernetes 提供了多种 Controller，包括 Deployment、ReplicaSet、DaemonSet、StatefuleSet、Job 等。

1. Deployment

是最常用的 Controller，Deployment 可以管理 Pod 的多个副本，并确保 Pod 按照期望的状态运行。

2. ReplicaSet

实现了 Pod 的多副本管理。使用 Deployment 时会自动创建 ReplicaSet，也就是说 Deployment 是通过 ReplicaSet 来管理 Pod 的多个副本，我们通常不需要直接使用 ReplicaSet。

3. DaemonSet

用于每个 Node 最多只运行一个 Pod 副本的场景。正如其名称所揭示的，DaemonSet 通常用于运行 daemon。

4. StatefuleSet

能够保证 Pod 的每个副本在整个生命周期中名称是不变的。而其他 Controller 不提供这个功能，当某个 Pod 发生故障需要删除并重新启动时，Pod 的名称会发生变化。同时 StatefuleSet 会保证副本按照固定的顺序启动、更新或者删除。

5. Job

用于运行结束就删除的应用。而其他 Controller 中的 Pod 通常是长期持续运行。

**Service**

Kubernetes Service 定义了外界访问一组特定 Pod 的方式。Service 有自己的 IP 和端口，Service 为 Pod 提供了负载均衡。

Kubernetes 运行容器（Pod）与访问容器（Pod）这两项任务分别由 Controller 和 Service 执行。

**Namespace**

Namespace 可以将一个物理的 Cluster 逻辑上划分成多个虚拟 Cluster，每个 Cluster 就是一个 Namespace。不同 Namespace 里的资源是完全隔离的。

Kubernetes 默认创建了两个 Namespace。

- default -- 创建资源时如果不指定，将被放到这个 Namespace 中。
- kube-system -- Kubernetes 自己创建的系统资源将放到这个 Namespace 中。

### 网络模型

为了保证网络方案的标准化、扩展性和灵活性，Kubernetes 采用了 Container Networking Interface（CNI）规范。

CNI 是由 CoreOS 提出的容器网络规范，它使用了插件（Plugin）模型创建容器的网络栈。

目前已有多种支持 Kubernetes 的网络方案，比如 Flannel、Calico、Canal、Weave Net 等。因为它们都实现了 CNI 规范，用户无论选择哪种方案，得到的网络模型都一样，即每个 Pod 都有独立的 IP，可以直接通信。区别在于不同方案的底层实现不同。

## k8s架构

**架构图：**

![image](https://image-hosts-1305019894.cos.accelerate.myqcloud.com/2021/08/04/cf0185e7a78b7.jpg)

### master

> Master 是 Kubernetes Cluster 的大脑，运行着如下 Daemon 服务：kube-apiserver、kube-scheduler、kube-controller-manager、etcd 和 Pod 网络（例如 flannel）。

**API Server（kube-apiserver）**

API Server 提供 HTTP/HTTPS RESTful API，即 Kubernetes API。API Server 是 Kubernetes Cluster 的前端接口，各种客户端工具（CLI 或 UI）以及 Kubernetes 其他组件可以通过它管理 Cluster 的各种资源。

**Scheduler（kube-scheduler）**

Scheduler 负责决定将 Pod 放在哪个 Node 上运行。Scheduler 在调度时会充分考虑 Cluster 的拓扑结构，当前各个节点的负载，以及应用对高可用、性能、数据亲和性的需求。

**Controller Manager（kube-controller-manager）**

Controller Manager 负责管理 Cluster 各种资源，保证资源处于预期的状态。Controller Manager 由多种 controller 组成，包括 replication controller、endpoints controller、namespace controller、serviceaccounts controller 等。

不同的 controller 管理不同的资源。例如 replication controller 管理 Deployment、StatefulSet、DaemonSet 的生命周期，namespace controller 管理 Namespace 资源。

**etcd**

etcd 负责保存 Kubernetes Cluster 的配置信息和各种资源的状态信息。当数据发生变化时，etcd 会快速地通知 Kubernetes 相关组件。

**Pod 网络**

Pod 要能够相互通信，Kubernetes Cluster 必须部署 Pod 网络，flannel 是其中一个可选方案。

### node

> Node 是 Pod 运行的地方，Kubernetes 支持 Docker、rkt 等容器 Runtime。 Node上运行的 Kubernetes 组件有 kubelet、kube-proxy 和 Pod 网络（例如 flannel）。

**kubelet**

kubelet 是 Node 的 agent，当 Scheduler 确定在某个 Node 上运行 Pod 后，会将 Pod 的具体配置信息（image、volume 等）发送给该节点的 kubelet，kubelet 根据这些信息创建和运行容器，并向 Master 报告运行状态。

**kube-proxy**

service 在逻辑上代表了后端的多个 Pod，外界通过 service 访问 Pod。service 接收到的请求是如何转发到 Pod 的呢？这就是 kube-proxy 要完成的工作。

每个 Node 都会运行 kube-proxy 服务，它负责将访问 service 的 TCP/UPD 数据流转发到后端的容器。如果有多个副本，kube-proxy 会实现负载均衡。

**Pod 网络**

Pod 要能够相互通信，Kubernetes Cluster 必须部署 Pod 网络，flannel 是其中一个可选方案。

## 安装

### 关闭swap

临时关闭

```shell
sudo swapoff -a
```

永久关闭

```shell
sudo vim /etc/fstab
## 注释
#/swap.img      none    swap    sw      0       0
```

### 安装kubeadm kubeadm kubectl（国内源）

```shell
sudo apt-get update && sudo apt-get install -y ca-certificates curl 
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

echo "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update && apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### 下载k8s镜像（国内源）

使用脚本一键下载并更名

```shell
#!/usr/bin/env bash

docker image pull "registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.8.0"
docker image tag "registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.8.0"
docker image rmi "registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.8.0"

for IT in etcd:3.4.13-0 pause:3.4.1 kube-proxy:v1.21.3 kube-scheduler:v1.21.3 kube-controller-manager:v1.21.3 kube-apiserver:v1.21.3
do
    docker image pull "registry.cn-hangzhou.aliyuncs.com/google_containers/$IT"
    docker image tag  "registry.cn-hangzhou.aliyuncs.com/google_containers/$IT" "k8s.gcr.io/$IT"
    docker image rmi  "registry.cn-hangzhou.aliyuncs.com/google_containers/$IT"
done
```

## 部署

### 初始化Master

```shell
kubeadm init --apiserver-advertise-address [k8s-master ip] --pod-network-cidr=10.244.0.0/16
```

**过程：**

1. kubeadm 执行初始化前的检查。
2. 生成 token 和证书。
3. 生成 KubeConfig 文件，kubelet 需要这个文件与 Master 通信。
4. 安装 Master 组件，会从 goolge 的 Registry 下载组件的 Docker 镜像，这一步可能会花一些时间，主要取决于网络质量。
5. 安装附加组件 kube-proxy 和 kube-dns。
6. Kubernetes Master 初始化成功。
7. 提示如何配置 kubectl。
8. 提示如何安装 Pod 网络。
9. 提示如何注册其他节点到 Cluster。

### 配置kubectl

```shell
mkdir -p $HOME/.kubesudo 
cp -i /etc/kubernetes/admin.conf $HOME/.kube/configsudo 
chown $(id -u):$(id -g) $HOME/.kube/config

# 命令补全
apt-get -y install bash-completion
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

#### kubectl常用命令

**logs：查看POD或容器的日志**

```shell
#查看POD中所有容器的日志
kubectl logs nginxpod

#查看POD中指定容器的日志
kubectl logs nginxpod -c nginx

#查看指定名称空间中的POD中的指定容器日志
kubectl logs -n test mysqlpod -c mysql
```

**describe：显示资源运行状态信息**

```shell
kubectl describe pod <Pod Name>

#查看指定名称空间的POD
kubectl describe pods nginxpod
kubectl describe pods mysql -n test
kubectl describe pod kube-flannel-ds-v0p3x --namespace=kube-system
```

**exec：在POD中执行命令**

```shell
#此方式只适用于POD只有一个容器时
kubectl exec -it nginxpod -- /bin/bash

#如POD中有多个容器需要使用-c选项指定容器
kubectl exec -it nginxpod -c nginx -- /bin/bash

#一次性执行多个命令或使用管道符
kubectl exec nfs -- sh -c "echo 'SET testkey www.linux321.com' | redis-cli"
kubectl exec nfs -- sh -c "echo 'GET testkey' | redis-cli"
```

**查看资源类型**

```shell
kubectl api-resouurce
```

**查看名称空间**

```shell
kubectl get namespaces
```

**查看节点状态**

```shell
#在master上执行
 kubectl get nodes
```

**查看pod状态**

```shell
kubectl get pod --all-namespaces
kubectl get po -A
```

### 部署flannel

```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

国内镜像仓库手动下载flannel

```shell
docker pull registry.cn-hangzhou.aliyuncs.com/k8sos/flannel:v0.14.0
```

### 子节点加入Master

**查看现有token**

```shell
kubeadm token list
```

**生成一个新token**

```shell
kubeadm token create --print-join-command
# 默认有效期24小时,若想久一些可以结合--ttl参数,设为0则用不过期
kubeadm token create --ttl 0
```

**查看ca证书的hash**

```shell
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

**node加入Master**

```shell
kubeadm join [k8s-master ip]:6443 --token [token] --discovery-token-ca-cert-hash [ca cert]
```

**子节点需要的镜像**

- flannel
- pause
- k8s-proxy

**查看节点状态**

```shell
#在master上执行
kubectl get nodes
#都处于ready状态，说明node节点已可用
```

**查看pod状态**

```shell
kubectl get pod --all-namespaces
kubectl get po -A
```

**查看pod运行情况**

```shell
kubectl describe pod kube-flannel-ds-v0p3x --namespace=kube-system
#如果镜像已经都下载好还没处于ready状态，可尝试重启容器服务systemctl restart docker
```

### 删除节点

应首先清空节点并确保该节点为空， 然后取消配置该节点。

使用适当的凭证与控制平面节点通信，运行：

```shell
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
```

在删除节点之前，请重置 kubeadm 安装的状态：

```shell
kubeadm reset
```

重置过程不会重置或清除 iptables 规则或 IPVS 表。如果你希望重置 iptables，则必须手动进行：

```shell
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

如果要重置 IPVS 表，则必须运行以下命令：

```shell
ipvsadm -C
```

现在删除节点：

```shell
kubectl delete node <node name>
```

## 运行应用

### 命令运行

**创建应用**

master先下载nginx镜像

```shell
kubectl create deployment NAME --image=image -- [COMMAND] [args...] [options]
kubectl create deployment nginx --image="nginx:1.18.0"
#或者
kubectl run deployment nginx --image="nginx:1.21.1"
```

**删除应用**

```shell
kubectl delete deployment/nginx
```

**查看pod**

```shell
kubectl get pods
#查看详细信息
kubectl describe pod
```

**删除pod**

```shell
kubectl delete pods

#删除所有POD
kubectl delete pods --all

#删除指定多个POD
kubectl delete pods nginxpod demoapp
```

**查看容器调度**

```shell
kubectl get pods -o wide
```

**拓展实例**

```shell
kubectl scale deployment nginx --replicas=3
```

**查看ReplicaSet**

```shell
kubectl get replicaset
```

### 配置运行

**创建Deployment的yml文件**

常用字段：

```shell
apiVersion: apps/v1    # API群组及版本
kind: Deployment       # 资源类型特有标识
metadata:
  name <string>        # 资源名称，在作用域中要唯一
  namespace <string>   # 名称空间；Deployment隶属名称空间级别
spec:
  minReadySeconds <integer>  # Pod就绪后多少秒内任一容器无crash方可视为“就绪”
  replicas <integer>         # 期望的Pod副本数，默认为1
  selector <object>          # 标签选择器，必须匹配template字段中Pod模板中的标签
  template <object>          # Pod模板对象

  revisionHistoryLimit <integer> # 滚动更新历史记录数量，默认为10
  strategy <Object>              # 滚动更新策略
    type <string>                # 滚动更新类型，可用值有Recreate和RollingUpdate；
    rollingUpdate <Object>       # 滚动更新参数，专用于RollingUpdate类型
      maxSurge <string>          # 更新期间可比期望的Pod数量多出的数量或比例；
      maxUnavailable <string>    # 更新期间可比期望的Pod数量缺少的数量或比例，10， 
  progressDeadlineSeconds <integer>  # 滚动更新故障超时时长，默认为600秒
  paused <boolean>                   # 是否暂停部署过程
```

执行apply命令运行

```shell
kubectl apply -f deployment.yml
```
