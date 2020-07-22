---
title:  阿里云 ECS 安装 kubernetes 演练
...

## 规划



| Master / Node | Hostname | Ip |
| -------- | -------- | -------- |
|   master   | k8smaster.tech    | 192.168.0.151     |
|   node   | k8s002    | 192.168.0.152     |
|   node   | k8s003    | 192.168.0.153    |


```
172.26.249.118 k8smaster.tech
172.26.249.119 k8snode1.tech	
172.26.249.120 k8snode2.tech
```

修改hostname：
```
hostnamectl set-hostname k8smaster.tech
hostnamectl set-hostname k8snode1.tech
hostnamectl set-hostname k8snode2.tech
```
## 安装基础软件
### 配置机器 （所有机器）

### 安装基础软件Docker （所有机器）
```
yum -y install yum-utils lvm2 device-mapper-persistent-data nfs-utils xfsprogs wget
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum -y install docker-ce docker-ce-cli containerd.io
```

### 修改docker引擎的配置
修改Docker Cgroup Driver为systemd, 并设置响应的镜像。
```
mkdir /etc/docker
touch /etc/docker/daemon.json
cat > /etc/docker/daemon.json <<EOF
{
    "exec-opts": ["native.cgroupdriver=systemd"],
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
EOF
```

### 重启Docker 
```
systemctl daemon-reload
systemctl enable  --now docker
```
### Kuberntes Repo添加 （所有机器）
添加编辑`/etc/yum.repos.d/kubernetes.repo`, 国内网络问题，这里使用阿里云的镜像。
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```




### Master 安装 （仅master机器）
```
yum install -y kubelet kubeadm kubectl 
systemctl enable --now kubelet
```
> 这个时候kubelet一直启动报错， 不停重启。可以使用`journalctl -u kubelet`查看日志。  参考：https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/ The kubelet is now restarting every few seconds, as it waits in a crashloop for kubeadm to tell it what to do. 


### Master 配置
```
kubeadm init \
        --apiserver-advertise-address 0.0.0.0 \
        --apiserver-bind-port 6443 \
        --cert-dir /etc/kubernetes/pki \
        --control-plane-endpoint k8smaster.tech \
        --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
        --kubernetes-version 1.18.2 \
        --pod-network-cidr 10.10.0.0/16 \
        --service-cidr 10.20.0.0/16 \
        --service-dns-domain cluster.local \
        --upload-certs
```
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

集群必须安装网络插件以实现Pod间通信，只需要在Master节点操作，其他Node节点会自动创建相关Pod；

该配置文件默认采用的Pod的IP地址为192.168.0.0/16，需要修改为集群初始化参数中采用的值，本例中为10.10.0.0/16；
```
wget https://docs.projectcalico.org/v3.8/manifests/calico.yaml
sed -i "s#192\.168\.0\.0/16#10\.10\.0\.0/16#" calico.yaml
kubectl apply -f calico.yaml
```
等待所有容器状态处于Running状态：可以通过命令`watch -n 2 kubectl get pods -n kube-system -o wide`来实时观察。通过命令`kubectl get nodes -o wide` 查看所有node状态。

2) 获取join命令参数，并保存输出结果：
#kubeadm token create --print-join-command > node.join.sh




```
kubeadm join k8smaster.tech:6443 --token iga2tp.t4fdinzvadminlqu \
    --discovery-token-ca-cert-hash sha256:4cf53dbc3b32901972487b3474c6760b385b780d2d73bbaef24e5bcb50966732 \
    --control-plane --certificate-key d12a595435127d11f8f3e051c75c8ef768b29eff88aa1a47c702e9cebd929b2b
```

