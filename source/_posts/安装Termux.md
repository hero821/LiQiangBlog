---
title: 安装Termux
date: 2021-07-25 22:00:00
tags:
- Termux
- 手机
categories:
- [Termux]
---

# 安装

方法1：打开 https://f-droid.org/packages/com.termux ，下载 APK 安装，推荐

方法2：打开 google play store 搜索 termux ，直接安装

<!-- more -->

# 配置

## 修改 apt 软件源

```shell
sed -i 's@^\(deb.*stable main\)$@#\1\ndeb https://mirrors.tuna.tsinghua.edu.cn/termux/termux-packages-24 stable main@' \
$PREFIX/etc/apt/sources.list
sed -i 's@^\(deb.*games stable\)$@#\1\ndeb https://mirrors.tuna.tsinghua.edu.cn/termux/game-packages-24 games stable@' \
$PREFIX/etc/apt/sources.list.d/game.list
sed -i 's@^\(deb.*science stable\)$@#\1\ndeb https://mirrors.tuna.tsinghua.edu.cn/termux/science-packages-24 science stable@' \
$PREFIX/etc/apt/sources.list.d/science.list
pkg update
```

## 访问手机存储

```shell
# 获取权限，会在 home 下生成 storage 目录
termux-setup-storage
# 查看 storage 目录结构
tree storage
storage
├── dcim -> /storage/emulated/0/DCIM
├── downloads -> /storage/emulated/0/Download
├── movies -> /storage/emulated/0/Movies
├── music -> /storage/emulated/0/Music
├── pictures -> /storage/emulated/0/Pictures
└── shared -> /storage/emulated/0

6 directories, 0 files
```

# 使用

## 安装 SSH 服务

**安装 OPENSSH**

```shell
pkg install openssh
# 启动sshd
sshd
# 停止sshd
pkill sshd
# 修改密码
passwd
# 获取用户
id
uid=10300(u0_a300) gid=10300(u0_a300) groups=10300(u0_a300),3003(inet),9997(everybody),20300(u0_a300_cache),50300(all_a300)
```

**配置远程登录**

方法1：通过密码登录

```shell
# 查看 termux 配置，默认 PasswordAuthentication yes
cat $PREFIX/etc/ssh/sshd_config
```

方法2：在客户端生成秘钥对

```shell
# 生成密钥对，复制 id_rsa.pub 内容到 termux：data\data\com.termux\files\home\.ssh\authorized_keys
ssh-keygen
```

方法3：在 termux 生成秘钥对

```shell
# 生成密钥对，复制 id_rsa 到客户端
ssh-keygen
cd .ssh
cat id_rsa.pub > authorized_keys
```

方法4：直接使用 openssh 安装的时候生成的密钥对

```shell
# 复制 ssh_host_rsa_key.pub 到 authorized_keys
cat ../usr/etc/ssh/ssh_host_rsa_key.pub > .ssh/authorized_keys
# 复制 ssh_host_rsa_key 到 客户端
cat ../usr/etc/ssh/ssh_host_rsa_key
```

**客户端登录**

```shell
# 密码登录
ssh u0_a300@192.168.137.2 -p 8022
# 密钥登录
ssh -i ~/.ssh/id_rsa 192.168.137.2 -p 8022
```

## 查看监听端口

Andorid 10 版本以后的 Termux 无法正常使用 netstat -an 命令，可以通过 nmap 命令查看

```shell
# 安装nmap
pkg install nmap
# 扫描端口
nmap 127.0.0.1
```

# 参考

https://www.sqlsec.com/2018/05/termux.html

https://github.com/termux/termux-app

