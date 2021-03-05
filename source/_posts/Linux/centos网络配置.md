---
title: centos网络配置
date: 2018-08-07 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: centos网络配置
categories: Linux
tags:
  - linux
  - centos
---

# centos网络配置
```bash
vim /etc/sysconfig/network-scripts/xxxx

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=d80e9953-d03e-4af1-9a1f-84f15732e8a3
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.198.130
PREFIX=24
GATEWAY=192.168.198.2
PEERROUTES=no
DNS1=8.8.8.8
```