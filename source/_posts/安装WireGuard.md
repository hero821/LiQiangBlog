---
title: 安装WireGuard
date: 2021-07-20 12:00:00
tags:
- Proxy
- WireGuard
- VPN
categories:
- [Proxy]
---

# 安装

## 安装windows端

安装wireguard-amd64-0.3.16.msi：可以正常安装，无法正常使用
安装TunSafe-1.4.exe：可以正常安装，可以正常使用，可以支持ListenPortTCP

## 安装centos端

> 需要升级内核

```shell
yum install epel-release elrepo-release
yum install yum-plugin-elrepo
yum install kmod-wireguard wireguard-tools
# 重启
reboot
# 加载内核模块
modprobe wireguard
# 检查WG模块加载是否正常
lsmod | grep wireguard
```

<!-- more -->

## 安装wireguard-go端

> 无需升级内核，多平台，性能差

```shell
mkdir -p ~/go/src/github.com
cd ~/go/src/github.com
git clone https://github.com/WireGuard/wireguard-go.git
cd wireguard-go
make
make install
yum install wireguard-tools
# export WG_QUICK_USERSPACE_IMPLEMENTATION=wireguard-go
wg-quick up wg0
```

# 配置

## 配置服务端

```shell
# wg genkey > privatekey
# wg pubkey < privatekey > publickey
wg genkey | tee privatekey | wg pubkey > publickey
vi /etc/wireguard/wg0.conf
###
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <Server Private Key>
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
###
systemctl enable wg-quick@wg0
# 启动服务端
wg-quick up wg0
# 停止服务端
wg-quick down wg0
# 查看节点列表
wg show
# 重载配置文件，不影响已有连接
wg syncconf wg0 <(wg-quick strip wg0)
```

## 配置客户端

```shell
wg genkey | tee privatekey | wg pubkey > publickey
vi /etc/wireguard/wg0.conf
###
[Interface]
Address = 10.0.0.2/24
ListenPort = 51820
PrivateKey = <Client Private Key>
###
```

## 连接 Client 和 Server

### 使用配置文件

服务端wg0.conf添加到客户端Peer

```shell
###
[Peer]
PublicKey = <Client Public key>
AllowedIPs = 10.0.0.2/32
PersistentKeepalive = 30
###
```

客户端wg0.conf添加到服务端Peer

```shell
###
[Peer]
PublicKey = <Server Public Key>
AllowedIPs = 10.0.0.0/24
Endpoint = <Server Public IP>:51820
PersistentKeepalive = 30
###
```

### 使用命令行

服务端执行命令

```shell
wg set wg0 peer <Client Public key> allowed-ips 10.0.0.2/32 persistent-keepalive 30
wg-quick save wg0
```

客户端执行命令

```shell
wg set wg0 peer <Server Public Key> allowed-ips 10.0.0.0/24 persistent-keepalive 30 endpoint <Server Public IP>:51820 
wg-quick save wg0
```

## 配置生成

https://www.wireguardconfig.com

# 参考

https://www.wireguard.com

https://fuckcloudnative.io/posts/wireguard-docs-practice
