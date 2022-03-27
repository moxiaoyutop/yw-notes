# 基本操作

### docker安装

```
1）下载并安装
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

$ sudo sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

# 官方源
# yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo

# 安装docker
yum install docker-ce docker-ce-cli containerd.io -y

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

### 添加内核

如果在 CentOS 使用 Docker 看到下面的这些警告信息：

```bash
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```

请添加内核配置参数以启用这些功能。

```bash
tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

然后重新加载 `sysctl.conf` 即可

```bash
sysctl -p
```

### 基础命令

```
# 拉去镜像
docker pull centos:7

# 运行容器
docker run -d -p 8080:8080 tomcat:9.0

# 列出镜像
docker image ls

# 查看镜像、容器、数据卷占用的空间
docker system df

# 修改后的容器保存镜像 // 慎用，镜像过于臃肿
docker commit \
    --author "Tao Wang <twang2218@gmail.com>" \
    --message "修改了默认网页" \
    webserver \
    nginx:v2

# 查看镜像内的历史记录
docker history nginx:v2

# 获取容器ip
docker inspect --format '{{ .NetworkSettings.IPAddress }}'

# 获取所有容器ip
docker inspect -f '{{.Name}} - {{.NetworkSettings.IPAddress }}' $(docker ps -aq)

# 进入容器
docker exec -it 69d1 /bin/bash

# 查看端口
docker port 21c86ff1dbff

# 镜像打包
# 镜像打包
docker save -o /root/xxx.tar  <name>

# 导入镜像
docker load -i /root/xxx.tar

# 容器打包
# 容器打包
docker export <name> > /root/xx.tar

# 导入容器
docker import xx.tar <name>:latest

# 查看所有数据卷
docker volume ls

# 查看指定数据卷信息
docker volume inspect my-vol

# 启动一个挂载数据卷容器
# 并加载一个 数据卷 到容器的 /usr/share/nginx/html 目录。
# 参数--mount默认情况下用来挂载volume，但也可以用来创建bind mount和tmpfs。
# 如果不指定type选项，则默认为挂载volume，volume是一种更为灵活的数据管理方式，
# volume可以通过docker volume命令集被管理。
docker run -d -P \
    --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine

# 删除数据卷
docker volume rm my-vol

# 另一种方式 参数--volume（或简写为-v）只能创建bind mount
docker run -d -p 8085:80 -p 8083:22 -p 8082:443  \
       --name gitlab \
       --restart always \
       --volume /docker_volume/gitlab/config:/etc/gitlab \
       --volume /docker_volume/gitlab/logs:/var/log/gitlab \
       --volume /docker_volume/gitlab/data:/var/opt/gitlab \
       gitlab/gitlab-ce:latest
```
