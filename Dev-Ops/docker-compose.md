---
title:  docker
...

## Docker官方安装方法
参考链接： https://docs.docker.com/engine/install/centos/#set-up-the-repository

优先选用官方安装方式， 安装前建议先创建sudo用户，采用sudo用户安装。

```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
# 依赖包不够新可以参考： https://stackoverflow.com/questions/65878769/cannot-install-docker-in-a-rhel-server

sudo systemctl start docker
sudo systemctl enable docker

# 测试
sudo docker run hello-world

```

如果上一步安装docker-ce的时候“没有可用软件包”， 则需要先安装 EPEL repo 和其他包
```
#install the EPEL repo and other required packages to your system.
sudo yum install epel-release wget -y

#Once the installation is completed, update your system to the latest version.
sudo yum update -y

#Next, restart your system to apply all the updates.
sudo reboot
```
## Docker 安装
运行命令： `curl -fsSL get.docker.com -o get-docker.sh && sudo sh get-docker.sh --mirror Aliyun`
如果命令报错，可以手动访问https://get.docker.com/ 下载脚本， sftp上去进行安装。 具体安装可以参考： https://www.runoob.com/docker/centos-docker-install.html

启动并设置开机启动
```
sudo systemctl enable docker
sudo systemctl start docker
```

> 如果不是云上的虚机， 最好先配置阿里云的yum源。

## Docker安装指定版本
有时候，系统比如centos版本不够新，满足不了最新的Docker 这个时候可以选择安装指定版本的Docker。 
```
yum -y install yum-utils lvm2 device-mapper-persistent-data nfs-utils xfsprogs wget
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum -y install docker-ce docker-ce-cli containerd.io
```
指定版本可以通过如下命令：
```
yum list docker-ce.x86_64 --showduplicates | sort -r  // 查找版本
```
结果类似如下：
```
docker-ce.x86_64            3:20.10.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.7-3.el7                     docker-ce-stable
```

```

sudo yum install docker-ce-19.03.9
```
## Docker 命令
### 容器命令
* 删除所有容器： `docker rm -f $(docker ps -a -q)`
* 查看容器详细信息：`docker inspect <容器>`
* 删除所有停止的容器： `docker container prune`
* 删除所有停止的容器： `docker rm $(docker ps -a -q  --filter status=exited)`
* 拷贝文件宿主机拷贝至容器 `docker cp 文件路径 {dockerId}:目标路径`， 示例：`docker cp foo.txt mycontainer:/foo.txt`
* 容器拷贝纸宿主机：`docker cp  {dockerId}:目标路径 文件路径`，示例`docker cp mycontainer:/foo.txt foo.txt`
* 进入到启动容器中：`docker exec -it <容器ID>  /bin/bash`

## Docker 自动完成脚本

```
sudo curl https://raw.githubusercontent.com/docker/docker-ce/master/components/cli/contrib/completion/bash/docker -o /etc/bash_completion.d/docker.sh

source /etc/bash_completion.d/docker.sh

```
## 快速构建自己镜像
需求： 修改maven镜像，使用阿里云的mirror下载包
```
# 1、 装有docker的虚机上创建一个目录， 比如： maven
# 2、进入目录，添加Dockerfile 和 settings.xml; 内容下面会有
# 3、 构建镜像
docker build -t harbor.icoding.tech/public/maven:3-openjdk-8 .
# 4、 推送镜像, 需要先登录
docker login harbor.icoding.tech
docker push harbor.icoding.tech/public/maven:3-openjdk-8
```

##### Dockerfile 如下
```
FROM maven:3-openjdk-8
COPY settings.xml /usr/share/maven/conf/
```

##### settings.xml 如下
```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 https://maven.apache.org/xsd/settings-1.2.0.xsd">
  
  <pluginGroups></pluginGroups>
  <proxies></proxies>
  <servers></servers>

  <mirrors>
    <mirror>
        <id>alimaven</id>
        <mirrorOf>central</mirrorOf>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
  </mirrors>
  <profiles></profiles>
```


## dockerd 配置
/etc/docker/daemon.json
```
{
    "log-driver": "json-file",
    "log-opts": {
    "max-size": "100m"
    },
    "storage-driver": "overlay2",
    "registry-mirrors":[
        "https://kfwkfulq.mirror.aliyuncs.com",
        "https://2lqq34jg.mirror.aliyuncs.com",
        "https://pee6w651.mirror.aliyuncs.com",
        "http://hub-mirror.c.163.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com"
    ]
}
```
run `systemctl daemon-reload`

### 镜像命令
- 查询对应REPOSITORY和tag的镜像ID `docker images registry.cn-hangzhou.aliyuncs.com/sino-dc/apiproxy:2.4 -q`
- 查找并删除`docker rmi -f $(docker images registry.cn-hangzhou.aliyuncs.com/sino-dc/apiproxy:2.4 -q)`

#### 打包镜像 & 导入镜像
```
docker save docker.elastic.co/logstash/logstash:7.6.0 -o /root/logstash-7.6.0.tar  # 打包
# scp 上传到目标机器
docker load < logstash-7.6.0.tar #导入
```

### 查看镜像内容
docker run -it --entrypoint sh image_name

### 推送镜像至hub.docker.com
在网站https://hub.docker.com/ 注册个账号；创建对应的repo  。 
```
sudo docker login --username=choelea  # 使用注册的docker id登录；密码就是登陆否则登录hub.docker.com网站的密码
sudo docker build -t choelea/curator:5.8.1 .
sudo docker tag choelea/curator:5.8.1 choelea/curator:5.8.1
sudo docker push choelea/curator:5.8.1
```
> 镜像的namespace需要保持和dockerhub的用户名一致； 拉取的全路径为：docker.io/choelea/curator


### 推送镜像至阿里云镜像服务
去阿里云创建后按说明操作。

### Docker commit

参考： https://www.jianshu.com/p/27fc4630bed9

## 其他组件安装
### mysql启动：

#### 快速使用
```
docker run -p 3307:3306 --name mysql3307 \
-v /mysqldata/3307/logs:/var/log/mysql  \
-v /mysqldata/3307/db:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

#### 修改配置使用
下面配置采用 https://hub.docker.com/_/mysql     5.7.29 

**创建目录**
```
mkdir /home/mysqld
cd /home/mysqld
mkdir   data  logs  conf
```
**创建配置 `conf/mysqld.cnf`**
下面拷贝的5.7.29版本的默认的配置 `/etc/mysql/mysql.conf.d/mysqld.cnf`
```
# Copyright (c) 2014, 2016, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License, version 2.0,
# as published by the Free Software Foundation.
#
# This program is also distributed with certain software (including
# but not limited to OpenSSL) that is licensed under separate terms,
# as designated in a particular file or component or in included license
# documentation.  The authors of MySQL hereby grant you an additional
# permission to link the program and your derivative works with the
# separately licensed software that they have included with MySQL.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License, version 2.0, for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
datadir		= /var/lib/mysql
#log-error	= /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address	= 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# 这里添加你的个性化配置
```

**启动**
```
docker run -p 3307:3306 --name mysql3307 \
-v $PWD/conf/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf \
-v $PWD/logs:/var/log/mysql  \
-v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root#123 -d mysql:5.7.29
```

### mongodb启动

```
docker run  --name mongod -p 27017:27017  \
-v /data/mongodb/configdb:/data/configdb \
-v /data/mongodb/db:/data/db \
-d mongo:latest
```

## 清理Docker所占用的空间

### 查看Docker的磁盘使用情况：

`docker system df` 该命令列出了 docker 使用磁盘的 4 种类型 
```
Images: 所有镜像占用的空间，包括拉取的镜像、本地构建的镜像

Containers: 运行中的容器所占用的空间（没运行就不占空间），其实就是每个容器读写层的空间

Local Volumes: 本地数据卷的空间

Build Cache: 镜像构建过程中，产生的缓存数据
```

`docker system df -v` 命令可以进一步查看单个image镜像、container容器空间占用细节，以确定是哪个镜像、容器或本地卷占用过高空间


  

### 解决方案：
`docker system prune` 命令可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像（即无tag的镜像）

`docker system prune -a` 命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了……所以使用之前一定要想清楚


## Docker  坑集
容器不认宿主机的/etc/hosts 除非host 网路模式。 （在用docker-compose的时候想要简单修改下 /etc/hosts 来保证所有的数据库访问OK；遇见此问题）
## docker-compose安装
https://docs.docker.com/compose/install/

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```
> 可能出现网速太慢的情况，手动下载, 然后sftp传上去`put docker-compose-Linux-x86_64 /usr/local/bin/docker-compose`。 
> 
 
删除`sudo rm /usr/local/bin/docker-compose`


## docker-compose 开机启动
一般来说设置`restart: always`即可。
https://stackoverflow.com/questions/43671482/how-to-run-docker-compose-up-d-at-system-start-up
 
 新版：
 ```
 # 停止服务
sudo docker compose down -v

# 启动服务
sudo docker compose up -d
 ```
 
 
 ## 错误收集
 
 ### ENTRYPOINT中脚本的路径错误
 Spring Boot Docker 的官方文档中Dockerfile的配置
 ENTRYPOINT ["run.sh"]  需要改成 ["./run.sh"]，否则报错如下
 ```
 docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"run.sh\": executable file not found in $PATH": unknown.
ERRO[0005] error waiting for container: context canceled 
 ```


###  certificate signed by unknown authority
打开daemon.json
```
sudo vi /etc/docker/daemon.json
```
加入insecure-registries
```
{  
   "insecure-registries":["私库地址"]
}
```
重启docker

sudo systemctl restart docker