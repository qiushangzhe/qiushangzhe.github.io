---
layout: post
title: 在linux下安装mysql
date: 2018-04-27 00:00:00
key: 2018-04-27
category: op
tags: [op,mysql]
---

## 安装

#### 0.提前准备：删除已安装的mysql

- 查看已安装的mysql `rpm -qa |  grep mysql`

- 删除方法

	- sudo rpm -e --nodeps 包名

#### 1. 第一步下载（去官网选好自己的版本）

<!-- more -->

- wget -c https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-common-5.7.21-1.el7.x86_64.rpm

- wget -c https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-client-5.7.21-1.el7.x86_64.rpm

- wget -c https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-server-5.7.21-1.el7.x86_64.rpm

- wget -c https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-libs-5.7.21-1.el7.x86_64.rpm

#### 2. 第二步安装：
- 一定按照顺序common --> libs --> clients --> server

- rpm -ivh mysql-community-common-5.7.21-1.el7.x86_64.rpm
- rpm -ivh mysql-community-libs-5.7.21-1.el7.x86_64.rpm
- rpm -ivh mysql-community-client-5.7.21-1.el7.x86_64.rpm
- rpm -ivh mysql-community-server-5.7.21-1.el7.x86_64.rpm

#### 3. 第三步骤开启mysql
sudo service mysqld start

#### 4. 第四步骤 登录

- 密码提前找到：

- mysql5.7以上的默认密码查看方式
grep "temporary password" /var/log/mysqld.log

- mysql -uroot -p登录

- 安装完mysql 之后，登陆以后，不管运行任何命令，总是提示这个
```
step 1: SET PASSWORD = PASSWORD('一个随机的密码');
step 2: ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
step 3: flush privileges;
```

over

https://www.cnblogs.com/frankielf0921/p/7258137.html
https://www.cnblogs.com/520playboy/p/6231158.html

## 其他
- 给某个数据库添加账号和权限

1. grant create,select,insert,update,delete on （数据库名）.*（数据表名，*为所有表） to 账号@"%" Identified by "密码";
2. flush privileges;