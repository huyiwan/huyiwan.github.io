---
title: 生成SSH公钥
date: 2018-08-07 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: 生成SSH公钥
categories: Linux
tags:
  - linux
  - ssh
---

# 生成SSH公钥
### 按如下命令来生成 ssh key
`ssh-keygen -t rsa -C "huhui8218219@163.com"`
### 按照提示完成三次回车，即可生成 ssh key，保存在~/.ssh/id_rsa.pub，查看：
```
[root@hu2i_ferhat ~]# cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC4QOap4Nh7OGD/jY+1Ga3m... huhui8218219@163.com
```