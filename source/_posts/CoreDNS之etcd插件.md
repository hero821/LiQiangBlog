---
title: CoreDNS之etcd插件
date: 2021-07-24 12:00:00
tags:
- DNS
- CoreDNS
- etcd
categories:
- [DNS]
---

# 配置etcd

```shell
docker run -d --name etcd \
-p 2379:2379 \
-p 2380:2380 \
--env ALLOW_NONE_AUTHENTICATION=yes \
bitnami/etcd:3.4.16-debian-10-r28

docker run -it --rm \
--env ALLOW_NONE_AUTHENTICATION=yes \
bitnami/etcd:3.4.16-debian-10-r28 \
etcdctl --endpoints http://172.17.0.1:2379 put /key1 value1

docker run -it --rm \
--env ALLOW_NONE_AUTHENTICATION=yes \
bitnami/etcd:3.4.16-debian-10-r28 \
etcdctl --endpoints http://172.17.0.1:2379 get / --prefix
```

<!-- more -->

# 配置CoreDNS

```shell
vi Corefile
###
. {
  etcd {
    fallthrough
    path /skydns
    endpoint http://172.17.0.1:2379
  }
  forward . /etc/resolv.conf
  log
  errors
  loadbalance
}
###
docker run -d --name coredns \
-p 53:53/udp \
-v $PWD/Corefile:/Corefile \
coredns/coredns:1.8.4
```

# 测试

```shell
yum install -y bind-utils

# 查询到：example.org. 3600 IN A 93.184.216.34
dig example.org @172.17.0.1

# 通过etcdctl添加A记录
docker run -it --rm \
--env ALLOW_NONE_AUTHENTICATION=yes \
bitnami/etcd:3.4.16-debian-10-r28 \
etcdctl --endpoints http://172.17.0.1:2379 put /skydns/org/example '{"host":"10.0.0.1","ttl":60}'

# 查询到：example.org. 3600 IN A 10.0.0.1
dig example.org @172.17.0.1

# 通过curl添加A记录
curl -X POST http://172.17.0.1:2379/v3/kv/put \
-d "{\"key\": \"$(echo -n '/skydns/org/example' | base64)\", \"value\": \"$(echo -n '{"host":"10.0.0.2","ttl":60}' | base64)\"}"

# 查询到：example.org. 3600 IN A 10.0.0.2
dig example.org @172.17.0.1

# 添加多个A记录，实现负载均衡
docker run -it --rm \
--env ALLOW_NONE_AUTHENTICATION=yes \
bitnami/etcd:3.4.16-debian-10-r28 \
etcdctl --endpoints http://172.17.0.1:2379 put /skydns/org/example/x1 '{"host":"10.0.0.1","ttl":60}'
docker run -it --rm \
--env ALLOW_NONE_AUTHENTICATION=yes \
bitnami/etcd:3.4.16-debian-10-r28 \
etcdctl --endpoints http://172.17.0.1:2379 put /skydns/org/example/x2 '{"host":"10.0.0.1","ttl":60}'

# 多次查询，每次返回的IP地址顺序不一样
dig +short example.org @172.17.0.1
dig +short example.org @172.17.0.1

# 清理etcd
docker run -it --rm \
--env ALLOW_NONE_AUTHENTICATION=yes \
bitnami/etcd:3.4.16-debian-10-r28 \
etcdctl --endpoints http://172.17.0.1:2379 del / --prefix

# 查询到：example.org. 3600 IN A 93.184.216.34
dig example.org @172.17.0.1
```

# 参考

https://github.com/bitnami/bitnami-docker-etcd#how-to-use-this-image

https://github.com/coredns/coredns/tree/master/plugin/etcd
