---
title:  基于minikube快读搭建kubernetes集群
...

在Centos7.5上基于minikube快读搭建kubernetes集群。

### 安装启动docker

安装步骤参考：https://www.runoob.com/docker/centos-docker-install.html
本次采用 `curl -sSL https://get.daocloud.io/docker | sh`

其他参考：
* 官方文档 https://kubernetes.io/docs/tutorials/hello-minikube/
* 实现集群外访问 https://blog.csdn.net/qq_45613785/article/details/129437313

启动：
```
systemctl start docker
systemctl enable docker.service
```

### 创建用户
操作 minikube 需要一个具有 root 权限的非root用户
```
adduser minikube
passwd minikukbe
usermod -aG wheel minikube

#添加到docker组
usermod -aG docker minikube

#更新docker组
newgrp docker
```


### 安装minikube
安装步骤参考：https://minikube.sigs.k8s.io/docs/start/

```
# 切换用户
su minikube
cd
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```

##### 启动集群
```
minikube start --image-mirror-country='cn' --kubernetes-version=v1.24.13
#image-mirror-country 为指定使用国内源
#kubernetes-version 指定部署的版本（最新版兼容性坑比较多，所以选择低版本
```

##### 验证状态

运行`minikube status`输出如下：
```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

### 安装 kubectl
由于 minikube 内置的 kubectl 命令功能不全，所以最好独立安装一个 kubectl

```
curl -LO "https://dl.k8s.io/release/v1.24.13/bin/linux/amd64/kubectl

## 安装
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

## 验证安装
kubectl version --client --output=yaml

## 命令输入tab 提示
source <(kubectl completion bash)
```

### 安装minikube dashboard
执行命令，启动dashboard插件 `minikube dashboard`。 
在无桌面的linux服务器上，由于无法打开浏览器，大致会收到如下的报错：
```
Error: no DISPLAY environment variable specified
xdg-open: no method available for opening 'http://127.0.0.1:39291/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/'

❌  Exiting due to HOST_BROWSER: failed to open browser: exit status 3
```

设置代理支持集群外部访问 `kubectl proxy --port=8001 --address='10.3.70.227' --accept-hosts='^.*'`
浏览器打开如下链接:

http://10.3.70.227:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/daemonset?namespace=default


### 尝试发布服务

```
kubectl expose deployment hello-node --port=8080 --target-port=8080 --type=NodePort
# port-forward 端口转发且允许任意ip访问
kubectl port-forward --address 0.0.0.0  service/hello-node 8080:8080
```