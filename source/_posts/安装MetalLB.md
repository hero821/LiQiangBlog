---
title: 安装MetalLB
date: 2021-08-16 20:00:00
tags:
- K8S
- MetalLB
categories:
- [K8S]
---

# 概念

kubernetes 本身并没有实现 load balancer。
对于 cloud 用户，可以使用云服务商提供的 provider。
对于 bare metal 的用户来说，可以使用 MetalLB。

<!-- more -->

MetalLB有两个核心特性：地址分配和对外公告

## 地址分配：

MetalLB 会为用户的 load balancer 类型 service 分配 IP 地址，该 IP 地址不是凭空产生的，需要用户预先分配。

## 对外公告：

MetalLB 为服务分配了外部 IP 地址后，需要让集群外的网络知道该 IP 的存在。 MetalLB 使用标准路由协议来实现这一点：ARP、NDP 或 BGP。

### Layer 2 模式（ARP/NDP）

Layer 2 模式下，每个 service 会有集群中的一个 node 来负责。当服务客户端发起 ARP/NDP（用于 IPv4 的 ARP，用于 IPv6 的 NDP）解析的时候，对应的 node 会响应该 ARP/NDP 请求，该 service 的流量都会指向该 node，看上去该 node 上有多个地址。

Layer 2 模式并不是真正的负载均衡，因为流量都会先经过一个 node 后，再通过 kube-proxy 转给多个 end points。如果该 node 故障，MetalLB会迁移 IP 到另一个 node，并重新发送 ARP 告知客户端迁移。现代操作系统基本都能正确处理 ARP，因此 failover 不会产生太大问题。

Layer 2 模式更为通用，不需要用户有额外的设备，但由于 Layer 2 模式使用 ARP/NDP，地址池分配需要跟客户端在同一子网，地址分配略为繁琐。

### BGP 模式

BGP 模式下，集群中所有 node 都会跟上联路由器建立 BGP 连接，并且会告知路由器应该如何转发 service 的流量。

BGP 模式是真正的 load balancer。

# 预配置

## 启用严格的 ARP 模式

如果您在 IPVS 模式下使用 kube-proxy，从 Kubernetes v1.14.2 开始，您必须启用严格的 ARP 模式。

```shell
# 方法1
kubectl edit configmap -n kube-system kube-proxy
###
  # 修改为 true
  strictARP: true
###

# 方法2
# 对比配置差异
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system
# 提交配置
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```

## 所有节点配置 IP 地址

Layer 2 模式，需要给所有 node 节点配置对应网段的 IP 地址，否则 ARP 无法通信

# 安装

## YAML 安装

```shell
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml
wget https://raw.githubusercontent.com/metallb/metallb/main/manifests/example-layer2-config.yaml -O metallb-config.yaml
vi metallb-config.yaml
###
  - 192.168.1.240/28 # 或者 192.168.1.240-192.168.1.250
###
kubectl apply -f metallb-config.yaml
```

## HELM 安装

```shell
# 配置代理
export {http,https,ftp}_proxy="http://192.168.3.254:1081"
# 添加仓库
helm repo add metallb https://metallb.github.io/metallb
# 搜索版本
helm search repo metallb
# 下载指定版本
helm pull metallb/metallb --version 0.10.2
tar zxvf metallb-0.10.2.tgz 
cd metallb
cp values.yaml values-override.yaml
# 修改配置
vi values-override.yaml
###
configInline:
  address-pools:
    - name: default
      protocol: layer2
      addresses:
        - 192.168.1.240/28
###
# 安装部署
helm install --create-namespace --namespace cmp-metallb metallb -f values-override.yaml .
```

# 测试

```shell
vi nginx-lb.yaml
###
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.20-alpine
          ports:
            - name: http
              containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
###
kubectl apply -f nginx-lb.yaml
kubectl get svc

# 测试
curl http://10.0.3.1
```

