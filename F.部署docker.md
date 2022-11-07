tags: worker, docker

# F. 部署 docker 组件

<!-- TOC -->

- [F. 部署 docker 组件](#f-部署-docker-组件)
    - [设置Repository](#设置Repository)
    - [安装docker](#安装docker)

<!-- /TOC -->

注意：
1. 如果没有特殊指明，本文档的所有操作**均在 wls-k8s-01 节点上执行**，然后远程分发文件和执行命令；

## Repository方式安装

##### 设置Repository

1.安装依赖包：

``` bash
$ apt-get update
$ apt-get install ca-certificates curl gnupg lsb-release
```

2.添加GPG key：

``` bash
$ mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

3.添加Repository：

``` bash
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

##### 安装docker

1.更新包：

``` bash
$ apt-get update
```

2.安装docker，docker-compose：

``` bash
# 列出可用版本
$ apt-cache madison docker-ce | awk '{ print $3 }'
5:20.10.2~3-0~ubuntu-focal
5:20.10.1~3-0~ubuntu-focal
5:20.10.0~3-0~ubuntu-focal
5:19.03.15~3-0~ubuntu-focal
```

3.选择所需版本安装：

``` bash
$ VERSION_STRING=5:19.03.15~3-0~ubuntu-focal
$ apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-compose-plugin
```

4.自定义配置文件

``` bash
$ cat /etc/docker/daemon.json
{
  "data-root": "/project/docker/main",
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com"
  ],
  "insecure-registries": ["自建镜像仓库"]
}
```

注：

data-root：docker存储目录，包括镜像、容器卷、容器运行目录等

registry-mirrors：第三方镜像仓库

5.安装docker-compose

```bash
$ curl -SL https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
$ ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

