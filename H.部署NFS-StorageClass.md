tags: helm, docker,master

# H. 部署NFS-StorageClass

注意：
1. 如果没有特殊指明，本文档的所有操作**均在 wls-k8s-01 节点上执行**，然后远程分发文件和执行命令；

## 安装NFS

``` bash
# 创建存储目录
$ mkdir -p /project/nfs_data/general
# 安装nfs服务端
$ apt-get install nfs-kernel-server
# 配置
$ cat /etc/exports
/project/nfs_data/general 10.0.0.0/24(rw,sync,no_subtree_check) 172.30.0.0/16(rw,sync,no_subtree_check)
```

## 下载nfs-subdir-external-provisioner，修改配置

``` bash
$ cd /opt/k8s/work/
$ wget https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/releases/download/nfs-subdir-external-provisioner-4.0.17/nfs-subdir-external-provisioner-4.0.17.tgz
$ tar -zxf nfs-subdir-external-provisioner-4.0.17.tgz
$ cd nfs-subdir-external-provisioner
$ cp values.yaml{,.tmpl} 
$ diff values.yaml values.yaml.tmpl
11,12c11,12
<   server: 10.0.0.5
<   path: /project/nfs_data/general
---
>   server:
>   path: /nfs-storage
23c23
<   provisionerName: nfs-storage
---
>   # provisionerName:
37c37
<   reclaimPolicy: Retain
---
>   reclaimPolicy: Delete
40c40
<   archiveOnDelete: false
---
>   archiveOnDelete: true
62c62
<   enabled: false
---
>   enabled: true
93c93
<   name: nfs-client-provisioner
---
>   name:
```

注：

server：nfs服务端地址

path：nfs服务共享路径

provisionerName：provisioner名称（与Deployment中变量 PROVISIONER_NAME 一致）

reclaimPolicy：回收策略，修改为 retain（pod，pvc被删除时，原数据不会被彻底删除）

serviceAccount:

​	name：授权容器访问k8s资源的用户名

### 安装

``` bash
$ helm upgrade --install nfs-subdir-external-provisioner . -n monitoring
...
$ kubectl get StorageClass
NAME              PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client        nfs-storage      Delete          Immediate           false                  94d
$ kubectl get deployment -n axe-sdk
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
nfs-client-provisioner   1/1     1            1           94d
```

### 测试

``` bash
$ cat test-claim.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
$ cat test-pod.yaml
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: harbor.awsl8.com/axe/busybox:1.26.2
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "sleep 300 && touch /mnt/ERROR && exit 0 || exit 1"
    volumeMounts:
      - name: nfs-pvc
        mountPath: /mnt
        subPath: JoJo
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc
      persistentVolumeClaim:
        claimName: test-claim
$ kubectl get pvc
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
test-claim   Bound    pvc-a470122e-1177-4c82-9c37-4212c52250c8   1Mi        RWX            nfs-client        94d

```

