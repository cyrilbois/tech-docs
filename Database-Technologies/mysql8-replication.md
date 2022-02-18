---
title:  centos7 上 Mysql 8 复制
...

# 基于Centos 7.5 搭建 Mysql 8 的复制
这可以在其中一个数据库服务器发生故障时提供冗余，并且可以提高数据库的可用性、可伸缩性和整体性能。 跨多个独立数据库同步数据的做法称为复制。 

 过去，这种类型的数据库复制被称为“主从” (master-slave) 复制。 MySQL 团队在2020年发文[MySQL Terminology Updates](https://dev.mysql.com/blog-archive/mysql-terminology-updates/)解释过 这种术语的负面问题，并宣布他们努力更新数据库程序及其文档以使用更具包容性的语言。 


|旧的术语 	| 新的术语|
| -------- | -------- | 
| master 	| source| 
| slave 	| replica| 
| blacklist 	| blocklist| 
| whitelist | 	allowlist| 

### Mysql 8 安装

由于Centos7 偏向于Maria DB，`yum install mysql` 默认会装MariaDB。  详细安装可以参考：https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-centos-7

大致步骤总结如下：
```
# 下载rpm文件： https://dev.mysql.com/downloads/repo/yum/   注意版本 mysql80-community-release-el7-5.noarch.rpm
# 上传rpm 文件至 Centos服务器
# 安装 Package 
sudo rpm -ivh mysql******8.noarch.rpm
# 安装 Mysql 服务
yum install mysql-server
# 启动
systemctl start mysqld
# 查看状态
systemctl status mysqld
# 默认安装后是随系统启动启动，需要禁用的话 `systemctl disable mysqld`
# 查看root临时密码
grep 'temporary password' /var/log/mysqld.log
# 安全配置  照着提示来
sudo mysql_secure_installation
# 验证和测试
mysqladmin -u root -p version
```

### 建库 用户 授权

```
CREATE DATABASE  IF NOT EXISTS `demo` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_bin */;

# 本地用户
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY 'password';  
#远程用户
CREATE USER 'demo'@'%' IDENTIFIED BY 'Demo#123';

GRANT ALL ON demo.* TO 'demo'@'%';

# https://www.jianshu.com/p/9a645c473676
ALTER USER 'demo'@'%' IDENTIFIED WITH mysql_native_password BY 'Demo#123';
```

### 修改迁移数据目录

参考： https://www.digitalocean.com/community/tutorials/how-to-change-a-mysql-data-directory-to-a-new-location-on-centos-7

```
# 关闭
systemctl stop mysqld
# 确保已经关闭
systemctl status mysqld
mkdir -p /data/mysql
rsync -av /var/lib/mysql /data
mv /var/lib/mysql /var/lib/mysql.bak
```

修改 vi /etc/my.cnf
```
datadir=/data/mysql
socket=/data/mysql/mysql.sock

[client]
port=3306
socket=/data/mysql/mysql.sock

```



