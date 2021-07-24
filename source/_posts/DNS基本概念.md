---
title: DNS基本概念
date: 2021-07-24 10:00:00
tags:
- DNS
categories:
- [DNS]
---

# 概念

DNS是 Domain Name System 的缩写，作用是根据域名查询IP地址

# 查询方式

## 递归查询

DNS服务器接收到客户机请求，必须使用一个准确的查询结果回复客户机。
如果DNS服务器本地没有存储查询DNS信息，那么该服务器会询问其他服务器，并将返回的查询结果提交给客户机。

## 迭代查询

DNS服务器会向客户机提供其他能够解析查询请求的DNS服务器地址。
当客户机发送查询请求时，DNS服务器并不直接回复查询结果，而是告诉客户机另一台DNS服务器地址。
客户机再向这台DNS服务器提交请求，依次循环直到返回查询的结果为止。

<!-- more -->

# DNS记录类型

A：地址记录（Address），返回域名指向的IP地址。
NS：域名服务器记录（Name Server），返回保存下一级域名信息的服务器地址。该记录只能设置为域名，不能设置为IP地址。
MX：邮件记录（Mail eXchange），返回接收电子邮件的服务器地址。
CNAME：规范名称记录（Canonical Name），返回另一个域名，即当前查询的域名是另一个域名的跳转。
PTR：逆向查询记录（Pointer Record），只用于从IP地址查询域名。

# 私有DNS解决方案

BIND
PowerDNS
Dnsmasq
CoreDNS

参考：https://en.wikipedia.org/wiki/Comparison_of_DNS_server_software