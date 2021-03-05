---
title: manjaro配置源，安装输入法和chrome
date: 2018-08-07 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: manjaro配置源，安装输入法和chrome
categories: Linux
tags:
  - linux
  - manjaro
---

# manjaro配置源，安装输入法和chrome

设置官方镜像源
`sudo pacman-mirrors -i -c China -m rank //更新镜像排名`
`sudo pacman -Syy //更新数据源`
设置 archlinuxcn 源。sudo nano  /etc/pacman.conf，在文件最后添加
```
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

导入GPG Key
`sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring`

安装vim
`sudo pacman -S vim`
安装chrome
`sudo pacman -S google-chrome`

安装google拼音输入法
```bash
sudo pacman -S fcitx-im #默认全部安装
sudo pacman -S fcitx-configtool
sudo pacman -S fcitx-googlepinyin 
```

添加输入法配置文件sudo vim ~/.xprofile
```bash
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```