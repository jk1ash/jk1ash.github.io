---
title: Docker学习笔记
description: 
date: 2023-01-01 00:00:00+0000
image: 
categories:
    - 2023
tags:
    - Linux
weight: 1
---

学习docker过程中的笔记整理

<!--more-->

## 安装

### 本地安装

```shell
# install tools for install key
sudo apt-get -y install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg-agent \
     software-properties-common
 
# download official docker key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 
# verify you have the docker key fingerprint
apt-key fingerprint 0EBFCD88
 
# add docker repo
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
 
# update repo for docker
apt update
 
# install docker.io
apt -y install docker-ce docker-ce-cli containerd.io
```

### 使用官方脚本安装

```shell
curl -fsSL https://get.docker.com | bash -s docker
```

### 安装docker-compose

```shell
apt-get install docker-compose
```

## 使用阿里云镜像加速器

```shell
sudo mkdir -p /etc/docker
vim  /etc/docker/daemon.json
 
#输入
{
  "registry-mirrors": ["https://wzlet8dn.mirror.aliyuncs.com"]
}
或者
{
  "registry-mirrors": ["https://registry.aliyuncs.com/google_containers"]
}


sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 使用腾讯云镜像加速器

```shell
sudo mkdir -p /etc/docker
vim  /etc/docker/daemon.json
 
#输入
{
"registry-mirrors": [
"https://mirror.ccs.tencentyun.com"
]
}
 
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### docker解除sudo限制

```shell
sudo usermod -aG docker $USER
newgrp docker
sudo systemctl restart docker
```

## 常用命令

```shell
#查看本地镜像
docker images
 
#删除本地镜像
docker image rm [name/id]
docker rmi [name/id]
 
#查看镜像详情
docker inspect [name/id]
 
#使用commit创建image
docker commit [name/id] test:latest
 
#使用dockerfile构建image
docker build -t test:latest dockerfile .
 
#查看所有容器
docker ps -a
docker container ls
 
#删除容器
docker rm [name/id]
 
#镜像改名或者改标签
docker tag test:latest
#标记本地镜像，将其归入某一仓库。
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
 
#镜像搜索
docker search [image]
 
#上传镜像
docker push [image]
 
#下载镜像
docker pull [image]

#一键清理退出的容器
docker rm $(docker ps -qf status=exited)
docker container prune
```

## Dockerfile常用指令

```shell
FROM
指定 base 镜像。
 
MAINTAINER
设置镜像的作者，可以是任意字符串。
 
COPY
将文件从 build context 复制到镜像。
COPY 支持两种形式：
 
    COPY src dest
 
    COPY ["src", "dest"]
 
注意：src 只能指定 build context 中的文件或目录。
 
ADD
与 COPY 类似，从 build context 复制文件到镜像。不同的是，如果 src 是归档文件（tar, zip, tgz, xz 等），文件会被自动解压到 dest。
 
ENV
设置环境变量，环境变量可被后面的指令使用。例如：
 
    ENV MY_VERSION 1.3
 
    RUN apt-get install -y mypackage=$MY_VERSION
 
EXPOSE
指定容器中的进程会监听某个端口，Docker 可以将该端口暴露出来。我们会在容器网络部分详细讨论。
 
VOLUME
将文件或目录声明为 volume。我们会在容器存储部分详细讨论。
 
WORKDIR
为后面的 RUN, CMD, ENTRYPOINT, ADD 或 COPY 指令设置镜像中的当前工作目录。
 
RUN
在容器中运行指定的命令。
 
CMD
容器启动时运行指定的命令。
Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效。CMD 可以被 docker run 之后的参数替换。
 
ENTRYPOINT
设置容器启动时运行的命令。
Dockerfile 中可以有多个 ENTRYPOINT 指令，但只有最后一个生效。CMD 或 docker run 之后的参数会被当做参数传递给 ENTRYPOINT。
```

### RUN、CMD和ENTRYPOINT

**区别**

> RUN 执行命令并创建新的镜像层，RUN 经常用于安装软件包。
> 
> CMD 设置容器启动后默认执行的命令及其参数，但 CMD 能够被 docker run 后面跟的命令行参数替换。
> 
> ENTRYPOINT 配置容器启动时运行的命令。

**Shell 和 Exec 格式**

可用两种方式指定 RUN、CMD 和 ENTRYPOINT 要运行的命令：Shell 格式和 Exec 格式，二者在使用上有细微的区别。

Shell 格式

`<instruction> <command>`

例如：

```shell
RUN apt-get install python3 
 
CMD echo "Hello world" 
 
ENTRYPOINT echo "Hello world"
```

当指令执行时，shell 格式底层会调用 /bin/sh -c <command>。

例如下面的 Dockerfile 片段：

```shell
ENV name Cloud Man 
 
ENTRYPOINT echo "Hello, $name"
```

执行 docker run <image> 将输出：

```shell
Hello, Cloud Man
```

注意环境变量 name 已经被值 Cloud Man 替换。

下面来看 Exec 格式。

Exec 格式

`<instruction> ["executable", "param1", "param2", ...]`

例如：

```shell
RUN ["apt-get", "install", "python3"] 
 
CMD ["/bin/echo", "Hello world"] 
 
ENTRYPOINT ["/bin/echo", "Hello world"]
```

当指令执行时，会直接调用 `<command>`，不会被 shell 解析。
例如下面的 Dockerfile 片段：

```shell
ENV name Cloud Man 
 
ENTRYPOINT ["/bin/echo", "Hello, $name"]
```

运行容器将输出：

```shell
Hello, $name
```

注意环境变量“name”没有被替换。
如果希望使用环境变量，照如下修改

```shell
ENV name Cloud Man 
 
ENTRYPOINT ["/bin/sh", "-c", "echo Hello, $name"]
```

运行容器将输出：

```shell
Hello, Cloud Man
```

==**CMD** 和 **ENTRYPOINT** 推荐使用 **Exec** 格式，因为指令可读性更强，更容易理解。**RUN **则两种格式都可以。==

#### RUN

RUN 指令通常用于安装应用和软件包。

RUN 在当前镜像的顶部执行命令，并通过创建新的镜像层。Dockerfile 中常常包含多个 RUN 指令。

RUN 有两种格式：

```shell
Shell 格式：RUN
 
Exec 格式：RUN ["executable", "param1", "param2"]
```

下面是使用 RUN 安装多个包的例子：

```shell
RUN apt-get update && apt-get install -y \ 
 
bzr \
 
cvs \
 
git \
 
mercurial \
 
subversion
```

#### CMD

CMD 指令允许用户指定容器的默认执行的命令。

此命令会在容器启动且 docker run 没有指定其他命令时运行。

1. 如果 docker run 指定了其他命令，CMD 指定的默认命令将被忽略。
2. 如果 Dockerfile 中有多个 CMD 指令，只有最后一个 CMD 有效。
   CMD 有三种格式：
3. Exec 格式：CMD ["executable","param1","param2"]
   这是 CMD 的推荐格式。
4. CMD ["param1","param2"] 为 ENTRYPOINT 提供额外的参数，此时 ENTRYPOINT 必须使用 Exec 格式。
5. Shell 格式：CMD command param1 param2

#### ENTRYPOINT

ENTRYPOINT 指令可让容器以应用程序或者服务的形式运行。

ENTRYPOINT 看上去与 CMD 很像，它们都可以指定要执行的命令及其参数。不同的地方在于 ENTRYPOINT 不会被忽略，一定会被执行，即使运行 docker run 时指定了其他命令。

ENTRYPOINT 有两种格式：

1. Exec 格式：ENTRYPOINT ["executable", "param1", "param2"] 这是 ENTRYPOINT 的推荐格式。
2. Shell 格式：ENTRYPOINT command param1 param2
   在为 ENTRYPOINT 选择格式时必须小心，因为这两种格式的效果差别很大。

**Exec 格式**

ENTRYPOINT 的 Exec 格式用于设置要执行的命令及其参数，同时可通过 CMD 提供额外的参数。

ENTRYPOINT 中的参数始终会被使用，而 CMD 的额外参数可以在容器启动时动态替换掉。

比如下面的 Dockerfile 片段：

```shell
ENTRYPOINT ["/bin/echo", "Hello"] 
 
CMD ["world"]
```

当容器通过 docker run -it [image] 启动时，输出为：

```shell
Hello world
```

而如果通过 docker run -it [image] CloudMan 启动，则输出为：

```shell
Hello CloudMan
```

**Shell 格式**

ENTRYPOINT 的 Shell 格式会忽略任何 CMD 或 docker run 提供的参数。

#### 总结

1. 使用 RUN 指令安装应用和软件包，构建镜像。
2. 如果 Docker 镜像的用途是运行应用程序或服务，比如运行一个 MySQL，应该优先使用 Exec 格式的 ENTRYPOINT 指令。CMD 可为 ENTRYPOINT 提供额外的默认参数，同时可利用 docker run 命令行替换默认参数。
3. 如果想为容器设置默认的启动命令，可使用 CMD 指令。用户可在 docker run 命令行中替换此默认命令。

## 网络

### docker的网络类型

1. none
2. host
3. bridge
4. joined container

### 自定义网络

除了 none, host, bridge 这三个自动创建的网络，用户也可以根据业务需要创建 user-defined 网络。Docker 提供三种 user-defined 网络驱动：bridge, overlay 和 macvlan。overlay 和 macvlan 用于创建跨主机的网络。

```shell
docker network create --driver bridge my_net
```

### 容器间通信

1. IP 通信

同一IP段的容器可直接通信。不同IP段的容器需加入同一网关才可通信。两个容器要能通信，必须要有属于同一个网络的网卡。

2. Docker DNS Server

docker daemon 实现了一个内嵌的 DNS server，使容器可以直接通过“容器名”通信。使用 docker DNS 有个限制：只能在 user-defined 网络中使用。例子：

```shell
docker run -it --network=my_net --name=bbox1 busybox
 
docker run -it --network=my_net --name=bbox2 busybox
```

3. joined container

joined 容器非常特别，它可以使两个或多个容器共享一个网络栈，共享网卡和配置信息，joined 容器之间可以通过 127.0.0.1 直接通信。

```shell
docker run -it --network=container:test busybox
```

## 存储

### docker的存储类型

1. bind mount 挂载host的目录
2. docker managed volume 容器管理存储

### docker共享数据

1. bind mount挂载host同一个目录实现共享
2. volume container 是专门为其他容器提供 volume 的容器。它提供的卷可以是 bind mount，也可以是 docker managed volume。其他容器可以通过 --volumes-from 使用volume container来实现数据共享。

### data-packed volume container

将volume container制作成image，通过docker managed volume 共享数据

## 日志

docker logs
options:

```shell
-f, --follow         Follow log output
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)
  -n, --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)
```

例子：

```shell
# 类似tail -f
docker logs -f containID
```
