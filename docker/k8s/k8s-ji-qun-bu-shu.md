# k8s集群部署

### 部署规划

> 10.211.55.18 k8s-node1&#x20;
>
> 10.211.55.19 k8s-node2&#x20;
>
> 10.211.55.20 k8s-node3

### 关闭防火墙

```
systemctl stop firewalld
setenforce 0
```

### 关闭swap

```
swapoff -a    临时关闭
free             可以通过这个命令查看swap是否关闭了
vim /etc/fstab  永久关闭
```

### 添加主机名与ip对应的关系

```
vim /etc/hosts
 添加如下内容：
10.211.55.18     k8s-node1
10.211.55.19     k8s-node2
10.211.55.20     k8s-node3
```

### 修改内核参数

```
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-iptables  = 1
EOF

sysctl --system
```

### 安装docker

```
1）下载并安装
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O/etc/yum.repos.d/docker-ce.repo
$ yum install docker-ce-20.10.2

2）配置镜像加速
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["http://hub-mirror.c.163.com", "https://docker.mirrors.ustc.edu.cn"]
}
EOF

3）设置开机启动
$ systemctl enable docker
$ systemctl start docker

4）查看Docker版本
$ docker --version
```

### 添加阿里云yum软件源

```
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 安装kubeadm,kubelet and kubectl

```
yum install -y kubelet-1.20.1 kubectl-1.20.1 kubeadm-1.20.1 kubernetes-cni-1.20.1 --nogpgcheck

# 开机启动
systemctl enable kubelet
```

### 部署kubernetes master

### 修改 `kubelet.service`

`/etc/systemd/system/kubelet.service.d/10-proxy-ipvs.conf` 写入以下内容

```
# 启用 ipvs 相关内核模块
[Service]
ExecStartPre=-/sbin/modprobe ip_vs
ExecStartPre=-/sbin/modprobe ip_vs_rr
ExecStartPre=-/sbin/modprobe ip_vs_wrr
ExecStartPre=-/sbin/modprobe ip_vs_sh
```

执行以下命令应用配置。

```
systemctl daemon-reload
```

初始化kubeadm

```
kubeadm init \
--apiserver-advertise-address=10.211.55.18 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.20.1 \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16
```



> 遇到的报错
>
> \#报错信息 The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp \[::1]:10248: connect: connection refused. #之前我的Docker是用yum安装的，docker的cgroup驱动程序默认设置为systemd。默认情况下Kubernetes cgroup为system，我们需要更改Docker cgroup驱动

```
# 添加以下内容
vim /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

# 重启docker
systemctl restart docker
# 重新初始化
kubeadm reset # 先重置
```

### 顺利部署好了master

![](<../../.gitbook/assets/image (11).png>)

### 使用kubectl工具

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get node
```

### 安装Pod网络插件

```
# 安装插件
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 查看是否部署成功
$ kubectl get pods -n kube-system

# 再次查看node，可以看到状态为ready
$ kubectl get node

```

> \#初始化集群coredns容器一直处于pending状态
>
> \#缺少网络插件，部署flannel网络插件

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

#已成功下载，正在init：0/1拉取镜像
kubectl describe pod -n kube-system kube-flannel-ds-amd64-c4h6p

#如果有报错拉镜像拉不下来，就手动去拉去 
docker pull images
```
