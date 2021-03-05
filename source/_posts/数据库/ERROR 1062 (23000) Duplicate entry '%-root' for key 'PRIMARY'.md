---
title: ERROR 1062 (23000) Duplicate entry '%-root' for key 'PRIMARY'
date: 2018-08-18 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: ERROR 1062 (23000)
categories: 数据库
tags:
  - mysql
---


# ERROR 1062 (23000) Duplicate entry '%-root' for key 'PRIMARY'

```
use mysql
mysql> select host, user from user;
```
将相应用户数据表中的host字段改成'%'；

```
update user set host='%' where user='root';
```
ERROR 1062 (23000): Duplicate entry '%-root' for key 'PRIMARY' 不予理会
```
flush privileges;
```
重新远程连接OK