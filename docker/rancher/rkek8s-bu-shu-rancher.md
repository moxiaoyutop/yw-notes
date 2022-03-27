# rke-k8s部署rancher

### **首先所有节点初始化系统相关配置**

```
# 安装小工具
yum -y install epel-release
yum -y install lrzsz vim gcc glibc openssl openssl-devel net-tools wget curl

# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
 
# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时
 
# 关闭swap
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 时间同步
yum install ntp   #每台主机安装ntp服务
systemctl start ntpd    #启动时钟同步服务
systemctl enable  ntpd   #设置开机启动
ntpq -p   #查看时钟同步状态

 
# 根据规划设置主机名
#hostnamectl set-hostname <hostname>
 
# 修改 hosts 配置（可以只修改 master，或者所有节点）
cat >> /etc/hosts << EOF
172.16.205.134 fenzhi-master-1
172.16.205.135 fenzhi-node1
172.25.226.103 fenzhi-node2
EOF
 
# 内核参数调优
cat >> /etc/sysctl.conf << EOF
vm.swappiness=0
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.neigh.default.gc_thresh1=4096
net.ipv4.neigh.default.gc_thresh2=6144
net.ipv4.neigh.default.gc_thresh3=8192
EOF

# 使内核参数生效
modprobe br_netfilter  #首先执行这个命令后才不会报错
sysctl -p

# 加载ipvs相关模块
cat > /etc/sysconfig/modules/ipvs.modules << eof
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
eof

# 加载模块
chmod 755 /etc/sysconfig/modules/ipvs.modules 
bash /etc/sysconfig/modules/ipvs.modules
lsmod | grep -e ip_vs -e nf_conntrack_ipv4	  #查看是否已经正确安装lipset软件包

# 保证在节点重启后能自动加载所需模块
cat >> /etc/rc.d/rc.local << eof
bash /etc/sysconfig/modules/ipvs.modules
eof

chmod +x /etc/rc.d/rc.local 


# SSH Server配置
sed -i 's/#AllowTcpForwarding yes/AllowTcpForwarding yes/' /etc/ssh/sshd_config
systemctl restart sshd
```

### 所有节点安装docker

```
# 若是节点主机上已安装有docker，则先卸载及其依赖包
yum remove docker \
     docker-client \
     docker-client-latest \
     docker-common \
     docker-latest \
     docker-latest-logrotate \
     docker-logrotate \
     docker-engine

wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
 
yum install -y docker-ce docker-ce-cli containerd.io	#默认安装最新版本的docker
yum install -y bash-completion #安装docker命令补全工具

systemctl enable docker && systemctl start docker

# 配置 docker 国内镜像加速
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://khi28py7.mirror.aliyuncs.com"]
}
EOF

systemctl daemon-reload
systemctl restart docker
systemctl enable docker
```

### **为所有节点创建用户，并设置 ssh 免密登录**

```
useradd rke
usermod -aG docker rke
echo "123456" | passwd --stdin rke

# 制作免密
# 生成 ssh 秘钥
ssh-keygen -t rsa
 
# 配置免密登录
ssh-copy-id rke@172.16.205.134
ssh-copy-id rke@172.16.205.135
ssh-copy-id rke@172.25.226.103

# 脚本配置
user='rke'
password='123456'
for i in `seq 241 245`
do
  expect <<EOF
  spawn ssh-copy-id $user@192.168.0.$i
  expect {
    "yes/no" {send "yes\n";exp_continue}
    "password" {send "$password\n"}
  }
  expect eof
EOF
done
```

### 主节点安装rke

```
#安装rke二进制包
wget https://github.com/rancher/rke/releases/download/v1.3.7/rke_linux-amd64

# 如果当前登录用户非 root 帐号，切换到 root
su root

# 移动 rke 二进制包
mv rke_linux-amd64 /usr/bin/rke
 
# 文件授权
chmod +x /usr/bin/rke
```

### **主节点安装 Kubctl**

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/bin/
```

**所有节点开放端口KubeAPI：`6443` 和etcd：`2379`**

### **手动编写一个 cluster.yml 文件**

```
#切换到rancher用户
su - rke
cat > cluster.yml << EOF
# address 公网ip
# internal_address 内网ip
# 如果是内网情况部署，只需要配置address
nodes:
  - address: 165.227.114.63
    internal_address: 172.16.22.12
    user: ubuntu
    role: [controlplane, worker, etcd]
  - address: 165.227.116.167
    internal_address: 172.16.32.37
    user: ubuntu
    role: [controlplane, worker, etcd]
  - address: 165.227.127.226
    internal_address: 172.16.42.73
    user: ubuntu
    role: [controlplane, worker, etcd]

services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h

ingress:
  provider: nginx
  options:
    use-forwarded-headers: "true"
EOF
```

### 启动集群

```
#启动集群
rke up --config cluster.yml
# 查看集群
kubectl --kubeconfig kube_config_cluster.yml get nodes

# 也可以将 kube_config_cluster.yml 文件添加到系统变量中
# 这样 kubectl 就不需要在指定 --kubeconfig 文件了
echo export KUBECONFIG=/home/rke/kube_config_cluster.yml >> ~/.bash_profile
source ~/.bash_profile

# 第二种方式
# rek用户 
cp kube_config_cluster.yml ~/.kube/config
# root用户
cp /home/rancher/kube_config_cluster.yml ~/.kube/config

kubectl get nodes
kubectl get pods -A
```

## 如果部署失败按照一下步骤清理环境

### 清理环境

```
# 防火墙规则清理
/sbin/iptables -P INPUT ACCEPT
/sbin/iptables -F

# 容器清理
docker system prune -f
docker stop $(docker ps -aq)
docker rm -f $(docker ps -aq)
docker volume rm $(docker volume ls -q)
docker image rm $(docker image ls -q)
rm -rf /etc/ceph \
       /etc/cni \
       /etc/kubernetes \
       /opt/cni \
       /opt/rke \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/etcd \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/lib/rancher/rke/log \
       /var/log/containers \
       /var/log/pods \
       /var/run/calico

# 重启服务
systemctl restart docker
```

## 二、**部署 Rancher（可以使用root用户进行操作）**

```
# 下载 helm 二进制包
wget https://get.helm.sh/helm-v3.5.4-linux-amd64.tar.gz
 
# 解压
tar -zxvf helm-v3.5.4-linux-amd64.tar.gz
 
# 这一步需要 root 用户操作，否则可能会有权限不足的问题
mv linux-amd64/helm /usr/bin

# 为 Rancher 创建 Namespace
kubectl create namespace cattle-system
```

## 选择 SSL 选项（这里选用 Rancher 生成的 TLS 证书，因此需要 cert-manager）

### 使用 Rancher 生成的 TLS 证书

```
#选择 SSL 选项（这里选用 Rancher 生成的 TLS 证书，因此需要 cert-manager）
# 如果你手动安装了CRD，而不是在Helm安装命令中添加了`--set installCRDs=true`选项，你应该在升级Helm chart之前升级CRD资源。
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.1/cert-manager.crds.yaml
 
# 添加 Jetstack Helm 仓库
helm repo add jetstack https://charts.jetstack.io
 
# 更新本地 Helm chart 仓库缓存
helm repo update
 
# 安装 cert-manager Helm chart
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.1

# 添加rancher存储库
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

#安装完 cert-manager 后，你可以通过检查 cert-manager 
kubectl get pods --namespace cert-manager
```

### 使用私有 CA 签发证书

> 如果您使用的是私有 CA，Rancher 需要您提供 CA 证书的副本，用来校验 Rancher Agent 与 Server 的连接。
>
> 拷贝 CA 证书到名为 `cacerts.pem` 的文件，使用 `kubectl` 命令在 `cattle-system` 命名空间中创建名为 `tls-ca` 的密文。

> 如果是生产环境。就把域名的证书转换成pem格式

> [https://docs.rancher.cn/docs/rancher2/installation/install-rancher-on-k8s/\_index/](https://docs.rancher.cn/docs/rancher2/installation/install-rancher-on-k8s/\_index/)
>
> 中文文档 关于ssl

> 测试环境自签证书脚本 官方提供
>
> [https://docs.rancher.cn/docs/rancher2/installation/resources/advanced/self-signed-ssl/\_index/](https://docs.rancher.cn/docs/rancher2/installation/resources/advanced/self-signed-ssl/\_index/)

```
# 将证书添加
kubectl -n cattle-system create secret generic tls-ca --from-file=cacerts.pem=./cacerts.pem

# 添加rancher存储库
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

#安装完 cert-manager 后，你可以通过检查 cert-manager 
kubectl get pods --namespace cert-manager
```

### 通过 helm 安装 Rancher

```
# 这是rancher自动生成ca
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set bootstrapPassword=admin

# 这是自签证书
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=it3.rancher.com \
  --set ingress.tls.source=secret \
  --set privateCA=true
```

* namespace：命名空间
* hostname：负载均衡器的 DNS 记录，你需要通过这个域名来访问 Rancher Server。

等待 Rancher 运行：

```
kubectl -n cattle-system rollout status deploy/rancher
 
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out

# 查看 Rancher 运行状态
kubectl -n cattle-system get deploy rancher
```

至此，Rancher 部署也就完成了！



接下来把配置的域名 host 解析到本地

vim /etc/hosts

120.xx.xxx.xx rancher.xxx.com



本地电脑也要修改一下 hosts 文件，linux 系统同上操作，windows 系统，hosts 文件路径为 C:\Windows\System32\drivers\etc\hosts，添加同上的操作。

2.配置完成后，打开浏览器访问：rancher.xxx.com，由于第一次访问，需要重新设置密码&#x20;
