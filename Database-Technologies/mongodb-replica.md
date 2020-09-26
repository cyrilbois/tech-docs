---
title:  mongodb 复制集
...
资源有限，分别用了两台虚机验证复制集，一台虚机启了连个mongod实例，其中一个实例为`arbiterOnly:true,`只作为选举节点。
## 官方文档
https://docs.mongodb.com/manual/replication/

## 单机安装

**安装**
```
cd /opt
sudo curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.2.1.tgz
sudo tar -xvf mongodb-linux-x86_64-rhel70-4.2.1.tgz
sudo mv mongodb-linux-x86_64-rhel70-4.2.1 mongodb
```

**设置环境变量**
编辑`/etc/profile`,在最后添加
```
export PATH=$PATH:/opt/mongodb/bin
```
使环境变量生效 `source /etc/profile`。 （这里也可以不配置环境变量， 后面sudo命令会使用全路径）

**创建文件目录**

创建文件目录`mkdir -p /data/mongodb/{data,log}`; 创建配置文件mongod.conf,内容如下：
```
systemLog:
  destination: file
  path: /data/mongodb/log/mongod.log # log path
  logAppend: true
storage:
  dbPath: /data/mongodb/data # data directory
net:
  bindIp: 0.0.0.0
  port: 27017 # port
processManagement:
  fork: true # daemon process
```
启动：`sudo /opt/mongodb/bin/mongod -f /data/mongodb/mongod.conf`
## 复制集架构图

![mongo-rs-arbiter.png](http://tech.jiu-shu.com/Database-Technologies/mongo-rs-arbiter.png)
## 安装mongo
下载 解压 设置环境变量 启动
```
mkdir -p /data /data/db
cd /data
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.2.1.tgz
tar -xvf mongodb-linux-x86_64-rhel70-4.2.1.tgz
export PATH=$PATH:/data/mongodb-linux-x86_64-rhel70-4.2.1/bin
```
添加配置文件`/data/mongod.conf`:
```
systemLog:
  destination: file
  path: /data/db/mongod.log # log path
  logAppend: true
storage:
  dbPath: /data/db # data directory
net:
  bindIp: 0.0.0.0
  port: 27017 # port
replication:
  replSetName: rs0
processManagement:
  fork: true
```
启动`mongod -f /data/mongod.conf `


## 配置复制集
登录想要设置为主数据库的mongo，这里是10.0.6.72这台机器。
```
$mongo
>rs.initiate({
    _id: "rs0",
    members: [
        {
                        _id: 0,
                        host: "172.26.206.237:27017" 
            },
            {
                        _id: 1,
                        host: "172.26.206.238:27017" 
            },
            {
                    _id: 3,
										arbiterOnly:true,
                    host: "172.26.206.238:27018"
            }
        ]
})
```
登录从数据库机器：
```
$mongo
rs.slaveOk();
```
这些来从数据库就可以用来读取数据了。
> 注意需要确定端口都开放了。

## 可能会遇到的问题及解决办法

##### Failed global initialization: BadValue Invalid or no user locale set. Please ensure LANG and/or LC_* environment variables are set correctly.


```
# 编辑，vim /etc/profile 添加 export LC_ALL=C
# vim /etc/profile 
# export LC_ALL=C
source /etc/profile
```





