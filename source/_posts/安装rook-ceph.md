---
title: 安装rook-ceph
date: 2021-08-14 21:00:00
tags:
- K8S
- ceph
categories:
- [K8S]
---

# 添加仓库

```shell
helm repo add rook https://charts.rook.io/release
```

# 下载指定版本，修改配置

```shell
# rook-ceph
helm fetch rook/rook-ceph --version 1.7.0
tar zxvf rook-ceph-v1.7.0.tgz
cd rook-ceph
cp values.yaml values-override.yaml
vi values-override.yaml
# rook-ceph-cluster
helm fetch rook/rook-ceph-cluster --version 1.7.0
tar zxvf rook-ceph-cluster-v1.7.0.tgz
cd rook-ceph-cluster
cp values.yaml values-override.yaml
vi values-override.yaml
```

<!-- more -->

# 安装部署

```shell
# rook-ceph
cd rook-ceph
helm install --create-namespace --namespace rook-ceph rook-ceph -f values-override.yaml .
# rook-ceph-cluster
cd rook-ceph-cluster
helm install --create-namespace --namespace rook-ceph rook-ceph-cluster -f values-override.yaml .
```

# 维护

## 通过 toolbox 维护

```shell
kubectl -n rook-ceph exec -it \
$(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') \
-- bash
ceph status
ceph df
ceph osd status
ceph osd df
ceph osd tree
ceph osd lspools
```

## 通过 dashboard 维护

```shell
# 添加nodeport
kubectl -n rook-ceph get svc rook-ceph-mgr-dashboard -o yaml > rook-ceph-mgr-dashboard-nodeport.yaml
vi rook-ceph-mgr-dashboard-nodeport.yaml
kubectl apply -f rook-ceph-mgr-dashboard-nodeport.yaml
# 获取密码，账号：admin
kubectl -n rook-ceph get secret rook-ceph-dashboard-password \
-o jsonpath="{['data']['password']}" | base64 --decode && echo
```
