# 基础操作

```
# 集群操作命令
tiup cluster start tidb-test          #启动集群
tiup cluster display tidb-test        #检查集群状态
tiup cluster prune tidb-test          #删除下线的节点
tiup cluster destroy ${cluster-name}  #销毁集群！！

# 显示pd成员信息
tiup ctl pd member --pd "http://192.168.18.75:2379"

# 使用id下线节点
tiup ctl pd member delete id 13716334332156344902 --pd "http://192.168.18.75:2379"

# 使用name下线节点
member delete name pd2

# 显示集群健康信息
tiup ctl pd health --pd "http://192.168.18.75:2379"

# 扩容缩容依照官网实例
# 扩容只要机器正常环境部署好不会有什么问题

# 缩容节点
tiup cluster scale-in <cluster-name> --node 10.0.1.4:9000

# 强制缩容
tiup cluster scale-in <cluster-name> --force --node 10.0.1.4:9000

# 缩容如果状态为Tombstone
# 销毁已经下线的节点
tiup cluster prune tidb-test

# TIDB-BR备份操作
# 下载br工具
wget https://download.pingcap.org/tidb-toolkit-v4.0.7-linux-amd64.tar.gz

# 全局备份
./br backup full \
    --pd "192.168.0.13:2379" \
    --storage "local:///tmp/backup" \
    --ratelimit 120 \
    --log-file backupfull.log

# 全局还原
./br restore full \
    --pd "192.168.0.36:2379" \
    --storage "local:///tmp/backup" \
    --ratelimit 128 \
    --log-file restorefull.log
```

> &#x20;参数优化

```
vim /etc/security/limits.conf
* soft nofile 655360
* hard nofile 655360
* soft nproc 655360
* hard nproc 655360
* soft memlock unlimited
* hard memlock unlimited

vim /etc/sysctl.conf
vm.max_map_count = 655360
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_max_syn_backlog = 8192
net.core.netdev_max_backlog = 10000
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

yum -y install numactl.x86_64
```

```
# 修改root密码
SET PASSWORD FOR 'root'@'%' = 'top123456';
FLUSH PRIVILEGES;

#创建只读
create user rw_it identified by '123456';
GRANT SELECT ON *.* TO 'rw_it'@'%' IDENTIFIED BY "123456";
```
