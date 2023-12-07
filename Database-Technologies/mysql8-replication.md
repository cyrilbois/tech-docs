---
title:  基于Centos 7.5 搭建 Mysql 8 的复制
...

这可以在其中一个数据库服务器发生故障时提供冗余，并且可以提高数据库的可用性、可伸缩性和整体性能。 跨多个独立数据库同步数据的做法称为复制。 

 过去，这种类型的数据库复制被称为“主从” (master-slave) 复制。 MySQL 团队在2020年发文[MySQL Terminology Updates](https://dev.mysql.com/blog-archive/mysql-terminology-updates/)解释过 这种术语的负面问题，并宣布他们努力更新数据库程序及其文档以使用更具包容性的语言。 


|旧的术语 	| 新的术语|
| -------- | -------- | 
| master 	| source| 
| slave 	| replica| 
| blacklist 	| blocklist| 
| whitelist | 	allowlist| 

## Mysql 8 安装及配置

### 安装
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

> 修改root用户密码： `ALTER USER 'root'@'localhost' IDENTIFIED BY 'Abc#123456';`。 

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
## 创建用户的时候设置
CREATE USER 'demo'@'%' IDENTIFIED WITH mysql_native_password BY 'Demo#123';

## 复制库最好赋予只读权限
GRANT SELECT ON demo.* to 'demo'@'%';

FLUSH PRIVILEGES;

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


## 配置 Replication
完成souce和replica数据库的安装和目录的配置后，进入配置复制阶段。  参考文章：https://www.digitalocean.com/community/tutorials/how-to-set-up-replication-in-mysql

### 源数据库Mysql配置

概述来讲需要做如下配置：
1. 开始binlog， 配置binlog。（默认此版本和对应的安装已经默认开启， 具体log在配置的datadir目录下）
2. 设置server-id；  默认是1。 源库可以不用设置，用默认的也行
3. 设置 relay-log=relay-bin.log    这里如果不设置会有异常 `[Warning] [MY-010604] [Repl] Neither --relay-log nor --relay-log-index were used; so replication may break when this MySQL server acts as a slave and has his hostname changed!! Please use '--relay-log=localhost-relay-bin' to avoid this problem.`
4. 确定目标库； 确定哪些是目标库或者说排除哪些
5. 创建复制使用的用户

#### 开启binlog
mysql8 安装在centos 上后默认是开启了binlog的， 这个可以从my.conf的注释中确认。
```
# Binary logging captures changes between backups and is enabled by
# default. It's default setting is log_bin=binlog
```

#### 设置 server-id
server-id的具体信息参看： https://dev.mysql.com/doc/refman/8.0/en/replication-options.html

查看和验证现有server-id方式: `mysql> select @@server_id`

通过在配置文件 `my.conf` 修改 source库mysql的`server-id=1`(默认是1)， replica库的mysql的`server-id=2` 。

#### 确定目标库
复制哪些库可以通过参数设置， 一般分两个参数： `binlog_ignore_db`  和 `binlog_db_db`。 一般通过ignore来排除不需要被复制的库， 通过编辑my.conf文件：

```
binlog_ignore_db=mysql
```
> 用ignore的方式， 新建库也会自动被复制

#### 创建复制使用的用户

```
CREATE USER 'replica_user'@'10.0.9.131' IDENTIFIED WITH mysql_native_password BY 'Replication#20220221';
GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'10.0.9.131';
FLUSH PRIVILEGES;
```

### 复制库Mysql配置

概述来讲需要做如下配置：
1. 开始binlog， 配置binlog。（默认此版本和对应的安装已经默认开启， 具体log在配置的datadir目录下, 保持和源库一致）
2. 设置server-id；  默认是1， 一般需要修改， 改成不和源库重复。 如： server_id=2;
3. 确定目标库； 确定哪些是目标库或者说排除哪些； 保持好和源库一致


#### 准备复制
MySQL 通过从源的二进制日志文件中逐行复制数据库事件并在副本上实现每个事件来实现复制。当使用 MySQL 的基于位置的二进制日志文件复制时，您必须为副本提供一组坐标，这些坐标详细说明了源二进制日志文件的名称开始复制数据库事件的点。为了确保在您检索坐标时没有用户更改任何数据，这可能会导致问题，您需要锁定数据库以防止任何客户端在您获取坐标时读取或写入数据。

##### 锁表并获取binlog和初始位置
```
FLUSH TABLES WITH READ LOCK;
```

查看当前binlog文件和文件 `SHOW MASTER STATUS`:

![master-status.png](http://tech.jiu-shu.com/Database-Technologies/master-status.png)

> 记录File 和 Position 备用

接下来有两个场景： 
1. 不存在已有数据需要迁移；【源库：解锁表】 -> 【复制库：创建库】 -> 【复制库：配置复制源】 -> 【复制库：开始复制】
2. 存在已有数据需要迁移;   源库：【dump数据】-> 【源库：解锁表】 -> 【复制库：创建库导入数据】 -> 【复制库：配置复制源】 -> 【复制库：开始复制】

> 注意及时解锁 `UNLOCK TABLES;`  

#####  配置复制源
在复制数据库端配置:
```
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='10.0.9.132',
SOURCE_USER='replica_user',
SOURCE_PASSWORD='Replication#20220221',
SOURCE_LOG_FILE='binlog.000002',
SOURCE_LOG_POS=2290;
```

##### 开始复制

`START REPLICA;`   查看状态 `SHOW REPLICA STATUS\G;`

##### 验证
源库操作， 复制库验证











