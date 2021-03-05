---
title: manjaro装错驱动开机黑屏
date: 2018-08-18 15:25:00
author: huyiwan
img: https://i.loli.net/2021/03/05/hQlfxVNEwFrUHZC.png
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: manjaro黑屏
categories: Linux
tags:
  - linux
  - manjaro装错驱动开机黑屏
---

# manjaro装错驱动开机黑屏
# 问题描述
manjaro开机直接黑屏，无法进入桌面

# 问题原因
回想上次关机前的相关操作，大概锁定了问题原因，就是手贱点击装了图中那玩意儿video-vesa后，开机直接黑屏。所以不要手贱安装。
![20190818044425611_2046.png](https://i.loli.net/2021/03/05/hQlfxVNEwFrUHZC.png)

# 解决办法，卸载安装的那个驱动
[参考文档https://wiki.manjaro.org/index.php/Configure_Graphics_Cards](https://wiki.manjaro.org/index.php/Configure_Graphics_Cards)
开机，等机器启动起来，因为一直是黑屏的，看不到启动好没有，所以多等几秒，然后Ctrl+Alt+F3进入tty3输入密码进入系统。
输入命令 `mhwd -li`
```bash
Installed PCI configs:
--------------------------------------------------------------------------------
                  NAME               VERSION          FREEDRIVER           TYPE
--------------------------------------------------------------------------------
           video-linux            2018.05.04                true            PCI
           video-vesa             2017.06.08                true            PCI

Warning: No installed USB configs!
```
找到了罪魁祸首video-vesa ，直接卸载 `mhwd -f -r pci video-vesa`
OK，`reboot` ，问题解决。