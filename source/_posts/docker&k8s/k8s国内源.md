---
title: k8s国内源
date: 2018-08-07 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: k8s国内源
categories: docker&k8s
tags:
  - dokcer
  - k8s
---

# k8s国内源
```bash
cat > /etc/yum.repos.d/kubernetes.repo <<EOF 
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```