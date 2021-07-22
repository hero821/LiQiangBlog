---
title: 配置Docker容器的访问策略
date: 2021-07-21 12:00:00
tags:
- Docker
- iptables
categories:
- [Docker]
---

# 需求

需要对容器暴露的主机端口进行白名单访问控制，仅允许某些IP访问主机端口。

```shell
docker run -d --name nginx -p 60080:80 nginx:1.20-alpine
```

# 方案

Docker通过操作iptables规则来提供网络隔离。
Docker安装了两个名为DOCKER-USER和DOCKER的自定义iptables链，并确保传入的数据包始终首先由这两个链检查。
Docker的所有iptables规则都添加到DOCKER链中，不要手动修改此链。
如果需要添加自定义规则，请使用DOCKER-USER链，DOCKER-USER链在Docker链之前应用，也就是说用户自定义规则优先于Docker规则。
DOCKER-USER链默认只有一条规则，意思是 允许所有流量通过

```shell
Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0  
```

<!-- more -->

不推荐直接在FORWARD链中添加规则，具体原因，分两种情况说明
1. 在FORWARD链底部添加规则
   由于Docker自动添加的ACCEPT规则在前面，所以自定义规则无法起作用。
2. 在FORWARD链顶部添加规则
   刚添加完规则的时候是起作用的。
   执行systemctl restart docker等操作之后，Docker会把自己的规则添加到FORWARD链顶部，导致自定义规则无法起作用。

# 案例

允许IP访问

```shell
iptables -I DOCKER-USER -i eth0 ! -s 192.168.1.1 -j DROP
```

允许网段访问

```shell
iptables -I DOCKER-USER -i eth0 ! -s 192.168.1.0/24 -j DROP
```

允许连续IP访问

```shell
iptables -I DOCKER-USER -m iprange -i eth0 ! --src-range 192.168.1.1-192.168.1.3 -j DROP
```

允许多IP访问

```shell
iptables -I DOCKER-USER -i eth0 -j DROP
iptables -I DOCKER-USER -i eth0 -p tcp -m tcp -s 192.168.1.0/24 --dport 60080 -j RETURN
iptables -I DOCKER-USER -i eth0 -p tcp -m tcp -s 192.168.2.0/24 --dport 60080 -j RETURN
iptables -nvL
Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     tcp  --  eth0   *       192.168.2.0/24       0.0.0.0/0            tcp dpt:60080
    0     0 RETURN     tcp  --  eth0   *       192.168.1.0/24       0.0.0.0/0            tcp dpt:60080
    0     0 DROP       all  --  eth0   *       0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0  
```
