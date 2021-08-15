---
title: 使用kubespray安装K8S
date: 2021-08-14 20:00:00
tags:
- K8S
- kubespray
categories:
- [K8S]
---

# 测试环境

k8s-01：4C+2G+10G
k8s-02：4C+2G+10G
k8s-03：4C+2G+10G

# 预配置

关闭selinux

关闭firewalld

开启ip_forward

修改yum源

修改pip源

安装软件

```shell
yum install git python-pip -y
```

免密登录：

```shell
ssh-keygen
# ssh-copy-id root@ip
ssh-copy-id root@192.168.128.201
ssh-copy-id root@192.168.128.202
ssh-copy-id root@192.168.128.203
```

<!-- more -->

# 安装

```shell
# 下载源码
# git clone https://github.com/kubernetes-incubator/kubespray.git -b v2.14.2
git clone https://ghproxy.com/https://github.com/kubernetes-incubator/kubespray.git -b v2.14.2
cd kubespray

# 添加依赖
vi requirements.txt
###
ruamel.yaml.clib==0.2.2
MarkupSafe==1.1.1
###

# 安装依赖
pip install -r requirements.txt

# 拷贝inventory
cp -rfp inventory/sample inventory/mycluster

# 配置代理（需要socks服务端开启允许局域网访问）
vi inventory/mycluster/group_vars/all/all.yml
###
http_proxy: "http://192.168.128.1:1081"
https_proxy: "http://192.168.128.1:1081"
download_validate_certs: False
###

# 添加需要的组件
vi inventory/mycluster/group_vars/k8s-cluster/addons.yml

# 手动配置inventory部署
vi inventory/mycluster/inventory.ini 
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml

# 自动配置inventory部署
yum -y install python3 python3-pip
pip3 install -r contrib/inventory_builder/requirements.txt
declare -a IPS=(192.168.128.201 192.168.128.202 192.168.128.203)
CONFIG_FILE=inventory/mycluster/hosts.yml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
# 修改主机名：node1 改为 k8s-01，node2 改为 k8s-02，node3 改为 k8s-03
vi inventory/mycluster/hosts.yml
ansible-playbook -i inventory/mycluster/hosts.yml cluster.yml

# 部署失败，重试
ansible-playbook -i inventory/mycluster/hosts.yml cluster.yml --limit @cluster.retry
# 部署失败，重置
ansible-playbook -i inventory/mycluster/hosts.yml reset.yml

# 添加节点（仅worker）
# 修改inventory
ansible-playbook -i inventory/mycluster/hosts.yml scale.yml
# 删除节点（仅worker）
# 修改inventory
ansible-playbook -i inventory/mycluster/hosts.yml remove-node.yml
# 添加和删除节点（包含master、worker、etcd）
# 修改inventory
ansible-playbook -i inventory/mycluster/hosts.yml cluster.yml
# 更新集群（全量更新）
ansible-playbook -i inventory/mycluster/hosts.yml cluster.yml -e kube_version=v1.24.4
# 更新集群（滚动更新，会先驱逐POD）
ansible-playbook -i inventory/mycluster/hosts.yml upgrade-cluster.yml -e kube_version=v1.24.4
```

