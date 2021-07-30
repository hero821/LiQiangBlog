---
title: 在Termux中安装mitmproxy
date: 2021-07-26 23:00:00
tags:
- Termux
- mitmproxy
categories:
- [Termux]
---

# 安装

安装 termux ，可以参考 {% post_link 安装Termux 这里 %}

安装 mitmproxy

```shell
pkg install libffi openssl python rust
pip install mitmproxy
mitmproxy --version
```

# 测试

测试 mitmproxy ，可以参考 {% post_link 安装mitmproxy 这里 %}
