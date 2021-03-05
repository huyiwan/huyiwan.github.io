---
title: ubuntu18.04新装mysql修改root密码 && 设置字符集为utf8
date: 2018-08-18 15:25:00
author: huyiwan
img: 
top: false
cover: true
coverImg: 
password: 
toc: false
mathjax: false
summary: ubuntu新装mysql修改密码，设置字符集
categories: 数据库
tags:
  - linux
  - ubuntu
---


# ubuntu18.04新装mysql修改root密码 && 设置字符集为utf8

MySQL 5.7不再弹出root密码设置
`sudo vim /etc/mysql/debian.cnf`
 
显示：
Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint
password = fPw**********b22
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = fPw**********b22
socket   = /var/run/mysqld/mysqld.sock
 
找到了，用户名：debian-sys-maint，密码：fPw**********b22（这是随机生成的）
退出来，Esc键，输入“：q!”
mysql -u debian-sys-maint -p
Enter password:输入刚才找到的密码
 
接下来，设置root用户密码（假设为123456）
 
mysql> update mysql.user set authentication_string=PASSWORD(“123456″) where User=’root’;
更改root密码，然后出现
Query OK, 1 row affected, 1 warning (0.12 sec)
Rows matched: 1  Changed: 1  Warnings: 1
 
mysql> update mysql.user set plugin=”mysql_native_password”;
解决出现警告的问题，然后出现
Query OK, 1 row affected (0.02 sec)
Rows matched: 4  Changed: 1  Warnings: 0
 
mysql> flush privileges;
更新所有权限，然后出现
Query OK, 0 rows affected (0.33 sec)
 
mysql> quit;
 
退出来，先关闭再启动MySQL
```
sudo /etc/init.d/mysql stop
sudo /etc/init.d/mysql start
```
 
用root用户和密码进去看看
mysql -u root -p

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Ubuntu下设置MySQL字符集为utf8
1.mysql配置文件地址
/etc/mysql/my.cnf
2.在[mysqld]在下方添加以下代码
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
3.重启mysql服务
sudo service mysql restart
4.检测字符集是否更新成utf8.
进入mysql，mysql -u root -p,输入show variables like '%character%' 查看字符集
+--------------------------+----------------------------+
| Variable_name | Value |
+--------------------------+----------------------------+
| character_set_client | utf8 |
| character_set_connection | utf8 |
| character_set_database | utf8 |
| character_set_filesystem | binary |
| character_set_results | utf8 |
| character_set_server | utf8 |
| character_set_system | utf8 |
| character_sets_dir | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
注意事项：在修改字符集之前已经建立的数据库，character_set_database值不会发生改变，往数据库中插入中文数据仍然会显示乱码，所以最好在安装完MySQL后就将字符集改成utf8，否则后续修改会较麻烦。