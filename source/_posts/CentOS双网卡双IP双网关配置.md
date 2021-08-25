---
title: CentOS双网卡双IP双网关配置
date: 2021-08-24 18:00:00
tags:
- 网络
- 路由
categories:
- [网络]
---

# 环境和现象

操作系统同时配置2个网卡

```shell
[root@localhost ~]# ip addr
1: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:b3:0b:5d brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.201/24 brd 192.168.3.255 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:feb3:b5d/64 scope link 
       valid_lft forever preferred_lft forever
2: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:b3:0c:3d brd ff:ff:ff:ff:ff:ff
    inet 10.0.3.201/24 brd 10.0.3.255 scope global ens224
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:feb3:c3d/64 scope link 
       valid_lft forever preferred_lft forever
```

路由表默认网关只能同时存在1个，2个网卡都设置网关，后者会覆盖前者

```shell
[root@localhost ~]# ip route
default via 192.168.3.254 dev ens192
192.168.3.0/24 dev ens192 proto kernel scope link src 192.168.3.201
10.0.3.0/24 dev ens224 proto kernel scope link src 10.0.3.201
```

192.168.3.201 可以ping通，10.0.3.201 无法ping通

```shell
[root@localhost ~]# ping 192.168.3.201 -c 1
PING 192.168.3.201 (192.168.3.201) 56(84) bytes of data.
64 bytes from 192.168.3.201: icmp_seq=1 ttl=128 time=5.27 ms

--- 192.168.3.201 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 5.272/5.272/5.272/0.000 ms

[root@localhost ~]# ping 10.0.3.1 -c 1
PING 10.0.3.1 (10.0.3.1) 56(84) bytes of data.
^C
--- 10.0.3.1 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms
```

<!-- more -->

# 分析原因

TODO

# 解决问题

```shell
echo 251 ens192 >> /etc/iproute2/rt_tables
echo 252 ens224 >> /etc/iproute2/rt_tables

ip route flush table ens192
ip route flush table ens224

ip route add default via 192.168.3.254 dev ens192 src 192.168.3.201 table ens192
ip route add default via 10.0.3.254 dev ens224 src 10.0.3.201 table ens224

ip rule add from 192.168.3.201/32 table ens192
ip rule add from 10.0.3.201/32 table ens224
```

