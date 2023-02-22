---
title: Mysql 运维相关脚本收集
description: Mysql 运维相关脚本收集
...
mysql 版本： 5.6

# 安装：
centos 默认会安装mariadb, 如果要安装mysql需要手动操作。
https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-centos-7

Mysql 常用命令： https://www.runoob.com/mysql/mysql-administration.html
# 建库及用户
创建数据库dbname及用户dbuser/dbpassword 并授权数据全部权限给用户dbuser
```sql
CREATE DATABASE  IF NOT EXISTS `dbname` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_bin */
grant all privileges on dbname.* to dbuser@localhost identified by 'dbpassword';
```

示例： 创建demo数据库及用户demo/demo#123 bing
```
CREATE DATABASE  IF NOT EXISTS `demo` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_bin */
GRANT ALL PRIVILEGES ON demo.* TO 'demo'@'%' IDENTIFIED BY 'demo#123' 

```
开启远程访问参考： https://www.digitalocean.com/community/tutorials/how-to-allow-remote-access-to-mysql


# SQL 收集
## 找出有记录的表
```
SELECT  * FROM  INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'test' and table_rows > 0;
```

## 查询表的所有列的信息
```
SHOW FULL COLUMNS  FROM `order`;
```
## 查询索引信息
```
SHOW INDEX FROM <表名> [ FROM <数据库名>]
```
## 粗略找出大记录的表
```
SELECT TABLE_ROWS, TABLE_NAME     FROM INFORMATION_SCHEMA.TABLES      WHERE TABLE_SCHEMA = 'test' ORDER BY TABLE_ROWS DESC;
```
## 快速删除树形表数据
如何快速删除树形比如：ProductCategory 这类模型的数据：
```sql
SET FOREIGN_KEY_CHECKS=0;
DELETE FROM okchem.ProductCategory where id > 0;  -- id>0 可以去除错误
SET FOREIGN_KEY_CHECKS=1;
```
> 采用where条件`where id > 0`可以去除如下错误：Error Code: 1175. You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column To disable safe mode, toggle the option in Preferences -> SQL Editor and reconnect.

## 快速查询表的依赖
查询表依赖那些表和查询那些表依赖此表； 

### 查询我依赖的：
```
SELECT TABLE_NAME,
       COLUMN_NAME,
       CONSTRAINT_NAME,
       REFERENCED_TABLE_NAME,
       REFERENCED_COLUMN_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = "schemaName" 
      AND TABLE_NAME = "TableName" 
      AND REFERENCED_COLUMN_NAME IS NOT NULL;
```

### 查询依赖‘我的’：
```
      
SELECT TABLE_NAME,
       COLUMN_NAME,
       CONSTRAINT_NAME,
       REFERENCED_TABLE_NAME,
       REFERENCED_COLUMN_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = "schemaName" 
      AND REFERENCED_TABLE_NAME = "TableName";
```

##  删除重复的行

```
DELETE t1 FROM contacts t1
        INNER JOIN
    contacts t2 
WHERE
    t1.id < t2.id AND t1.email = t2.email; 
```
// 当表的记录太多，这种join很危险， 最好的方式是先查出来重复的email，再加上in的条件
```
DELETE t1 FROM contacts t1
        INNER JOIN
    contacts t2 
WHERE
    t1.id < t2.id AND t1.email = t2.email and t2.email in ('..',,,,,,,); 
```


## 使用函数
### 为空的时候给默认值
```
select ifnull(p.isActive,0) from product
```
### 转换成JSON

## 创建Function
### Split delimited strings
参考： https://blog.fedecarg.com/2009/02/22/mysql-split-string-function/

```
CREATE FUNCTION SPLIT_STR(
  x VARCHAR(255),
  delim VARCHAR(12),
  pos INT
)
RETURNS VARCHAR(255)
RETURN REPLACE(SUBSTRING(SUBSTRING_INDEX(x, delim, pos),
       LENGTH(SUBSTRING_INDEX(x, delim, pos -1)) + 1),
       delim, '');
```
# Mysql 导出导入
## 导出
```
mysqldump -uroot -proot --databases db1 db2 >/tmp/db1_and_db2.sql
mysqldump -u <db_username> -h <db_host> -p db_name table_name > table_name.sql  // 导出某个表
```
示例：`mysqldump -h 192.168.1.90 -u root -p demo > ./demo.sql` 导出demo数据库。`mysqldump -h 192.168.1.90 -u root -p demo HotWordsCopy > ./HotWordsCopy.sql` 导出demo数据库中的HotWordsCopy表。

> 如果设置了不能从本地访问，就需要从其他机器采用制定制定host来导出。
## 导入
```
shell> mysql –u root –p target_db_name < backup-file.sql
```
或者选定数据库，使用source命令
# Mysql 分库备份脚本
```
#!/bin/sh
#Backup databases into separated files excluding system schemas
BACKUP_FOLDER=/home/okchem/mysqlbackup
MYUSER=user
MYPASS=password
SOCKET=/data/mysql/mysql.sock
MYCMD="mysql -u$MYUSER -p$MYPASS -S $SOCKET"
MYDUMP="mysqldump -u$MYUSER -p$MYPASS -S $SOCKET"

mkdir -p ${BACKUP_FOLDER}

#for database in `$MYDUMP -e "show databases;"|sed '1,2d'|egrep -v "mysql|schema"`
for database in `$MYCMD -e "show databases;" | egrep -Evi "database|mysql|schema|test"`
do 
	$MYDUMP $database >${BACKUP_FOLDER}/${database}_$(date +%Y%m%d).sql
    #If compression is needed, use this command: $MYDUMP $database |gzip >/server/backup/${database}_$(date +$F).sql.gz
done

```


# Mysql 客户端导出数据

在mysql 服务端可以很方便的导出到文件，也有灵活的选择。 如果需要导出的文件到其他服务器，不在mysql服务器上。 有两个选择：

 1. 在mysql 服务器上导出文件，通过sftp上传至目标机器
 2. 在目标机器安装mysql 客户端，通过shell 脚本来导出数据 （此篇关注点）

## 验证环境
Linux 系统：Centos 7
## 安装Mysql Client
参考：[Installing MySQL on Linux Using RPM Packages from Oracle](https://dev.mysql.com/doc/refman/5.7/en/linux-installation-rpm.html)

## Shell 脚本

``` shell
#!/bin/sh
##############################################################################################################################################
# This script is used to retrieve data from mysql and output it into txt file. Also it will generate md5 file which can be used to verify the integrity.
# Script will make folder named "YYYYMMDD", also the file name will follow the pattern A/I{tableName}YYYYMMDD{6 sequence number} such as I0100320170303000001
##############################################################################################################################################

##############Global Configuration begins ####################
# Root folder where the data will be stored
BEE_ROOT_GLOBAL=/data/b2bbuyerdata
MYSQL_HOST=192.168.1.90
MYSQL_PORT=3306
MYSQL_USERNAME=username
MYSQL_PASSWD=password
##############Global Configuration ends ####################

# exportAndMD5Sum querySql tableName. Output the query result into tableNameYYYYMMDD000001.txt and tableNameYYYYMMDD000001.md5
# .md5 file is used to verify data integrity. 
exportAndMD5Sum()
{
	if [ "$#" != 2 ];then
		echo  "Usage: exportAndMD5Sum querySql tableName";
		exit;
	fi
	# Starting export data using mysql command
	SQL=$1;
	tableName=$2;
	TIMESTAMP=`date +%Y%m%d`
	BEE_ROOT=${BEE_ROOT_GLOBAL}/${TIMESTAMP}
	
	_tmpFile=${BEE_ROOT}/${tableName}${TIMESTAMP}000001.tmp;
	destFile=${BEE_ROOT}/${tableName}${TIMESTAMP}000001.AVL;
	destMD5File=${BEE_ROOT}/${tableName}${TIMESTAMP}000001.CHK;
	
	# Create Folder
	[ ! -d "$BEE_ROOT" ] &&  mkdir "$BEE_ROOT"
	
	# Mysql command to output data into file
	`mysql -h ${MYSQL_HOST} -p${MYSQL_PORT} -u ${MYSQL_USERNAME} --password=${MYSQL_PASSWD} -e "${SQL}" > "${_tmpFile}"`
	
	# If not empty(has records) change the file name, otherwise remove it.
	if [ -f "$_tmpFile" ] && [ -s "$_tmpFile" ]
		then
			mv ${_tmpFile} ${destFile}
			#`md5sum ${destFile} > ${destMD5File}`
			md5=($(md5sum ${destFile}))
			echo $md5 > ${destMD5File}
		else
			rm ${_tmpFile}	
	fi
}
if [ "$1" = "I" ]; then
	echo "Starting export all data from mysql ............."
	exportAndMD5Sum "SELECT username,country,source,city,email,first_name,last_name,province,status,CAST(is_reveive_email AS UNSIGNED) AS is_reveive_email,created_stamp,last_updated_stamp FROM b2bbuyer.user" "I01001"
	exportAndMD5Sum "select u.username,a.address,a.city,a.company_name,a.country,a.first_name,CAST(a.is_default AS UNSIGNED) AS is_default ,a.last_name,a.province,a.tel_country_code,a.tel_ext,a.tel_no,a.zip_code,a.created_stamp,a.last_updated_stamp from b2bbuyer.user u inner join b2bbuyer.user_delivery_address a where a.user_id=u.id" "I01002"
	exportAndMD5Sum "select u.username,c.email,c.address,c.city,c.company_name,c.contact,c.country,c.fax_country_code,c.fax_ext,c.fax_tel_no,c.main_products,c.province,c.register_no,c.tax_no,c.tel_country_code,c.tel_ext,c.tel_no,c.website from b2bbuyer.user u inner join b2bbuyer.user_company c where c.user_id=u.id" "I01003"
else
	echo "Starting export yesterday's data from mysql ............."
	exportAndMD5Sum "SELECT username,country,source,city,email,first_name,last_name,province,status,CAST(is_reveive_email AS UNSIGNED) AS is_reveive_email,created_stamp,last_updated_stamp FROM b2bbuyer.user where last_updated_stamp < (UNIX_TIMESTAMP(CURDATE())*1000) and last_updated_stamp > ((UNIX_TIMESTAMP(CURDATE())-60*60*24)*1000)" "A01001"
	exportAndMD5Sum "select u.username,a.address,a.city,a.company_name,a.country,a.first_name,CAST(a.is_default AS UNSIGNED) AS is_default ,a.last_name,a.province,a.tel_country_code,a.tel_ext,a.tel_no,a.zip_code,a.created_stamp,a.last_updated_stamp from b2bbuyer.user u inner join b2bbuyer.user_delivery_address a where a.user_id=u.id and a.last_updated_stamp < (UNIX_TIMESTAMP(CURDATE())*1000) and a.last_updated_stamp > ((UNIX_TIMESTAMP(CURDATE())-60*60*24)*1000)" "A01002"
	exportAndMD5Sum "select u.username,c.email,c.address,c.city,c.company_name,c.contact,c.country,c.fax_country_code,c.fax_ext,c.fax_tel_no,c.main_products,c.province,c.register_no,c.tax_no,c.tel_country_code,c.tel_ext,c.tel_no,c.website from b2bbuyer.user u inner join b2bbuyer.user_company c where c.user_id=u.id and c.last_updated_stamp < (UNIX_TIMESTAMP(CURDATE())*1000) and c.last_updated_stamp > ((UNIX_TIMESTAMP(CURDATE())-60*60*24)*1000)" "A01003"
fi
```

## 添加cron job
参考cronjob `crontab -l`
编辑cronjob `crontab -e`

```
0 1 * * * /data/scripts/mysql-job.sh A
20 1 * * 0 /data/scripts/mysql-job.sh I
```
两个cron job 分别：

 1. 每天1点执行
 2. 每周日1点20 执行

参考：crontab 时间可以参考： https://www.cnblogs.com/intval/p/5763929.html
# Mysql 客户端导入数据
## 从txt文件导入
参考： https://blog.csdn.net/huihui520com/article/details/79080512

https://segmentfault.com/a/1190000009333563

```
use test;
load data infile 'D:/tmp/hotwords.txt' into table hot fields terminated by ',' lines terminated by'\r\n';
ALTER TABLE okchem.hot ADD `id` INT NOT NULL AUTO_INCREMENT PRIMARY KEY;
```
需要解决问题：--secure-file-priv option so it cannot execute this statement
```
windows下：修改my.ini 在[mysqld]内加入secure_file_priv =

linux下：修改my.cnf 在[mysqld]内加入secure_file_priv =
```

# ubuntu 系统忘记密码

* 找到 mysql 的配置目录 `/etc/mysql`
* 找到文件 debian.cnf
* 查看文件中的 debian-sys-maint 对应的密码
* 使用这个账号登录 `mysql -u debian-sys-maint -p`
* 进入mysql, 使用mysql数据库 `use mysql`;
* 更新密码 `grant all privileges on \*.* to 'root'@'%' identified by 'password' with grant option` 或者 `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`;
* 刷新权限 `flush privileges`
* 退出mysql exit; 重启mysql `service mysql restart`


# Mysql 慢查询日志
## 查看是否开启及日志文件路径
使用sql `show variables like 'slow_query%';` 来查看配置。 参考：[Mysql开启慢查询-简书](https://www.jianshu.com/p/f7fed1b3b8d6)

## 分析慢查询日志
```
mysqldumpslow -s c -t 10 /database/mysql/mysql06_slow.log  // 得到访问次数最多的10个SQL
mysqldumpslow -s t -t 10 -g “left join” /database/mysql/mysql06_slow.log // 得到按照时间排序的前10条里面含有左连接的查询语句。
```
# mysql 数据迁移
## 自增字段问题
新增表格，需要将旧的数据迁入新表。Mysql的自增字段默认行为：
1. 取最大的(比如： 创建表后，只插入一条数据， ID直接指定为9， 那么下一条插入的数据在不指定ID值的情况下，ID是10）
2. 删除数据后，ID的起点不会因为删除而改变。 （插入N条数据，假如这N条都是未指定ID的插入，也就是说下一个ID是N+1， 这个时候删除所有的数据，再以不指定ID的方式插入一条数据，这个时候ID是**N+1**）
# Mysql 系统变量配置
## windows 下安装的mysql的配置文件地址
从服务列表`services.msc` 中找到mysql的服务，右键查看属性中的“可执行文件路径”。参考：
https://blog.csdn.net/postnull/article/details/72455768
## Win 7 设置表明区分大小写
参考： https://blog.csdn.net/postnull/article/details/72455768
在my.ini 文件中添加 `lower_case_table_names=2` 