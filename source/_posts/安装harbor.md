---
title: 安装harbor
date: 2021-08-14 21:00:00
tags:
- K8S
- harbor
categories:
- [K8S]
---

# 添加仓库

```shell
helm repo add harbor https://helm.goharbor.io
```

# 下载指定版本，修改配置

```shell
# 查找版本
helm search repo harbor/harbor --versions
# 下载指定版本
helm fetch harbor/harbor --version 1.6.2
# 修改配置
tar zxvf harbor-1.6.2.tgz
cd harbor
cp values.yaml values-override.yaml
vi values-override.yaml
###
expose:
  type: nodePort
  tls:
    auto:
      commonName: "goharbor.io"
  nodePort:
    ports:
      http:
        nodePort: 30003
      https:
        nodePort: 30002
externalURL: https://192.168.3.201:30002
trivy:
  # 第一种方法：填写 GitHub Token，自动下载 Trivy DB
  gitHubToken: "xxx"
  # 第二种方法：跳过更新，手动下载 Trivy DB 到 /home/scanner/.cache/trivy/db/trivy.db
  # skipUpdate: true
notary:
  enabled: false
###
```

<!-- more -->

# 安装部署

```shell
# 安装
helm install --create-namespace --namespace cmp-harbor harbor -f values-override.yaml .
# 升级
helm -n cmp-harbor upgrade harbor -f values-override.yaml .
```

# 配置

```shell
# 配置docker，允许自签名证书
vi /etc/docker/daemon.json 
###
{
  "insecure-registries": ["https://192.168.3.201:30002"]
}
###
systemctl daemon-reload
systemctl restart docker
# 登录
docker login -u admin -p Harbor12345 192.168.3.201:30002
cat ~/.docker/config.json
# 推送镜像
docker push 192.168.3.201:30002/library/nginx:1.20-alpine
```
