---
title:  Centos 7 上安装 Mysql 5.7
...

### 安装Mysql 5.7
```
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql-community-server
# 上面这步可能会出现错误：mysql-community-common-5.7.43-1.el7.x86_64.rpm 的公钥尚未安装， 解决办法： `rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022` 参考： https://blog.csdn.net/qq_53810226/article/details/124836467
systemctl start  mysqld.service
systemctl enable  mysqld.service
```
获取随机生成的密码： `grep "password" /var/log/mysqld.log  `
进一步运行其他的statement前必须要修改密码: `ALTER USER 'root'@'localhost' identified by 'Abc#123456';`

连接mysql： `mysql -u root -p`

修改用户密码及访问限制
```SQL
use mysql;
ALTER USER 'root'@'localhost' identified by 'Abc#123456';
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
-- UPDATE USER SET HOST = '%' WHERE USER = 'root'; // 这个不生效
update user set host = '%' where user = 'root';
FLUSH   PRIVILEGES;
```
