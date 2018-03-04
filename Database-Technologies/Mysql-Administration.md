---
title: Mysql 运维相关脚本收集
description: Mysql 运维相关脚本收集
---
mysql 版本： 5.6

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