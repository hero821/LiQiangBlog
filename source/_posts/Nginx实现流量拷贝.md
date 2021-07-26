---
title: Nginx实现流量拷贝
date: 2021-07-26 22:00:00
tags:
- Nginx
categories:
- [Nginx]
---

# 基于 Nginx Mirror 模块

## 配置

```shell
vi /etc/nginx/conf.d/default.conf
###
server {
  listen    8080;
  server_name localhost;
  location / {
    mirror /mirror;
    proxy_pass http://192.168.0.2:30000;
  }
  location /mirror {
    internal;
    proxy_pass http://127.0.0.1:8888$request_uri;
  }
}
###
nginx -s reload
```

## 测试

```shell
# 启动 fiddler 用于抓包，监听端口 8888
# 请求
curl -L -X POST 'http://localhost:8080/swagger-ui.html?param1=value1' \
-H 'Content-Type: application/json' \
-d '{
  "param2": "value2"
}'
# 在 fiddler 中，可以看到相关记录
```

<!-- more -->

# 基于 lua 脚本

## 配置

```shell
# TODO
```

## 测试

```shell
# TODO
```

# 参考

http://nginx.org/en/docs/http/ngx_http_mirror_module.html