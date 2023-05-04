tags: kubernetes, SubjectAccessReview

# J.用户权限查询

注意：
1. 如果没有特殊指明，本文档的所有操作**均在 wls-k8s-01 节点上执行**；

## 快捷指令'auth can-i'

`kubectl` 提供 `auth can-i` 子命令，用于快速查询 API 鉴权。检查当前用户：

``` bash
$ kubectl auth can-i create deployments --namespace monitoring
yes
```

检查其他用户可执行操作：

``` bash
$ kubectl auth can-i create deployments --namespace default --as system:serviceaccount:monitoring:prometheus-k8s
no
```



## SubjectAccessReview资源

检查用户或组是否可以执行某操作。[介绍](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/authorization-resources/subject-access-review-v1/)

```bash
$ kubectl -n monitoring create -f - -o yaml << EOF
apiVersion: authorization.k8s.io/v1
kind: SubjectAccessReview
spec:
  nonResourceAttributes:
    path: /metrics
    verb: get
  user: system:serviceaccount:monitoring:prometheus-k8s
EOF
# 返回
apiVersion: authorization.k8s.io/v1
kind: SubjectAccessReview
metadata:
  creationTimestamp: null
spec:
  nonResourceAttributes:
    path: /metrics
    verb: get
  user: system:serviceaccount:monitoring:prometheus-k8s
status:
  allowed: true
  reason: 'RBAC: allowed by ClusterRoleBinding "prometheus-k8s" of ClusterRole "prometheus-k8s"
    to ServiceAccount "prometheus-k8s/monitoring"'
```

- nonResourceAttributes：表示非资源访问请求的信息 （在创建角色时，用 nonResourceURLs 定义的资源）

  - path：请求的URL路径
  - verb：HTTP动作

- user：查询的用户名，格式：system:serviceaccount:<命名空间>:<serviceaccount资源定义的名称>

- status：由服务器填充返回，如果允许该操作，allowed为true，否则为false

  