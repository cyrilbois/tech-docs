---
title:  在Centos 7.5 上安装 Harbor
showOnHome: false
...

Harbor 是一个开源的企业级注册服务器，用于存储和分发 Docker 镜像。Harbor 通过添加企业通常需要的功能（例如安全性、身份和管理）来扩展开源 Docker Distribution。作为企业私有注册中心，Harbor 提供了更好的性能和安全性。让注册表更接近构建和运行环境可以提高镜像传输效率。Harbor 支持设置多个注册中心，并在它们之间复制镜像。此外，Harbor 还提供高级安全功能，例如用户管理、访问控制和活动审计。

### 准备条件

准备一台虚机，本次采用Centos 7.5系统的虚机， 创建具有sudo权限的非 root 用户。

### 安装EPEL仓库
在开始之前，将EPEL repo 和其他所需的软件包安装到系统中。
```
sudo yum install epel-release wget -y

#安装完成后，将系统更新到最新版本。
sudo yum update -y

#接下来，保险起见，重新启动系统以应用所有更新。
sudo reboot
```

### 更新python

从源代码安装
```
# 安装依赖包
sudo yum install gcc openssl-devel bzip2-devel libffi-devel

# 下载源代码 & 解压
wget https://www.python.org/ftp/python/3.9.5/Python-3.9.5.tgz
tar -xf Python-3.9.5.tgz

# 生成make 文件 
cd Python-3.9.5
./configure prefix=/usr/local/python3 #这是指定安装python3目录

# 编译安装
make && make install
# make install 之后提示ok 就没问题 要是报错就要找对应错误 先通过yum安装依赖包 然后重新编译安装  

# 将原来的链接备份
mv /usr/bin/python /usr/bin/python.bak
 
# 添加python3的软链接  如果要保留python2的软连接 那就设设置 python3  使用时候就用 python3执行就好
ln -s /usr/local/python3/bin/python3.9 /usr/bin/python #/usr/bin/python3
 
# 测试是否安装成功了
python -V #python3 -V
```

### 安装 Docker 和 Docker-Compose
Harbor 部署为多个 Docker 容器。因此，它可以部署在任何支持 Docker 的 Linux 发行版上。目标主机需要安装 Docker 和 Docker Compose。
```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce


pip install docker-compose

# 启动docker
sudo systemctl start docker

#通过运行映像验证 docker 是否已正确安装hello-world。
sudo docker run hello-world
```

### 安装 Harbor
可以从发布页面下载安装程序的二进制文件。选择在线或离线安装程序。
```
# 在线安装程序：
wget https://github.com/vmware/harbor/releases/download/v1.2.0/harbor-online-installer-v1.2.0.tgz

# 离线安装程序：
wget https://github.com/vmware/harbor/releases/download/v1.2.0/harbor-offline-installer-v1.2.0.tgz

# 使用tar命令解压包。
tar -xvf harbor-online-installer-1.2.0.tgz
```
生成您自己的 SSL 证书
Harbor 的默认安装使用HTTP- 因此，您需要将选项添加--insecure-registry到客户端的 Docker 守护进程，然后重新启动 Docker 服务。HTTPS强烈建议安装 Harbor 。将来会为我们节省很多时间。生成您自己的 SSL 证书（替换harbor.example.com为您主机的 FQDN）。

mkdir cert && cd cert

openssl req -sha256 -x509 -days 365 -nodes -newkey rsa:4096 -keyout  harbor.example.com.key -out harbor.example.com.crt
配置港口
编辑 Harbor 配置文件。

vim harbor.cfg
更改hostname为主机的 FQDN 并启用https.

hostname = harbor.example.com

ui_url_protocol = https

ssl_cert = /root/cert/harbor.example.com.crt

ssl_cert_key = /root/cert/harbor.example.com.key
运行install.sh以完成安装。

./install.sh
在后台运行 Harbor。

docker-compose up -d
访问 Harbor Web 界面
在开始之前，您需要允许端口80通过防火墙。

sudo firewall-cmd --permanent --zone=public --add-port=80/tcp

sudo firewall-cmd --reload
http://harobr.example.com如果配置了 DNS，则可以访问 Harbor 服务器。使用默认username和登录password。

admin

Harbor12345