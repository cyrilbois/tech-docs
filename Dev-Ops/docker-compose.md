---
title:  docker
...
## Docker 安装
运行命令： `curl -fsSL get.docker.com -o get-docker.sh && sudo sh get-docker.sh --mirror Aliyun`
如果命令报错，可以手动访问https://get.docker.com/ 下载脚本， sftp上去进行安装。

启动并设置开机启动
```
sudo systemctl enable docker
sudo systemctl start docker
```
## Docker 命令
### 容器命令
* 删除所有容器： `docker rm -f $(docker ps -a -q)`
* 删除所有停止的容器： `docker container prune`
* 删除所有停止的容器： `docker rm $(docker ps -a -q  --filter status=exited)`
* 拷贝文件宿主机拷贝至容器 `docker cp 文件路径 {dockerId}:目标路径`， 示例：`docker cp foo.txt mycontainer:/foo.txt`
* 容器拷贝纸宿主机：`docker cp  {dockerId}:目标路径 文件路径`，示例`docker cp mycontainer:/foo.txt foo.txt`

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
sudo docker login  # 使用注册的docker id登录；注意需要sudo；否则登录不成功
sudo docker build -t jiu-shu/curator:5.8.1 .
sudo docker tag jiu-shu/curator:5.8.1 choelea/curator:5.8.1
sudo docker push choelea/curator:5.8.1
```


### 推送镜像至阿里云镜像服务
去阿里云创建后按说明操作。

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
 
 
 
 ## 错误收集
 
 ### ENTRYPOINT中脚本的路径错误
 Spring Boot Docker 的官方文档中Dockerfile的配置
 ENTRYPOINT ["run.sh"]  需要改成 ["./run.sh"]，否则报错如下
 ```
 docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"run.sh\": executable file not found in $PATH": unknown.
ERRO[0005] error waiting for container: context canceled 
 ```


