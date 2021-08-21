---
title: 安装dashboard
date: 2021-08-14 23:00:00
tags:
- K8S
- harbor
categories:
- [K8S]
---

# yaml安装

```shell
# 下载
# wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml -O dashboard.yaml
wget https://ghproxy.com/https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml \
-O dashboard.yaml
# 修改配置
## 修改 namespace
## 启用 NodePort
## 修改 ClusterRoleBinding 中 ClusterRole 为 cluster-admin
vi dashboard.yaml
# 提交
kubectl apply -f dashboard.yaml
```

<!-- more -->

# helm安装

```shell
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard
helm pull kubernetes-dashboard/kubernetes-dashboard --version 4.2.0
tar zxvf kubernetes-dashboard-4.2.0.tgz 
cd kubernetes-dashboard
cp values.yaml values-override.yaml
vi values-override.yaml
###
extraArgs:
  - --token-ttl=0
service:
  type: NodePort
  nodePort: 30000
metricsScraper:
  enabled: true
metrics-server:
  enabled: true
  args:
    - --kubelet-preferred-address-types=InternalIP
    - --kubelet-insecure-tls
###
helm install --create-namespace --namespace cmp-dashboard dashboard -f values-override.yaml .

# 获取 token
kubectl -n cmp-dashboard get secrets \
$(kubectl -n cmp-dashboard get secrets | grep kubernetes-dashboard-token | awk '{print $1}') \
-o jsonpath="{['data']['token']}" | base64 --decode && echo

# 提升权限
vi dashboard-rbac.yaml
###
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
	name: dashboard-kubernetes-dashboard
namespace: cmp-dashboard
###
kubectl apply -f dashboard-rbac.yaml

# 获取 token
kubectl -n cmp-dashboard get secrets \
$(kubectl -n cmp-dashboard get secrets | grep kubernetes-dashboard-token | awk '{print $1}') \
-o go-template="{{.data.token | base64decode}}" && echo
```

