---
title: 安装etcd
date: 2021-07-27 11:00:00
tags:
- etcd
categories:
- [etcd]
---

# 安装

## 二进制

## docker

```shell
# 配置 etcd 使用主机IP 地址
export NODE1=172.17.0.1
# 配置一个 Docker 卷来存储 etcd 数据
docker volume create --name etcd-data
export DATA_DIR=etcd-data
# 配置 REGISTRY
# export REGISTRY=quay.io/coreos/etcd
export REGISTRY=quay.mirrors.ustc.edu.cn/coreos/etcd
# 配置 TAG
# TAG=latest
export TAG=v3.4.16
# 启动
docker run \
-p 2379:2379 \
-p 2380:2380 \
--volume=${DATA_DIR}:/etcd-data \
--name etcd ${REGISTRY}:${TAG} \
/usr/local/bin/etcd \
--data-dir=/etcd-data --name node1 \
--initial-advertise-peer-urls http://${NODE1}:2380 --listen-peer-urls http://0.0.0.0:2380 \
--advertise-client-urls http://${NODE1}:2379 --listen-client-urls http://0.0.0.0:2379 \
--initial-cluster node1=http://${NODE1}:2380
```

## 源码

# 测试