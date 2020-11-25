# 学习 Docker

### 参考

- [Docker--从入门到实践](https://yeasy.gitbook.io/docker_practice/image/list)
- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

### 安装

```sh
# 卸载旧版本 Docker
sudo apt-get remove docker \
               docker-engine \
               docker.io \
               containerd \
               runc

# 安装
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
# 添加Docker的官方GPG密钥，确认所下载软件包的合法性
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# 添加软件源
sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# 默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯
# 而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket
# 为了不使用 root 用户，将当前用户添加至 docker 用户组
sudo groupadd docker
sudo usermod -aG docker $USER

# 切换 su 用户再切回，将可以非 root 用户身份使用 Docker
docker --version
docker run hello-world
```

### 镜像使用

#### 获取镜像

```sh
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

例如:

```sh
docker pull ubuntu:18.04

# PHP-参考：https://hub.docker.com/_/php/
docker pull php:7.3-alpine
docker pull php:7.3-fpm-alpine

docker pull nginx
docker pull mysql:8.0
```

#### 运行容器

```sh
docker run [选项] 镜像名称 [命令] [参数列表]
```

例如:

```sh
docker run -it --rm ubuntu:18.04 bash
```

#### 查看镜像

```sh
# 列出镜像
docker image ls

# 清除虚悬镜像-被新版本覆盖的无效的镜像
docker image prune

# 中间层镜像-被顶层镜像依赖的镜像,没有名字,不需要删除
```

#### 删除镜像

```sh
docker image rm [选项] <镜像1> [<镜像2> ...]
```

例如:

```sh
docker image rm php

# 删除含有 redis 关键字的镜像
docker image rm $(docker image ls -q redis)
# 删除所有未被容器使用的镜像
docker image rm $(docker image ls)
```
