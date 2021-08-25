---
title: 安装openstack-T版-使用kolla
date: 2021-08-22 14:00:00
tags:
- OpenStack
- kolla
categories:
- [OpenStack]
---

# 环境

| 用途                                       | 主机名       | 操作系统    | CPU  | 内存 | sda  | sdb  | ens192        | ens224 |
| :----------------------------------------- | ------------ | ----------- | ---- | ---- | ---- | ---- | ------------- | ------ |
| control\|compute\|network\|storage\|deploy | openstack-01 | CentOS 1810 | 8C   | 16G  | 200G | 200G | 192.168.3.101 | -      |
| compute\|storage                           | openstack-02 | CentOS 1810 | 8C   | 16G  | 40G  | 200G | 192.168.3.102 | -      |
| compute\|storage                           | openstack-03 | CentOS 1810 | 8C   | 16G  | 40G  | 200G | 192.168.3.103 | -      |

<!-- more -->

# 安装

1. 【所有节点】启用2个网卡，ens192需要配置IP，ens224无需配置IP

   略

3. 【所有节点】关闭 selinux、firewalld、NetworkManager，开启 ip_forward

   略

3. 【所有节点】修改 yum源、pip源

   略

4. 【所有节点】修改 hostname、hosts

   ```shell
   # 在 openstack-01 主机上执行
   # 修改 hostname
   hostnamectl set-hostname openstack-01
   # 修改 hosts
   vi /etc/hosts
   ###
   192.168.3.101 openstack-01
   192.168.3.102 openstack-02
   192.168.3.103 openstack-03
   192.168.3.101 registry
   ###
   
   # 其他主机同理，略
   ```

5. 【所有节点】安装 python 虚拟环境

   ```shell
   # 安装
   yum install python-virtualenv
   virtualenv ~/virtualenv
   # 激活，如需退出，输入 deactivate
   source virtualenv/bin/activate
   # 更新 pip、setuptools、docker
   pip install -U pip
   pip install -U setuptools
   pip install -U docker
   ```
   
6. 【所有节点】安装 docker

   ```shell
   # 安装
   curl -o /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
   yum install docker-ce-18.06.2.ce
   systemctl enable docker && systemctl start docker
   # 配置
   mkdir /etc/systemd/system/docker.service.d
   tee /etc/systemd/system/docker.service.d/kolla.conf << 'EOF'
   [Service]
   MountFlags=shared
   EOF
   #
   mkdir -p /etc/docker
   tee /etc/docker/daemon.json <<-'EOF'
   {
     "insecure-registries": ["registry:4000"]
   }
   EOF
   # 重启
   systemctl daemon-reload
   systemctl restart docker
   ```

7. 【所有节点】给硬盘添加分区标签

   ```shell
   parted /dev/sdb -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_BS 1 -1
   parted /dev/sdb print
   ```

8. 【仅deploy节点】配置免密登录

   ```shell
   ssh-keygen
   ssh-copy-id root@openstack-01
   ssh-copy-id root@openstack-02
   ssh-copy-id root@openstack-03
   ```

9. 【仅deploy节点】安装 ansible、kolla、kolla-ansible

   ```shell
   # 安装软件
   yum install python-devel libffi-devel gcc openssl-devel libselinux-python
   # 安装 ansible
   pip install 'ansible<2.10'
   # 配置 ansible
   mkdir /etc/ansible
   vi /etc/ansible/ansible.cfg
   ###
   [defaults]
   host_key_checking=False
   pipelining=True
   forks=100
   ###
   
   # 部署 registry
   docker run -d --name registry --restart=always -p 4000:5000 -v /opt/registry:/var/lib/registry registry:latest
   
   # 下载源码
   yum install git
   # export {http,https,ftp}_proxy="http://192.168.3.254:1081"
   git clone https://github.com/openstack/kolla -b stable/train
   git clone https://github.com/openstack/kolla-ansible -b stable/train
   
   # 安装 kolla
   pip install ./kolla
   # 安装 kolla-ansible
   pip install ./kolla-ansible
   
   # 复制配置文件到 /etc/kolla 目录
   mkdir /etc/kolla
   cp -r kolla-ansible/etc/kolla/* /etc/kolla
   # 复制 inventory 文件到当前目录
   cp kolla-ansible/ansible/inventory/* .
   ```
   
10. 【仅deploy节点】准备初始配置

       ```shell
       # 配置 inventory
       vi multinode
       ###
       [control]
       openstack-01
       
       [network]
       openstack-01
       
       [compute]
       openstack-01
       openstack-02
       openstack-03
       
       [monitoring]
       openstack-01
       
       [storage]
       openstack-01
       openstack-02
       openstack-03
       
       [deployment]
       openstack-01       ansible_connection=local
       ###
       # 测试
       ansible -i multinode all -m ping
       
       # 配置 password
       kolla-genpwd
       # 修改 keystone_admin_password 密码
       vi /etc/kolla/passwords.yml
       ###
       keystone_admin_password: xxxxxx
       ###
       
       # 配置 globals.yml
       vi /etc/kolla/globals.yml
       ###
       kolla_base_distro: "centos"
       kolla_install_type: "source"
       openstack_release: "train"
       kolla_internal_vip_address: "192.168.3.100"
       docker_registry: "registry:4000"
       network_interface: "ens192"
       neutron_external_interface: "ens224"
       enable_ceph: "yes"
       enable_cinder: "yes"
       enable_neutron_qos: "yes"
       glance_backend_ceph: "yes"
       glance_backend_file: "no"
       nova_compute_virt_type: "qemu"
       ###
       ```

11. 【仅deploy节点】构建镜像

    ```shell
    # 生成 kolla-build.conf
    cd kolla
    pip install tox
    tox -e genconfig
    
    # 构建镜像
    kolla-build --base centos --type source --tag=train --registry registry:4000 --push
    # 构建指定模块镜像
    kolla-build keystone nova
    # 通过正则表达式筛选
    kolla-build ^nova-
    
    # 查看镜像
    curl http://192.168.3.101:4000/v2/_catalog
    # 查看镜像版本
    curl http://192.168.3.101:4000/v2/kolla/kolla/centos-source-base/tags/list
    ```

12. 【仅deploy节点】安装

    ```shell
    # 安装依赖
    kolla-ansible -i multinode bootstrap-servers
    # 部署前检查
    kolla-ansible -i multinode prechecks -e ansible_python_interpreter=/root/virtualenv/bin/python
    # 部署
    kolla-ansible -i multinode deploy -e ansible_python_interpreter=/root/virtualenv/bin/python
    # 创建环境变量文件
    kolla-ansible -i multinode post-deploy
    
    # 常用命令
    # 用于从系统中删除已部署的容器
    tools/cleanup-containers
    # 用于删除 neutron-agents 容器启动时在 Docker 主机上触发的网络更改的残余
    tools/cleanup-host
    # 用于从本地 Docker 缓存中删除 Kolla 构建的所有 Docker 镜像
    tools/cleanup-images --all
    ##  用于部署和启动所有 Kolla 容器
    kolla-ansible -i INVENTORY deploy
    ## 用于清理集群中的容器和卷
    kolla-ansible -i INVENTORY destroy
    ## 用于恢复完全停止的 mariadb 集群
    kolla-ansible -i INVENTORY mariadb_recovery
    ## 用于在部署每个 OpenStack 服务之前检查是否满足所有要求
    kolla-ansible -i INVENTORY prechecks
    ## 用于在部署节点上进行后期部署以获取管理 openrc 文件
    kolla-ansible -i INVENTORY post-deploy
    ## 用于拉取容器的所有镜像
    kolla-ansible -i INVENTORY pull
    ## 用于重新配置 OpenStack 服务
    kolla-ansible -i INVENTORY reconfigure 
    ## 用于升级现有的 OpenStack 环境
    kolla-ansible -i INVENTORY upgrade
    ## 用于进行部署后烟雾测试
    kolla-ansible -i INVENTORY check
    ## 用于停止运行容器
    kolla-ansible -i INVENTORY stop
    ## 用于检查并在必要时更新容器，而无需生成配置
    kolla-ansible -i INVENTORY deploy-containers
    ```

13. 【仅deploy节点】使用

    ```shell
    # 安装客户端
    pip install -U 'pip<21'
    pip install -U 'setuptools<45'
    pip install --ignore-installed python-openstackclient==4.0.0
    pip install --ignore-installed 'openstacksdk<0.40'
    # 生成 openrc 文件
    kolla-ansible post-deploy
    # 加载环境变量
    . /etc/kolla/admin-openrc.sh
    # 修改测试脚本
    cp kolla-ansible/tools/init-runonce .
    vi init-runonce
    ###
    EXT_NET_CIDR=${EXT_NET_CIDR:-'10.0.3.0/24'}
    EXT_NET_RANGE=${EXT_NET_RANGE:-'start=10.0.3.31,end=10.0.3.60'}
    EXT_NET_GATEWAY=${EXT_NET_GATEWAY:-'10.0.3.254'}
    ###
    # 执行测试脚本
    ./init-runonce
    ```

# 参考

kolla 与 openstack 版本对应关系

https://docs.openstack.org/releasenotes/kolla-ansible/index.html

用户指南

https://docs.openstack.org/kolla-ansible/train/user/index.html

