---
title: ubuntu的virtualbox无法挂载U盘问题
date: 2018-08-18 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: ubuntu的virtualbox无法挂载U盘问题
categories: 虚拟机
tags:
  - ubuntu
  - vmware
---


# ubuntu的virtualbox无法挂载U盘问题

1. 安装“Oracle VM VirtualBox Extension Pack”，直接下载，下载后双击即可启动VirtualBox进行自动安装。这个扩展包可以为VirtualBox增加一系列的功能支持：USB2.0设备、因特尔网卡的PXE启动和VirtualBox远程显示系统等。安装之前需要关闭所有VirtualBox中的虚拟机。
2. 修改/etc/group文件（sudo vi /etc/group），找到“vboxusers:x:129:”，修改为“vboxusers:x:125:<当前用户名>”，保存即可。注意使用sudo进行操作，否则不能保存。