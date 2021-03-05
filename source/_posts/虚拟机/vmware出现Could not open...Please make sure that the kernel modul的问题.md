---
title: vmware出现Could not open...Please make sure that the kernel modul的问题
date: 2018-08-18 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: manjaro vmware Could not open.
categories: 虚拟机
tags:
  - manjaro
  - vmware
---

# vmware出现Could not open...Please make sure that the kernel modul的问题

打开vmware出现提示框，显示如下内容，进不去虚拟机
Could not open /dev/vmmon: ?????????.
Please make sure that the kernel module `vmmon' is loaded.

![20190818034018848_8908.png](https://i.loli.net/2021/03/05/tam8dYFzUPyxZCG.png)

在终端输入命令 `sudo /etc/init.d/vmware start`再打开虚拟机就能进去了。

```bash
[huhui@huhui-pc ~]$ sudo /etc/init.d/vmware start
[sudo] huhui 的密码：
Starting VMware services:
   Virtual machine monitor                                             done
   Virtual machine communication interface                             done
   VM communication interface socket family                            done
   Blocking file system                                                done
   Virtual ethernet                                                    done
   VMware Authentication Daemon                                        done
   Shared Memory Available                                             done
```