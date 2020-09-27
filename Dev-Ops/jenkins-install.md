---
title:  Jenkins 安装
...


## 安装jenkins
参考： https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/#setup-wizard

运行如下命令即可：
```
docker network create jenkins
docker volume create jenkins-docker-certs
docker volume create jenkins-data

docker container run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --volume "$HOME":/home docker:dind


docker container run --name jenkins-tutorial --rm --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  --volume "$HOME":/home \
  --publish 8080:8080 jenkinsci/blueocean
```

## 访问
http://{ip}:8080 访问， 初始密码通过`docker logs -f jenkins-tutorial` 查看密码, 可以通过如下的log找到密码`32568150b93148ce9ac741163c519ea4`。
```
*************************************************************
*************************************************************
*************************************************************
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:
32568150b93148ce9ac741163c519ea4
This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
*************************************************************
*************************************************************
*************************************************************
```

## 最该更新中心的URL
受限于国内网络问题，插件列表，包括插件安装都有些问题，很多插件下载不了，甚至插件列表都无法下载。 我们先解决插件列表的问题：

首选需要找到配置`hudson.model.UpdateCenter.xml`的位置。 通过`docker volume inspect jenkins-data` 查看对应宿主机上的路径, 这里查看到路径如下：`/var/lib/docker/volumes/jenkins-data/_data`
```
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/jenkins-data/_data",
        "Name": "jenkins-data",
        "Options": {},
        "Scope": "local"
    }
]
```
找到配置文件路径后修改配置：  https://updates.jenkins.io/update-center.json 修改为：http://mirror.xmission.com/jenkins/updates/update-center.json。

> 很多时候更新update center也未必好使，这个时候可以手动安装插件：http://updates.jenkins-ci.org/download/plugins/ 找到对应的插件下载后手动安装。

安装如下插件
 - http://updates.jenkins-ci.org/download/plugins/http_request/
 - http://updates.jenkins-ci.org/download/plugins/generic-webhook-trigger/

## 修改时区
Script Console运行如下命令：
```
System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Shanghai')
```

## 创建任务

1. 工程编写好Jenkinsfile
2. 进入jenkins创建一个新的任务，选择”流水线“类型; 流水线脚本从SCM中获取
![jenkins-pipeline-define.png](http://tech.jiu-shu.com/Dev-Ops/jenkins-pipeline-define.png)


## 问题收集

#### 碰见git拉取代码ssl的问题可以如下处理
```
docker exec -it jenkins-tutorial sh   # 进去容器
git config --global http.sslVerify false  # 禁用掉ssl检查
```