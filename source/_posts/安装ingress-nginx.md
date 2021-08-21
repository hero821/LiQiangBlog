---
title: 安装ingress-nginx
date: 2021-08-15 11:00:00
tags:
- K8S
- ingress-nginx
categories:
- [K8S]
---

# yaml安装

```shell
# 下载
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.46.0/deploy/static/provider/cloud/deploy.yaml \
-O ingress-nginx.yaml
# 修改配置
vi ingress-nginx.yaml
# 提交
kubectl apply -f ingress-nginx.yaml
```

<!-- more -->

# helm安装

```shell
# 添加仓库
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
# 搜索版本
helm search repo ingress-nginx
# 下载指定版本
helm pull ingress-nginx/ingress-nginx --version 3.32.0
tar zxvf ingress-nginx-3.32.0.tgz 
cd ingress-nginx
cp values.yaml values-override.yaml
# 修改配置
vi values-override.yaml
###
controller:
  hostNetwork: true
  kind: Deployment
  nodeSelector:
    kubernetes.io/hostname: k8s-01
###
# 安装部署
helm install --create-namespace --namespace cmp-ingress-nginx ingress-nginx -f values-override.yaml .
# 检测版本
kubectl -n cmp-ingress-nginx exec -it \
$(kubectl -n cmp-ingress-nginx get pods -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}') \
-- /nginx-ingress-controller --version
```

