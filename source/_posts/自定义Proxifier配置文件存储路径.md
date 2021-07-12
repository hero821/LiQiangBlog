---
title: 自定义Proxifier配置文件存储路径
date: 2020-10-01 12:00:00
tags: 
- 软件配置
- windows
categories:
- 软件配置
---
# 发现问题
今天使用Proxifier的时候，发现配置文件存储路径为：C:\Users\\***\AppData\Roaming\Proxifier\Profiles。
这样就会存在一些风险，例如：重装系统的时候（格式化系统盘），如果没想起来备份，可能就再也找不到了。
风险既然发现，就要想办法规避。
<!-- more -->
# 解决问题
## 过程
* 首先，查看Proxifier帮助文档（路径：D:\Program Files (x86)\Proxifier\Proxifier.chm），无果。
* 然后，Google各种查资料，无果。
* 再然后，试了试在C:\Users\\***\AppData\Roaming\Proxifier下面创建Profiles快捷方式，删除Profiles文件夹，结果启动Proxifier之后，竟然又自动生成了Profiles文件夹，失败。
* 最后，我想起来Linux有个ln命令，效果是我想要的，开始Google查询“windows 软连接”，找到了mklink命令。

查看help：
```shell
λ mklink
创建符号链接。

MKLINK [[/D] | [/H] | [/J]] Link Target

        /D      创建目录符号链接。默认为文件
                符号链接。
        /H      创建硬链接而非符号链接。
        /J      创建目录联接。
        Link    指定新的符号链接名称。
        Target  指定新链接引用的路径
                (相对或绝对)。
```
## 结论
注：需要管理员权限
```shell
λ mklink /D C:\Users\***\AppData\Roaming\Proxifier\Profiles E:\Config\Proxifier\Profiles
```
