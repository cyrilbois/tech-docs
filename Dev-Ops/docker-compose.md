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
* 删除所有停止的容器： `docker rm $(docker ps -a -q  --filter status=exited)`

### 镜像命令
- 查询对应REPOSITORY和tag的镜像ID `docker images registry.cn-hangzhou.aliyuncs.com/sino-dc/apiproxy:2.4 -q`
- 查找并删除`docker rmi -f $(docker images registry.cn-hangzhou.aliyuncs.com/sino-dc/apiproxy:2.4 -q)`


## 其他组件安装
### mysql启动：
```
docker run -p 3307:3306 --name mysql3307 \
-v /mysqldata/3307/logs:/var/log/mysql  \
-v /mysqldata/3307/db:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
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
 


