---
title: phpMyAdmin无法登陆解决
date: 2018-08-31 11:53:39
categories: ['后端'] 
tags: 后端
comments: true
---

### 问题

ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)

### 解析

通常这种情况是没有设置密码或者密码错误导致的



### 解决
首先确定 相关端口打开了 3306 888(默认)

1.关闭mysql

```bash
/bin/systemctl stop mysql.service
```
or
```bash
/etc/init.d/mysql stop
```

2.在mysql关闭的情况下：

```bash
/bin/systemctl stop mysql.service --skip-grant-tables
```
or
```bash
/etc/init.d/mysql start --skip-grant-tables
```

3.连接mysql,进入mysql命令行
```bash
mysql -u root -p          ## 出现password：的时候直接回车可以进入。
mysql> use mysql;         Database changed
mysql> update user set password=password("123456") where user="root";  # 给root用户设置新密码
mysql> flush privileges;       # 刷新数据库
mysql> quitBye                 # 退出mysql                                                       
```
4. 改好之后,重启mysql服务就可以了。去掉这句 `--skip-grant-tables`