---
title:  Kong API Gateway 插件开发 - 推送请求日志至Elasticsearch
...

前面有有一篇 [Kong Apigateway 快速开始 Docker+DB版](http://tech.jiu-shu.com/Micro-Services/kong-guickstart) 采用了docker 的方式快速探索了Kong Apigateway的DB的模式。在有一定了解，并对kong的plugin做了尝试之后，这里继续探索插件的开发。
## 背景
需求需要记录API请求的信息，包括请求Body和响应的Body，并推送至elasticsearch， 使用kibana来查询。 （旧的说法是ELK）
## Kong的架构
Kong被部署在NGINX和Apache Cassandra或PostgreSQL等可靠技术之上，并为您提供了易于使用的RESTful API来操作和配置系统。
![kong-architecture.png](http://tech.jiu-shu.com/Micro-Services/kong-architecture.png)

## Kong Apigateway Overview
![Kong-GS-overview.png](http://tech.jiu-shu.com/Micro-Services/Kong-GS-overview.png)
## 安装
非Docker的方式安装

**创建repo  **

创建`/etc/yum.repos.d/bintray-kong-kong-rpm.repo` 使用如下内容：
```
#bintray--kong-kong-rpm - packages by  from Bintray
[bintray--kong-kong-rpm]
name=bintray--kong-kong-rpm
baseurl=https://kong.bintray.com/kong-rpm/centos/7
gpgcheck=0
repo_gpgcheck=0
enabled=1
```
**安装** 命令`yum install kong`完成安装。 

 ## 配置（声明式）
 Kong可以在有或没有数据库的情况下运行。**使用数据库**时，将使用kong.conf配置文件在启动时设置Kong的配置属性，并将数据库存储为所有已配置实体（例如Kong所代理的路线和服务）的存储。**不使用数据库时**，将使用kong.conf的配置属性和kong.yml文件将实体指定为声明性配置。


#### 生成并配置kong.yml

Kong 的其他的配置，比如service routes 等需要单独的配置， 这里采用声明式的方式来配置。 
```
mkdir /root/kong
cd /root/kong
kong config init  # 这里会生成一个kong.yml文件
```
kong.yml的配置信息参考如下：
```
_format_version: "1.1"
services:
- name: demo-service
  url: http://10.3.69.45:8000
  routes:
  - name: demoapi
    paths:
    - /
routes:
- name: demoapi
  service: demo-service
  protocols: [http]
  paths: ['/']

plugins:
#- name: tcp-log-body  插件开发后替换使用
- name: tcp-log
  config:
    host: 10.3.69.41
    port: 5000
    #index_name: bayuquanapp-api-log
```
#### 配置kong.conf
kong.conf 是个必备的配置,  默认位置和大部分软件一样在/etc/kong/kong.conf。 安装完成后这个目录下面会有个默认带有很多注解的kong.conf.default文件作为参考。 需要手动添加kong.conf, 配置示例如下：
```
proxy_listen = 0.0.0.0:9000 reuseport backlog=16384, 0.0.0.0:9443 http2 ssl reuseport backlog=16384   # 一般不用配置，默认端口个8000， 这里因为端口冲突才配置
admin_listen = 127.0.0.1:9001 reuseport backlog=16384, 127.0.0.1:8444 http2 ssl reuseport backlog=16384  # 一般不用配置，默认端口个8001， 这里因为端口冲突才配置
# plugins = bundled,tcp-log-body   #配置指定启动plugins
database = off   # 设定使用声明式而非数据库
declarative_config = /root/kong/kong.yml # 指定kong.yml 
```

验证配置是否正确：  `kong config -c /etc/kong/kong.conf parse kong.yml`

#### 启动 Kong
```
kong start # kong.conf 在默认位置
kong start -c /path/to/kong.conf  #指定 配置文件路径
```


## 插件开发及安装
找个目录克隆插件代码：`git clone https://github.com/choelea/kong-plugin-tcp-log-body`

进入目录运行命令打包安装
```
luarocks make
luarocks pack kong-plugin-tcp-log-body 0.1.0
luarocks install kong-plugin-tcp-log-body-0.1.0-1.all.rock
```

**修改配置使用插件**
1. 修改kong.conf 中的`plugins = bundled,tcp-log-body`
2. 修改kong.yml （参考上面kong.yml  的注释）
3. 重启`kong restart`

## 参考

https://openresty-reference.readthedocs.io/en/latest/Lua_Nginx_API/
