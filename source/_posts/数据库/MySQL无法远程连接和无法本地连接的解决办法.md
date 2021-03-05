---
title: MySQL无法远程连接和无法本地连接的解决办法
date: 2018-08-18 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: MySQL无法远程连接和无法本地连接的解决办法
categories: 数据库
tags:
  - mysql
---

# MySQL无法远程连接和无法本地连接的解决办法

#### 无法远程的解决办法 
```bash
update user set host = '%'' where user = 'root';  //提示错误不用管，直接flush
flush privileges;
```
#### 无法本地连接的解决办法
```bash
drop user ""@localhost;
flush privileges;
```

