---
title: mysql主从复制
date: 2018-08-18 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: mysql主从复制
categories: 数据库
tags:
  - mysql
---


## 环境
mysql 5.7 
centos7.0
192.168.1.109（主） 192.168.1.109（从）


## 配置主从同步
#日志文件名称
log-bin=master-a-bin
#二进制日志格式，有row、statment、mixed三种类型
binlog-format=ROW
#各服务器id必须不一样
server-id=1
#同步的数据库名称
binlog-do-db=zc

## 配置从服务器登录主服务器的帐号授权
grant replication slave on *.* to 'root'@'192.168.1.19' identified by 'Mysql,218219.';
flush privileges;

## 从服务器的配置（my.cnf）
#日志文件名称
log-bin=master-a-bin
#二进制日志格式，有row、statment、mixed三种类型
binlog-format=ROW
#各服务器id必须不一样
server-id=2

## 重启主服务器mysql
systemctl restart mysqld.service
show master status\G;

grant all privileges on *.* to 'root'@'%' identified by 'Mysql,218219.' with grant option;
grant all  on *.* to 'root'@'%' identified by 'Mysql,218219.' with grant option;
flush privileges;

## 重启从服务器mysql
systemctl restart mysqld.service

change master to master_host='192.168.1.19',master_user='root',master_password='Mysql,218219.',master_port=3306,master_log_file='master-a-bin.000003', master_log_pos=578;

#开启从服务器
start slave;
show slave status\G;

## 如果从服务器链接不上主服务器
systemctl disable firewalld
service firewalld stop

## Slave_IO_Running: NO的原因
如果安装了一台linux 又克隆了两台，一主两从 , 关键点就在于我是克隆的，才导致了报Slave_IO_Running: NO 
原因：mysql 有个uuid , 然而uuid 是唯一标识的，所以我克隆过来的uuid是一样的，只需要修改一下uuid 就ok了，找到auto.cnf 文件修改uuid。auto.cnf文件一般在 /var/lib/mysql/auto.cnf , 如果没有那就用linux 查询命令找：find -name auto.cnf