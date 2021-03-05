---
title: 安装mysql8.0后初始化
date: 2018-08-18 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: MySQL8.0初始化
categories: 数据库
tags:
  - mysql
---

![20200329231019753_1768576315.png](https://i.loli.net/2021/03/05/fU94IJaBSjMx5Ni.png)

`mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql`

`systemctl start mysqld.service ` //启动mysql
`mysql -uroot -p`

进入mysql后需要修改密码才能做其他操作
`ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';`