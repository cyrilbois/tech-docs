---
title: Centos 常用命令
sort: 1
description: Centos 常用命令
...

> 以下命令仅在centos7上验证过

### 查看系统信息
```
uname -a
hostnamectl
cat  /etc/redhat-release  # 系统详细信息
```

 - 查看多少个CPU `grep 'model name' /proc/cpuinfo | wc -l`

### 软件安装
#### CentOS7配置阿里云yum源和EPEL源
 以centos7为例安装阿里云yum源
```
cd /etc/yum.repos.d/
mkdir repo_bak
mv *.repo repo_bak/
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
## ready to install software
```
EPEL源
```
yum list | grep epel-release
yum install -y epel-release
# 如果出现错误：没有可用软件包 epel-release， 可采用  `yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm`
```
https://www.cnblogs.com/jimboi/p/8437788.html  配置好源了， 安装啥都方便了
#### yum 安装
示例如下：
```
sudo yum install elasticsearch
```

#### rpm  命令

**安装**
```
sudo rpm -ivh kibana-4.6.6-x86_64.rpm  // 安装后通过 sudo service kibana start 来启动
```
**查看安装程序路径**
```
sudo rpm -ql kibana  // 查看到安装在了/opt/kibana
```
```
sudo rpm -qa|grep jdk  // 查看安装了哪些jdk
sudo rpm -ql java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64 //查看具体包的安装路径
```

#### 开机启动服务
```
systemctl start xxx.service #启动
systemctl stop xxx.service #停止
systemctl enable xxx.service (开机启动)
systemctl disable xxx.service (禁止开机启动)
systemctl enable docker # 开机启动docker
```
#### 查看服务日志
如果是通过systemd启动，可使用如下命令
```
journalctl -u kubelet
```
### 时间同步 Chrony

启动 Chrony 来同步时间： 
```
systemctl start chronyd.service
systemctl enable chronyd.service
```
Chrony是一个开源自由的网络时间协议 NTP 的客户端和服务器软软件。它能让计算机保持系统时钟与时钟服务器（NTP）同步，因此让你的计算机保持精确的时间，Chrony也可以作为服务端软件为其他计算机提供时间同步服务。

### ssh 登录
ssh-copy-id -i id_rsa.pub osboxes@192.168.1.186
###  延长SSH会话
编辑`vim /etc/ssh/sshd_config`
```
ClientAliveInterval 30  #客户端每隔多少秒向服务发送一个心跳数据
ClientAliveCountMax 1800 #客户端多少秒没有相应，服务器自动断掉连接
```
重启sshd服务`service sshd restart`

### 修改主机名称
 - 方式一 hostnamectl set-hostname sino-dev
 - 方式二 `vi /etc/sysconfig/network`
```
NETWORKDING=yes
HOSTNAME=SINO-DEV
```
通过`service network restart`重启网络服务生效。查看运行`hostname`
### 开放端口

```
$ sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
$ sudo firewall-cmd --reload
```

### netstat 使用
#### 查看某个服务是否在运行

```
sudo netstat -aple | grep nginx
```
只查看tcp或者udp的connections需要添加-t参数: `sudo netstat -nplt` 
> 更多更实用的netstat命令参考：[Linux netstat 命令示例](http://www.binarytides.com/linux-netstat-command-examples/)

### 查看centos 版本
```
cat /etc/centos-release
```
### 设置环境变量 

```
export KAFKA_HOME=/home/osboxes/kafka_2.10-0.10.0.1
echo $KAFKA_HOME
```
在 /etc/profile 文件中设置的变量是全局变量。而 .bashrc文件（在用户的家目录下）则只对当前用户有用。~/.bashrc、~/.bash_file 是当前用户目录下的配置信息。修改后用 source 命令更新。

### SELinux
```
getenforce #  查看是否开启
sudo setenforce 0 # 关闭
sudo sed -i ‘s/^SELINUX=enforcing$/SELINUX=permissive/’ /etc/selinux/config  # 永久关闭
```
### 磁盘空间
```
df -h
```
```
free
```
### 查看目录所占空间大小

```
du -smh *
```
### 统计
```
ls -1 | wc -l   //统计文件夹下面的文件数量
wc - lc  file1 //统计文件内容的行数
```

### 搜索文件

```
find . -name .project // 搜索当前目录及子目录下的 名称为 '.project'的文件
find . -name .project  -exec rm -f {} \;   // 搜索出来并删除掉
```

### grep 搜索文件内容
**指定的文件类型中查找**
当前子目录中查找： `grep -r abcd *.properties` 当前子目录递归查找含有`abcd` 的*.properties 文件
指定目录及子目录中查找：`grep -r 3306 /home/okchem/storage92g/srm/ ` 

### 拷贝整个文件夹
```
cp -avr /home/vivek/letters /usb/backup
```
### 用户和组
> /etc/group file that lists all users groups  可以使用cut命令列出来`cut -d: -f1 /etc/group`


 - **查看当前用户的group:	** `$ groups`
 - **查看用户的group:	** `$ groups root` `id -Gn root`
 - **添加用户到组:	** `$ sudo usermod -a -G osboxes nginx` 添加用户nginx到组osboxes  `usermod -a -G <groupname> username` 添加完成请用`groups <username> ` 来验证
 - **获取组的所有用户:	** `getent group kibana`

其他有用资源： [CentOS7之新建用户与SSH登陆](https://segmentfault.com/a/1190000004141370)

### 文件编码
```
file -i <文件名>
vim --cmd 'set encoding=utf-8' <文件名> 
```

### 文件权限

通过`ls -l 文件路径`获取类似信息：`- rw- r-x r- - (数量) userame groupname mtime filename`
> 这里的数量，如果是文件，指的是数据库连接的文件数量， 如果是文件夹暂时不知  :(

**文件权限的信息如下：**
- rw- 文件属主的权限
- r-x 文件属组的权限
- r-- 其他用户的权限

**文件夹有所不同**
- x 进入目录
- rx 显示目录内的文件名
- wx 修改目录内的文件名

#### 修改文件(夹)owner
chown 代表change owner；`chown --help` 提供了更详细的信息
```
sudo chown -R joe:root /ebs # 修改了owner为joe 组为root
sudo chown -R joe /ebs # 修改 /ebs 的owner为joe
```
#### 修改文件[夹]访问权限
chmod 代表change mode; 
例如：`chmod 644 important.txt` owner可读可写,group可读，others可读
> First position refers to the user. Second refers to the group of the user, and the third refers to all others. 4 = read 2 = write 1 = execute

添加执行权限也可以用:`chmod +x test.sh `, 这样会给owner+group+others都添加上执行权限。 

如果想查看文件/文件夹八进制模式值，可以通过 `stat -c "%a %n"  <filename>`

通过`man stat` 可以发现
```
-c  --format=FORMAT
          use  the  specified  FORMAT instead of the default; output a newline after
          each use of FORMAT
%a     Access rights in octal
%n     File name
```

文件权限更详细的解释可以参考：[Linux File Permissions](https://www.pluralsight.com/blog/it-ops/linux-file-permissions)
### PS 命令
#### 查看java进程 `ps -ef|grep java`
#### 产看进程的详细信息 `ps -auxwe | grep subscribe`

```
ps -ef|grep java  
ps -auxwe | grep subscribe // 查看进程更详细的信息
lsof -p 110559 | grep cwd   // 查看进程执行路径
```
### 命令行快捷键
**CTRL-a** 光标移至行首
**CTRL-e** 光标移至行尾
**CTRL-u** 删除整行
**CTRL-h**	删除光标前字符

### gzip / gunzip
 压缩单个文件 `gzip fileName` 压缩后的名字=原文件名字加上后缀.gz
 解压缩单个文件`gunzip filename` 或者 `gzip -d filename`
gzip 不能用来压缩整个文件夹至一个.gz 文件。压缩整个文件夹请参考targ + gzip 命令 即： `tar -z`命令。
> gzip -r  dictName 命令会压缩整个文件夹dictName 里面的所有文件，每个文件被压缩成一个单独的*.gz 文件
### tar 命令
gzip / bzip2 是用来压缩单个文件， tar是用来归档。 所以tar结合gzip/bzip2 可以方便的进行整个文件夹的压缩及归档。
压缩整个文件夹`tar -zcvf outputFileName folderToCompress` 
[Examples](https://www.tecmint.com/18-tar-command-examples-in-linux/)
```
tar -xvf videos-14-09-12.tar.bz2 // 解压
```
> bzip2 

### sftp 命令
#### sftp登录
```
sftp  name@123.21.331.1
```
#### sftp 下载文件(夹)
```
get  /home/joe/test.lsq  /home/xxx/
get -r  folder  /home/joe/
```
#### sftp 上传文件
```
put  /name1.html  /name2/
```

### scp 命令
```
scp PLC.png root@10.3.69.65:/www/media/images/   ##复制当前目录下的PLC.png 到目标机器的/www/media/images/ 
```

### crontab 命令
创建执行任务， 添加cron job
参考cronjob `crontab -l`
编辑cronjob `crontab -e`

```
0 1 * * * /data/scripts/mysql-job.sh A  // 每天1点执行
20 1 * * 0 /data/scripts/mysql-job.sh I  // 每周日1点20 执行
```
Cron Job的日志位置： /var/log/cron
参考：crontab 时间可以参考： https://www.cnblogs.com/intval/p/5763929.html
> 注意cron的时间有可能和date命令的时间不一致。`tail -f /var/log/cron` 这个命令可以查看cron的时间。 当执行`crontab -e`的时候`/var/log/cron`会有记录。

### 后台运行
```
nohup ./startAgent.sh > /dev/null 2>&1 &
```

### 其他
- java安装路径查看： update-alternatives --display java

### base64
```
echo -n 'admin' | base64
echo 'YWRtaW4=' | base64 --decode
```

### stress 安装
centos7 安装包： https://centos.pkgs.org/7/epel-x86_64/
```
wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/s/stress-1.0.4-16.el7.x86_64.rpm
yum localinstall stress-1.0.4-16.el7.x86_64.rpm
stress --cpu 4 --timeout 600 // 压测 4 个cpu情况
stress --vm 4 --vm-bytes 512m --vm-hang 100 --timeout 600s  // 压测内存
stress -d 1 --hdd-bytes 3G // 写数据
```

### 后台启停Jar
启动脚本：
```
#！ /bin/bash

nohup java -jar sdk-api.jar > /dev/null 2>&1 &
echo $! > sdk-api.pid
```
停止脚本
```
#！ /bin/bash

kill -9 $(cat sdk-api.pid)
rm -f sdk-api.pid
```

### 文件同步

参考链接使用rsync同步文件或文件夹
https://geekdudes.wordpress.com/2019/08/28/configuring-rsync-centos-7/


### 安装python3


```
sudo yum install -y https://repo.ius.io/ius-release-el7.rpm
// 国内无法访问，可以翻*下载rpm文件传上去安装

sudo yum update
sudo yum install -y python36u python36u-libs python36u-devel python36u-pip
```

### vim 语法高亮

便捷 ~/.vimrc 添加：
```
syntax on
```

### shasum 未找到命令
使用 `yum install perl-Digest-SHA`进行安装