---
title:  Centos 7 上安装 Mysql 5.7
...

### 安装Mysql 5.7
```
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql-community-server
systemctl start  mysqld.service
systemctl enable  mysqld.service
```
获取随机生成的密码： `grep "password" /var/log/mysqld.log  `

连接mysql： `mysql -u root -p`

修改用户密码及访问限制
```
use mysql;
ALTER USER 'root'@'localhost' identified by 'Abc#123456';
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
UPDATE USER SET HOST = '%' WHERE USER = 'root';
update user set host = '%' where user = 'root';
FLUSH   PRIVILEGES;
```
