---
title:  在Centos 7.5 上安装 Harbor
showOnHome: false
...

Harbor 是一个开源的企业级注册服务器，用于存储和分发 Docker 镜像。Harbor 通过添加企业通常需要的功能（例如安全性、身份和管理）来扩展开源 Docker Distribution。作为企业私有注册中心，Harbor 提供了更好的性能和安全性。让注册表更接近构建和运行环境可以提高镜像传输效率。Harbor 支持设置多个注册中心，并在它们之间复制镜像。此外，Harbor 还提供高级安全功能，例如用户管理、访问控制和活动审计。

### 准备条件

准备一台虚机，本次采用Centos 7.5系统的虚机， 创建具有sudo权限的非 root 用户。


### 安装 Docker 和 Docker-Compose
Harbor 部署为多个 Docker 容器。因此，它可以部署在任何支持 Docker 的 Linux 发行版上。目标主机需要安装 Docker 和 Docker Compose。
参考 ： http://tech.jiu-shu.com/Dev-Ops/docker-compose


### 安装 Harbor


#### 下载在线安装包
可以从发布页面下载安装程序的二进制文件, 选择在线或离线安装程序。参考链接： https://goharbor.io/docs/2.8.0/install-config/download-installer/  本次示例采用在线安装包。
```
cd /opt
# 在线安装程序：
sudo wget https://github.com/goharbor/harbor/releases/download/v2.7.2/harbor-online-installer-v2.7.2.tgz

# 使用tar命令解压包。
tar -xvf harbor-online-installer-v2.7.2.tgz
```

#### 准备SSL证书
本次采用**受信任的第三方 CA**证书，假定域名是harbor.example.com，如果是需要自生成证书，可以参考：[Harbor Configure Https](https://goharbor.io/docs/2.8.0/install-config/configure-https/)

将证书复制到宿主机目录： ` /etc/docker/certs.d/` 以供后面配置

#### 配置
进入解压后harbor文件夹, 拷贝 `harbor.yml.tmpl` 保存为 `harbor.yml`。 修改配置`harbor.yml`： 
```
hostname: harbor.example.com
https:
  certificate: /etc/docker/certs.d/harbor.example.com.crt
  private_key: /etc/docker/certs.d/harbor.example.com.key

## harbor admin 的初始密码，第一次运行install生效，后期需要从UI修改。 应该是保存到数据库了
harbor_admin_password: *********
```
运行`sudo install.sh`以完成安装。

#### 停止和重启

运行install.sh安装后， 如果需要停启harbor， 需要通过docker compose来操作。 

```
# 停止服务
sudo docker compose down -v

# 启动服务
sudo docker compose up -d
```
