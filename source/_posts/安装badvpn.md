---
title: 安装badvpn
date: 2021-07-22 12:00:00
tags:
- Proxy
- badvpn
categories:
- [Proxy]
---

# 安装

```shell
yum install cmake openssl openssl-devel nspr nspr-devel nss nss-devel
ln -s /usr/include/nss3 /usr/include/nss
git clone https://github.com/ambrop72/badvpn.git
cd badvpn
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr
make install
```

<!-- more -->

# 配置

## 服务端

一般情况，服务端只需启动socks5服务即可；如果需要转发UDP，服务端还需要启动udpgw服务。

```shell
badvpn-udpgw --listen-addr 127.0.0.1:7300
```

## 客户端

```shell
ip tuntap del dev tun0 mode tun
ip tuntap add dev tun0 mode tun
ip addr add 10.0.0.1/24 dev tun0
ip link set tun0 up
badvpn-tun2socks --tundev tun0 --netif-ipaddr 10.0.0.2 --netif-netmask 255.255.255.0 \
--socks-server-addr 12.34.56.78:1080 --username <username> --password <password> \
--udpgw-remote-server-addr 127.0.0.1:7300
```

# 测试

## 测试TCP

```shell
ip route add 192.168.1.0/24 via 10.0.0.1
curl http://192.168.1.1
```

## 测试UDP

```shell
ip route add 8.8.8.8/32 via 10.0.0.1
nslookup baidu.com 8.8.8.8
```

# 参考

https://github.com/yangchuansheng/love-gfw/blob/master/docs/badvpn-linux.md