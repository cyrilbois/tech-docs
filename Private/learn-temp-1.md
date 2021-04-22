---
title:  Sentinel
description: Sentinel 提供一个轻量级的开源控制台，它提供机器发现以及健康情况管理、监控（单机和集群），规则管理和推送的功能
showOnHome: false
...

Sentinel 提供一个轻量级的开源控制台，它提供机器发现以及健康情况管理、监控（单机和集群），规则管理和推送的功能

### 启动控制台
参考考文档[alibaba Sentinel 控制台](https://github.com/alibaba/Sentinel/wiki/%E6%8E%A7%E5%88%B6%E5%8F%B0) 下载jar包直接运行
```
java -Dserver.port=8888 -Dcsp.sentinel.dashboard.server=localhost:8888 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar    // 这里只是改了下端口号
```

##### 容器镜像制作
1. 新建文件夹
2. 下载sentinel-dashboard.jar
3. 创建Dockerfile
4. 构建镜像： `docker build -t sentinel-dashboard:1.8.1 .`
5. 根据阿里云镜像仓库说明推送至阿里云

### 客户端接入
参考考文档[alibaba Sentinel 控制台](https://github.com/alibaba/Sentinel/wiki/%E6%8E%A7%E5%88%B6%E5%8F%B0) 

### 测试结果





