---
title: 安装redsocks
date: 2021-07-21 23:00:00
tags:
- Proxy
- redsocks
- iptables
categories:
- [Proxy]
---

# 安装

```shell
yum install libevent-devel git gcc
git clone https://github.com/darkk/redsocks
cd redsocks
make
cp redsocks /usr/bin
```

<!-- more -->

# 启动

## 手动启动

```shell
redsocks -v
redsocks -c /etc/redsocks/redsocks.conf
```

## 自动启动

```shell
vi /lib/systemd/system/redsocks.service
###
[Unit]
Description=redsocks
After=network-online.target
[Service]
Type=simple
Restart=always
ExecStart=/usr/bin/redsocks -c /etc/redsocks/redsocks.conf
[Install]
WantedBy=multi-user.target
###
systemctl enable redsocks && systemctl start redsocks && systemctl status redsocks
```

# 配置

## 配置redsocks

```shell
mkdir /etc/redsocks
cp redsocks.conf.example /etc/redsocks/redsocks.conf
cd /etc/redsocks
vi redsocks.conf
###
base {
# ...
daemon = on;
# ...
}
redsocks {
# ...
ip = 127.0.0.1;
# ...
}
###
```

## 配置iptables

仅192.168.0.0/24走代理

```shell
# 新建链
iptables -t nat -N REDSOCKS
# 应用REDSOCKS
iptables -t nat -A OUTPUT -p tcp -j REDSOCKS
iptables -t nat -A PREROUTING -p tcp -j REDSOCKS
# 仅192.168.0.0/24走代理
iptables -t nat -A REDSOCKS -p tcp -d 192.168.0.0/24 -j REDIRECT --to-ports 12345
iptables -t nat -A REDSOCKS -p tcp -j RETURN
```

仅192.168.0.0/24不走代理

```shell
# 新建链
iptables -t nat -N REDSOCKS
# 应用REDSOCKS
iptables -t nat -A OUTPUT -p tcp -j REDSOCKS
iptables -t nat -A PREROUTING -p tcp -j REDSOCKS
# 仅192.168.0.0/24不走代理
iptables -t nat -A REDSOCKS -d 192.168.0.0/24 -j RETURN
iptables -t nat -A REDSOCKS -p tcp -j REDIRECT --to-ports 12345
```

指定用户建立的连接走代理

```shell
iptables -t nat -A OUTPUT -p tcp -m owner --uid-owner luser -j REDSOCKS
firefox
```

指定组建立的连接走代理

```shell
groupadd socksified
usermod --append --groups socksified luser
iptables -t nat -A OUTPUT -p tcp -m owner --gid-owner socksified -j REDSOCKS
# 以特定的组执行命令，结合上面命令使用
id
uid=1000(luser) gid=1000(luser) groups=1000(luser),1001(socksified)
sg socksified -c id
uid=1000(luser) gid=1001(socksified) groups=1000(luser),1001(socksified)
sg socksified -c firefox
```

常用配置

```shell
iptables -t nat -N REDSOCKS
iptables -t nat -A OUTPUT -p tcp -j REDSOCKS
iptables -t nat -A PREROUTING -p tcp -j REDSOCKS
iptables -t nat -A REDSOCKS -d 0.0.0.0/8 -j RETURN
iptables -t nat -A REDSOCKS -d 10.0.0.0/8 -j RETURN
iptables -t nat -A REDSOCKS -d 100.64.0.0/10 -j RETURN
iptables -t nat -A REDSOCKS -d 127.0.0.0/8 -j RETURN
iptables -t nat -A REDSOCKS -d 169.254.0.0/16 -j RETURN
iptables -t nat -A REDSOCKS -d 172.16.0.0/12 -j RETURN
iptables -t nat -A REDSOCKS -d 192.168.0.0/16 -j RETURN
iptables -t nat -A REDSOCKS -d 198.18.0.0/15 -j RETURN
iptables -t nat -A REDSOCKS -d 224.0.0.0/4 -j RETURN
iptables -t nat -A REDSOCKS -d 240.0.0.0/4 -j RETURN
iptables -t nat -A REDSOCKS -p tcp -j REDIRECT --to-ports 12345
```

