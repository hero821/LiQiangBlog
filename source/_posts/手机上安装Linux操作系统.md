---
title: 手机上安装Linux操作系统
date: 2021-07-25 23:00:00
tags:
- Termux
- 手机
categories:
- [Termux]
---

# 涉及软件

- termux
- anlinux 或者 andronix

<!-- more -->

# 安装

1. google play store 中安装 termux 和 anlinux
2. 打开 anlinux，选择 仪表盘 菜单
3. 点击 选择 按钮，选中 CentOS，点击 确定 按钮
4. 点击 复制 按钮，复制指令
5. 点击 启动 按钮，打开 termux，粘贴指令，点击输入法键盘 回车 按钮
6. 等待安装完成

安装完成之后，当前目录会生产4个文件

- centos-binds
- centos-fs
- centos.sh
- start-centos.sh

# 配置

关于 termux 的配置，可以参考 {% post_link 安装Termux 这里 %}

# 启动

```shell
./start-centos.sh
```

# 使用

## 安装 openresty

```shell
# 安装 wget
yum install wget
# 配置 yum 源
cd /etc/yum.repos.d
wget https://openresty.org/package/centos/openresty.repo
# 安装 openresty
yum install openresty
# 修改端口为8080（手机权限问题，默认端口80无法启动）
vi /usr/local/openresty/nginx/conf/nginx.conf
# 启动 openresty
openresty
# 配置 lua
###
server {
  listen       8082;
  server_name  localhost;
  location / {
    default_type text/html;
    content_by_lua_block {
      ngx.say("<p>hello, world</p>")
    }
  }
}
###
openresty -s reload
# 测试
curl http://127.0.0.1:8082
```

# 卸载

1. 打开 anlinux，选择 删除系统 菜单
2. 点击 选择 按钮，选中 CentOS，点击 确定 按钮
3. 点击 复制 按钮，复制指令
4. 点击 启动 按钮，打开 termux，粘贴指令，点击输入法键盘 回车 按钮
5. 等待卸载完成

# 参考

https://github.com/EXALAB/AnLinux-App