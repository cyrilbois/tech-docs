---
title:  ELK docker-compose 监控及日志收集
description: 采用docker 部署elk， 配置使用ELK来收集日志，做监控。 
...

## 环境及版本

| 环境/服务 | 版本 | 备注|
| -------- | -------- |-------- |
| Centos     | CentOS Linux release 7.5.1804 (Core)     |  宿主机  |
| ELK          |  7.6.0 | docker-compose 方式运行，这里假定宿主机IP地址为：10.3.69.41 |
| nginx        |   1.61.1  | 运行在docker-compose 中的一个服务|

## 安装ELK运行
- 安装docker
- 安装docker-compose
- 克隆 `https://github.com/choelea/docker-elk`
- 进入docker-elk运行`docker-compose up`

> 下载镜像会比较耗时，如果下载失败可以尝试手动`docker pull elasticsearch:7.6.0`, 同样方法下载logstash和kibana。

运行起来后就可以通过：http://10.3.69.41:5601   (账号: elastic/joe2020)访问kibana。 通过kibana的帮助文档来安装配置，进入http://10.3.69.41:5601/app/kibana#/home 找到对应项目按说明操作。
## Nginx Metrics
参考http://10.3.69.41:5601/app/kibana#/home/tutorial/nginxMetrics 来安装配置metricbeat。 

需要注意的是metricbeat的nginx模块的配置:`/etc/metricbeat/modules.d/nginx.yml`, 其默认配置是读取127.0.0.1/server-status来获取nginx状态的。 本实例的版本需要手动配置nginx保证此路径的正常访问。
```
location /server-status {
            stub_status on;
}
```
有用的参考： 
- https://www.nginx.com/blog/monitoring-nginx/
- http://nginx.org/en/docs/http/ngx_http_stub_status_module.html
- https://www.tecmint.com/enable-nginx-status-page/ （限制访问IP方案， 但是docker部署的nginx不适用）

## Nginx 日志收集
参考http://10.3.69.41:5601/app/kibana#/home/tutorial/nginxLogs 设置即可。

示例配置/etc/filebeat/modules.d/nginx.yml如下：
```
# Module: nginx
# Docs: https://www.elastic.co/guide/en/beats/filebeat/7.6/filebeat-module-nginx.html

- module: nginx
  # Access logs
  access:
    input: 
      exclude_lines: ['.GET /server-status.'] # 排除/server-status
    enabled: true
    var.paths: ["/home/nantong-box/AllApps/logs/nginx/access.log"]
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:

  # Error logs
  error:
    enabled: true
    var.paths: ["/home/nantong-box/AllApps/logs/nginx/error.log"]
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:
```


> filebeat 配置错了，不太好查，需要通过： `filebeat -e -c /etc/filebeat/filebeat.yml` 手动启动来验证。

## 应用日志配置
前面已经安装了filebeat，filebeat自带了很多模块， 比如nginx mysql等， 但是如果需要收集自己的应用服务的日志则需要手动配置。 

### nodejs winston日志配置示例
这个示例中修改了默认的index， 需要手动去http://10.3.69.41:5601/app/kibana#/management/kibana/index_patterns 创建 index pattern。
```
- type: log
  paths:
    - /home/nantong-box/AllApps/logs/apiproxy/app-*.log
  index: "bizlog-apiproxy-%{+yyyy.MM.dd}" # 不用filebeat的默认index， 使用自定义index
  tags: ["apiproxy"]
	#fields:
  #  appId: "apiproxy"  # 也可以使用此方式添加字段
```
对应的应用参考：https://github.com/choelea/tokenbased-api-gateway。 
winston 的日志格式： 
```
format: combine(
        timestamp(),
        logstash()
    ),
```
