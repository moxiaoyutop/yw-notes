# 集群部署

> 通过es的官方文档中得知，es内存配置最优配置为32G内存
>
> 安装完系统之后优化服务器de

```
ulimit -n 65535
#查询
ulimit -a

# 打开文件数
vim /etc/security/limits.conf
elasticsearch  -  nofile  65535

#禁用所有交换文件
swapoff -a

#配置 swappiness
vim /etc/sysctl.conf
vm.swappiness = 1
sysctl -p
#Linux 系统上可用的另一个选项是确保将 sysctl 值 vm.swappiness设置为1. 这减少了内核交换的倾向，并且在正常情况下不应该导致交换，同时仍然允许整个系统在紧急情况下进行交换。
```

> ### 线程数
>
> Elasticsearch 使用多个线程池来执行不同类型的操作。重要的是它能够在需要时创建新线程。确保 Elasticsearch 用户可以创建的线程数至少为 4096。
>
> 这可以通过[`ulimit -u 4096`](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/setting-system-settings.html#ulimit)在启动 Elasticsearch 之前设置为 root 或设置`nproc`为`4096`in 来完成[`/etc/security/limits.conf`](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/setting-system-settings.html#limits.conf)。
>
> 当作为服务运行时，包分发`systemd`将自动为 Elasticsearch 进程配置线程数。无需额外配置。
>
> [https://www.elastic.co/guide/en/elasticsearch/reference/6.8/max-number-of-threads.html](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/max-number-of-threads.html)

### 开始安装elasticsearch

```
#java安装
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie"  http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz

tar xf jdk-8u131-linux-x64.tar.gz -C /usr/local/

mv /usr/local/jdk1.8.0_131/ /usr/local/java

vim /etc/profile
JAVA_HOME=/usr/local/java
PATH=$PATH:$JAVA_HOME/bin

ln -s /usr/local/java/bin/java /usr/bin/

#查看版本
java -version

#rpm包下载
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.2.rpm

yum install elasticsearch-6.8.2.rpm -y

mkdir /etc/systemd/system/elasticsearch.service.d/

cat >> /etc/systemd/system/elasticsearch.service.d/override.conf << EOF
[Service]
LimitMEMLOCK=infinity
EOF

systemctl daemon-reload

vim /etc/elasticsearch/elasticsearch.yml
#修改配置文件
elasticsearch.yml
集群配置好 集群名字 ip 脑裂 node-1 node-2 node-3

#查看集群
curl -XGET 'http://localhost:9200/_cluster/health?pretty'
```

> 配置文件

> 为防止数据丢失，配置 `discovery.zen.minimum_master_nodes`设置至关重要，以便每个符合主节点条件的节点都知道必须可见_的主节点节点的最小数量才能形成集群。_
>
> 为避免脑裂，应将此设置设置为符合主节点的_法定人数：_
>
> 如果有三个符合主节点资格的节点，那么最小主节点应该设置为`(3 / 2) + 1`或`2`：
>
> ```
> discovery.zen.minimum_master_nodes: 2
> ```

```
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
cluster.name: it-pro-es
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: node-1
#
# Add custom attributes to the node:
#
node.master: true
node.data: true
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /var/lib/elasticsearch
#
# Path to log files:
#
path.logs: /var/log/elasticsearch
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#

network.host: 0.0.0.0 
#
# Set a custom port for HTTP:
#
http.port: 9200
http.cors.enabled: true
http.cors.allow-origin: "*"
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.zen.ping.unicast.hosts: ["192.168.18.57", "192.168.18.58", "192.168.18.59", "192.168.18.152", "192.168.18.153"]
#
# Prevent the "split brain" by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
#
#discovery.zen.minimum_master_nodes: 
discovery.zen.minimum_master_nodes: 3
# For more information, consult the zen discovery module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
gateway.recover_after_nodes: 5
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
```
