---
title: Linux路由相关概念
date: 2021-07-25 10:30:00
tags:
- 网络
- 路由
categories:
- [网络]
---

# 路由规则 ip rule

**操作系统默认规则**

```shell
ip rule list
0:      from all lookup local 
32766:  from all lookup main 
32767:  from all lookup default 
```

**每列的含义**

- Priority: 路由规则优先级，根据数字从小到大依次进行匹配，从0到32767
- Selector: 对IP数据包进行匹配的条件，如 from all 表示所有IP数据包
- Action: 对IP数据包执行的动作，如 lookup local 表示需要查找local路由表进行处理

<!-- more -->

**命令格式**

```shell
ip rule help
Usage: ip rule { add | del } SELECTOR ACTION
       ip rule { flush | save | restore }
       ip rule [ list [ SELECTOR ]]
SELECTOR := [ not ] [ from PREFIX ] [ to PREFIX ] [ tos TOS ] [ fwmark FWMARK[/MASK] ]
            [ iif STRING ] [ oif STRING ] [ pref NUMBER ] [ l3mdev ]
            [ uidrange NUMBER-NUMBER ]
ACTION := [ table TABLE_ID ]
          [ nat ADDRESS ]
          [ realms [SRCREALM/]DSTREALM ]
          [ goto NUMBER ]
          SUPPRESSOR
SUPPRESSOR := [ suppress_prefixlength NUMBER ]
              [ suppress_ifgroup DEVGROUP ]
TABLE_ID := [ local | main | default | NUMBER ]
```

**SELECTOR**

- from PREFIX：选择要匹配的源前缀

- to PREFIX：选择要匹配的目的前缀
- tos TOS：选择要匹配的TOS值
- fwmark MARK：选择要匹配的标记值
- iif NAME：选择要匹配的传入接口设备，例如：可以为转发的数据包和本地数据包创建单独的路由表
- oif NAME：选择要匹配的传出接口设备，仅适用于来自绑定到设备的本地套接字的数据包

**ACTION**

- table TABLEID：要查找的路由表标识符
- nat ADDRESS：要转换的IP地址（用于源地址）

# 路由表 ip route

**操作系统中有256个路由表**

- 编号255：是 local 路由表，处理本地IP和广播地址路由，local路由表只由kernel维护，不能更改和删除
- 编号254：是 main 路由表，处理所有非策略常规路由，不指定路由表名时默认使用的路由表，ip route 命令默认操作的就是这个路由表
- 编号253：是 default 路由表，所有其他路由表都没有匹配到的情况下，根据该表中的条目进行处理

- 编号1-252：是 用户可以自定义的 路由表
- 编号0：是 系统保留 路由表

**操作系统内部使用数字编号来标识路由表，为了方便记忆，有时需要使用符号标识路由表，对应关系在 /etc/iproute2/rt_tables 中**

```shell
cat /etc/iproute2/rt_tables 
255   local
254   main
253   default
0     unspec
```

**查询路由表**


```shell
ip route list
default via 192.168.128.2 dev ens33 proto static metric 100 
192.168.128.0/24 dev ens33 proto kernel scope link src 192.168.128.200 metric 100  
```

**每列的含义**

- PREFIX：目标地址前缀，可以是 default 代表所有地址 0.0.0.0/0，可以是 cidr
- proto RTPROTO：路由协议标识符，可以是数字编号或符号标识，对应关系在 /etc/iproute2/rt_protos 中
  - redirect：路由是由于ICMP重定向而安装的
  - kernel：路由是在自动配置期间由内核安装的
  - boot：路由是在启动过程中安装的
  - static：路由是管理员安装的，以覆盖动态路由
  - ra：路由是通过路由器发现协议安装的
- scope SCOPE：路由前缀所覆盖的目标地址范围，可以是数字编号或符号标识，对应关系在 /etc/iproute2/rt_scopes 中
- metric METRIC：路由跳数，指该条路由记录的质量，一般情况下，如果有多条到达相同目的地的路由记录，路由器会采用metric值小的那条路由
- via ADDRESS：下一跳路由器的地址
- dev NAME：输出设备名称
- src ADDRESS：发送数据包时首选的源地址

# 样例

双网卡：10.0.0.2/24 和 172.16.0.2/24
使来自 192.168.1.0/24 的数据包走 10.0.0.1 网关
是来自 192.168.2.0/24 的数据包走 172.16.0.1 网关

```shell
# 定义路由表名称
echo 100 t100 >> /etc/iproute2/rt_tables
echo 200 t200 >> /etc/iproute2/rt_tables
# 新增规则
ip rule add from 192.168.1.0/24 table t100
ip rule add from 192.168.2.0/24 table t200
# 添加路由
ip route add default via 10.0.0.1 table t100
ip route add default via 172.16.0.1 table t200
# 刷新路由表
ip route flush cache
```

