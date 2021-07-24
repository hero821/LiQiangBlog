---
title: 安装CoreDNS
date: 2021-07-24 10:00:00
tags:
- DNS
- CoreDNS
categories:
- [DNS]
---

# 安装

## 二进制

```shell
wget https://github.com/coredns/coredns/releases/download/v1.8.4/coredns_1.8.4_linux_amd64.tgz
mkdir coredns_1.8.4_linux_amd64
tar zxvf coredns_1.8.4_linux_amd64.tgz -C coredns_1.8.4_linux_amd64
```

## docker

```shell
docker pull coredns/coredns:1.8.4
```

## 源码

```shell
git clone https://github.com/coredns/coredns -b v1.8.4
cd coredns
make
```

<!-- more -->

# 配置

```shell
./coredns -plugins
vi Corefile
###
. {
  hosts {
    10.0.0.1 example.org
    fallthrough
  }
  forward . /etc/resolv.conf
  cache
  log
  errors
}
###
```

# 测试

```shell
./coredns -dns.port=1053
dig @localhost -p 1053 a example.org
```

