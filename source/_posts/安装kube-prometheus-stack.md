---
title: 安装kube-prometheus-stack
date: 2021-08-15 14:00:00
tags:
- K8S
- kube-prometheus-stack
- prometheus-operator
categories:
- [K8S]
---

# 介绍

prometheus-operator 已弃用，更名为 kube-prometheus-stack，以更清楚地反映它安装了 kube-prometheus 项目堆栈，其中 Prometheus Operator 只是一个组件。
原项目地址：https://github.com/helm/charts/tree/master/stable/prometheus-operator
新项目地址：https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

<!-- more -->

# 安装部署

```shell
# 添加仓库
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
# 搜索版本
helm search repo prometheus-community/kube-prometheus-stack --versions
# 下载指定版本
helm pull prometheus-community/kube-prometheus-stack --version 16.6.0
tar zxvf kube-prometheus-stack-16.6.0.tgz 
cd kube-prometheus-stack
cp values.yaml values-override.yaml
# 修改配置
vi values-override.yaml
###
prometheus:
  service:
    nodePort: 30004
    type: NodePort
  prometheusSpec:
    secrets:
      - etcd-certs
grafana:
  service:
    nodePort: 30006
    type: NodePort
alertmanager:
  service:
    nodePort: 30008
    type: NodePort
prometheusOperator:
  tls:
    enabled: false
  service:
    nodePort: 30010
    nodePortTls: 30012
    type: NodePort
kubeEtcd:
  endpoints:
    - 192.168.3.201
  serviceMonitor:
    scheme: https
    caFile: /etc/prometheus/secrets/etcd-certs/ca.pem
    certFile: /etc/prometheus/secrets/etcd-certs/admin-k8s-01.pem
    keyFile: /etc/prometheus/secrets/etcd-certs/admin-k8s-01-key.pem
###
# 安装
helm install --create-namespace --namespace cmp-prometheus kube-prometheus-stack -f values-override.yaml .
# 升级
helm -n cmp-prometheus upgrade kube-prometheus-stack -f values-override.yaml .
# 检测 prometheus 状态
http://192.168.3.201:30004/targets
# 登录 grafana，账号密码：admin/prom-operator
http://192.168.3.201:30006
```

# 解决指标无数据问题

## kube-controller-manager

```shell
vi /etc/kubernetes/manifests/kube-controller-manager.yaml
# 修改：- --bind-address=0.0.0.0
# 注释：- --port=0
```

## kube-scheduler

```shell
vi /etc/kubernetes/manifests/kube-scheduler.yaml
# 修改：- --bind-address=0.0.0.0
# 注释：- --port=0
```

## kube-proxy

```shell
kubectl -n kube-system edit configmaps kube-proxy
# 修改：metricsBindAddress: 0.0.0.0:10249
kubectl -n kube-system delete pod -l k8s-app=kube-proxy
```

## kube-etcd

```shell
kubectl -n cmp-prometheus create secret generic etcd-certs \
--from-file=/etc/ssl/etcd/ssl/ca.pem \
--from-file=/etc/ssl/etcd/ssl/admin-k8s-01.pem \
--from-file=/etc/ssl/etcd/ssl/admin-k8s-01-key.pem
```

