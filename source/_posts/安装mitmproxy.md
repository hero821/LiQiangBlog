---
title: 安装mitmproxy
date: 2021-07-24 18:00:00
tags:
- Proxy
- mitmproxy
- Python
categories:
- [mitmproxy]
---

# 说明

## 三个工具的用途

mitmproxy：为您提供交互式命令行界面
mitmweb：为您提供基于浏览器的 GUI
mitmdump：为您提供非交互式终端输出

<!-- more -->

# 安装

## Windows 安装

**使用zip文件**

下载：https://snapshots.mitmproxy.org/7.0.0/mitmproxy-7.0.0-windows.zip
解压：mitmproxy-7.0.0-windows.zip

**使用installer文件**

下载：https://snapshots.mitmproxy.org/7.0.0/mitmproxy-7.0.0-windows-installer.exe
安装：mitmproxy-7.0.0-windows-installer.exe

## Linux 安装

```shell
wget https://snapshots.mitmproxy.org/7.0.0/mitmproxy-7.0.0-linux.tar.gz
mkdir mitmproxy-7.0.0-linux
tar -zxvf mitmproxy-7.0.0-linux.tar.gz -C mitmproxy-7.0.0-linux
cd mitmproxy-7.0.0-linux
./mitmproxy --version
```

# 配置证书

## 下载证书

方法一：可以通过浏览器打开 http://mitm.it 下载证书，要求浏览器已经配置proxy
方法二：首次启动 mitmproxy 之后，证书会自动生成到 ~/.mitmproxy 目录，可以到 ~/.mitmproxy 目录中下载

## 安装证书到 Windows

选择【受信任的根证书颁发机构】

## 安装证书到 Linux

```shell
cp mitmproxy-ca-cert.pem /etc/pki/ca-trust/source/anchors
update-ca-trust
curl --proxy 192.168.128.1:8080 https://www.baidu.com
```

## 不安装证书，需要手动指定证书

```shell
curl --proxy 192.168.128.1:8080 --cacert mitmproxy-ca-cert.pem https://www.baidu.com
```

# 测试

## 使用 mitmproxy

参考：https://docs.mitmproxy.org/stable/mitmproxytutorial-userinterface

```shell
# 第一个窗口，启动mitmproxy
./mitmproxy
# 第一个窗口，指定监听端口，启动mitmproxy
./mitmproxy -p 8888

# 第二个窗口，请求url之后，第一个窗口可以看到数据
curl --proxy http://127.0.0.1:8080 http://wttr.in/Dunedin?0
curl --proxy http://127.0.0.1:8080 http://wttr.in/Innsbruck?0

# 第一个窗口，支持快捷键和命令行操作
## 使用箭头键 ↑ 和 ↓ 更改焦点 (>>)
## 按 ENTER 查看详细信息
## 按 q 退出当前视图
## 按 ？获取所有可用键盘快捷键的列表
## 按 : 打开底部的命令提示符
## 输入 :console.view.flow @focus 等同于按 ENTER 键
```

## 使用 mitmweb

```shell
# 启动之后，自动打开浏览器 http://127.0.0.1:8081 
./mitmweb
```

## 使用 mitmdump

```shell
./mitmdump -s script.py
```

## 使用 addons：AddHeader

```shell
vi AddHeader.py
###
class AddHeader:
    def response(self, flow):
        flow.response.headers["name"] = str('mitmproxy')


addons = [
    AddHeader()
]
###
./mitmdump -s AddHeader.py
```

## 使用 addons：ModifyResponse

```shell
vi ModifyResponse.py
###
from mitmproxy import ctx
from mitmproxy import http


class ModifyResponse:
    def response(self, flow: http.HTTPFlow) -> None:
        if flow.request.url == 'https://www.baidu.com/':
            flow.response.content = flow.response.content.decode('utf8').replace(
                '<title>百度一下，你就知道</title>', '<title>mitmproxy</title>'
            ).encode('utf8')
            ctx.log.info("修改title为mitmproxy")


addons = [
    ModifyResponse()
]
###
./mitmdump -s ModifyResponse.py
```

