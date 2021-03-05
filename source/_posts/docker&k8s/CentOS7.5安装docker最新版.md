---
title: CentOS7.5安装docker最新版
date: 2018-08-07 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: CentOS7.5安装docker最新版
categories: docker&k8s
tags:
  - dokcer
  - k8s
  - centos
---

# CentOS7.5安装docker最新版
# 移除旧版
```bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
# 安装一些必要的工具
`yum install -y yum-utils device-mapper-persistent-data lvm2`
# 添加软件源信息
`yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`
# 更新缓存
`yum makecache fast`
# 安装docker
`yum -y install docker-ce`
# 设置开机启动
`systemctl enable docker`
`systemctl start docker`
# 验证是否安装成功
`docker info`