---
title: manjaro安装vscode
date: 2018-08-07 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: manjaro安装vscode
categories: Linux
tags:
  - linux
  - vscode
---

# manjaro安装vscode
首先官网去下载安装包[vscode官网](https://code.visualstudio.com/) code-stable-xxxxxxx.tar.gz

解压安装包到/opt/目录
`sudo tar -zxvf code-stable-1562627471.tar.gz -C /opt/`

加上执行权限
`sudo chmod +x /opt/VSCode-linux-x64/code`

加入系统环境
`ln -s /opt/VSCode-linux-x64/code /usr/local/bin/code`
code就可以直接启动了

在/usr/share/applications/下创建visualstudiocode.desktop文件
`vim /usr/share/applications/visualstudiocode.desktop`
拷贝如下内容：
[Desktop Entry]
Name=Visual Studio Code
Comment=Multi-platform code editor for Linux
Exec=/opt/VSCode-linux-x64/code
Icon=/usr/share/icons/code.png
Type=Application
StartupNotify=true
Categories=TextEditor;Development;Utility;
MimeType=text/plain;

图标
`sudo cp /opt/VSCode-linux-x64/resources/app/resources/linux/code.png /usr/share/icons/`