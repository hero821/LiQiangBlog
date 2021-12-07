---
title: 安装sshuttle
date: 2021-07-21 22:00:00
tags:
- Proxy
- sshuttle
categories:
- [Proxy]
---

# 说明

支持TCP、UDP，不支持ICMP

# 安装

```shell
export {http,https,ftp}_proxy="http://127.0.0.1:1081"
yum install python3 python3-devel gcc
```

## 主机环境安装

```shell
pip3 install sshuttle
```

## 虚拟环境安装

```shell
pip3 install virtualenv
virtualenv -p python3 /tmp/sshuttle
source /tmp/sshuttle/bin/activate
python -V && pip -V
pip install sshuttle
```

<!-- more -->

# 配置

## 代理TCP

```shell
sshuttle -r root@127.0.0.1:60022 0.0.0.0/0 -x 192.168.128.0/24 -v
```

## 代理TCP和DNS

```shell
sshuttle -r root@127.0.0.1:60022 0.0.0.0/0 -x 192.168.128.0/24 --dns -v
```

## 代理TCP和UDP

```shell
sshuttle -r root@127.0.0.1:60022 0.0.0.0/0 -x 192.168.128.0/24 --method tproxy -v
# 添加路由
ip route add local default dev lo table 100
ip rule add fwmark 1 lookup 100
ip -6 route add local default dev lo table 100
ip -6 rule add fwmark 1 lookup 100
```

## 常用配置

```shell
sshuttle -r root@127.0.0.1:60022 0.0.0.0/0 \
-x 0.0.0.0/8 \
-x 10.0.0.0/8 \
-x 100.64.0.0/10 \
-x 127.0.0.0/8 \
-x 169.254.0.0/16 \
-x 172.16.0.0/12 \
-x 192.168.0.0/16 \
-x 198.18.0.0/15 \
-x 224.0.0.0/4 \
-x 240.0.0.0/4 \
-v
```

