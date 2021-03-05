---
title: vim右键不能粘贴的解决办法
date: 2018-08-07 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: vim右键不能粘贴
categories: Linux
tags:
  - linux
  - vim
---


# vim右键不能粘贴的解决办法
今天安装好manjaro后，使用vim insert的时候发现右键不能粘贴，于是百度找到如下方法解决。

修改/usr/share/vim/vim81/defaults.vim文件，不同发行版位置可能位置不一样，
我百度到的是/usr/local/share/vim/vim81/defaults.vim这个地方，
但是我的manjaro发现没有，于是
`find /usr/ -type f -name 'defaults.vim'`
发现我的是/usr/share/vim/vim81/defaults.vim这个文件
`sudo vim /usr/share/vim/vim81/defaults.vim`
大概在第70多行的地方
```bash
if has('mouse')
  set mouse=a
endif
```
把set mouse=a修改为set mouse-=a
```bash
if has('mouse')
  set mouse-=a
endif
```
:wq保存退出即可。