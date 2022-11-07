tags: helm, docker,master

# G. 部署 helm

注意：
1. 如果没有特殊指明，本文档的所有操作**均在 wls-k8s-01 节点上执行**，然后远程分发文件和执行命令；

## 二进制版本安装

1.下载所需版本：

``` bash
$ cd /opt/k8s/work
$ wget https://get.helm.sh/helm-v3.8.2-linux-amd64.tar.gz
```

2.解压，移动到需要的目录：

``` bash
$ tar -zxf helm-v3.8.2-linux-amd64.tar.gz
$ mv linux-amd64/helm /opt/k8s/bin/
$ helm version
version.BuildInfo{Version:"v3.8.2", GitCommit:"6e3701edea09e5d55a8ca2aae03a68917630e91b", GitTreeState:"clean", GoVersion:"go1.17.5"}
```
