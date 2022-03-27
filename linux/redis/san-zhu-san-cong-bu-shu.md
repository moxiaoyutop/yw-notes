# 三主三从部署

```
#安装依赖
yum install -y gcc gcc-c++ make

#下载
wget http://download.redis.io/releases/redis-5.0.8.tar.gz

#解压、编译
tar -zxvf redis-5.0.8.tar.gz
cd redis-5.0.8
make MALLOC=libc PREFIX=/usr/local/redis install
#创建Redis相关工作目录(目录可自定义) 3台机器都操作 路径不同自行修改
mkdir /usr/local/redis/{data/{redis_7000,redis_7001},conf,log} -p

拷贝从配置文件
cp /usr/local/redis/conf/redis_7000.conf /usr/local/redis/conf/redis_7001.conf 
sed -i 's/7000/7001/g' /usr/local/redis/conf/redis_7001.conf
cat /usr/local/redis/conf/redis_7001.conf | grep -Ev '^#|^$'

配置软连接
ln -s /usr/local/redis/bin/redis-server /usr/bin/
ln -s /usr/local/redis/bin/redis-cli /usr/bin/

4.启动Redis
集群内每台服务器分别启动两个redis
redis-server /usr/local/redis/conf/redis_7000.conf
redis-server /usr/local/redis/conf/redis_7001.conf
#以上操作为三个节点均要操作

#创建集群 (只需要在node1节点执行即可)
redis-cli -a ZJFDqasdfeDTsSqQ --cluster create 152.32.185.40:7000 152.32.185.40:7001 152.32.191.135:7002 152.32.191.135:7003 128.1.132.11:7004 128.1.132.11:7005 --cluster-replicas 1

#登入集群
redis-cli -a ZJFDqasdfeDTsSqQ -c  -h 152.32.185.40 -p 7000

#查看集群node节点信息
cluster nodes
```

> 配置文件如下

```
cat /usr/local/redis/conf/redis-7001.conf | grep -Ev '^#|^$'

bind 0.0.0.0
protected-mode yes
port 7001
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised no
pidfile /usr/local/redis/conf/redis_7001.pid
loglevel notice
logfile "/usr/local/redis/log/redis_7001.log"
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir "/usr/local/redis/data/redis_7001/"
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
replica-priority 100
masterauth ZJFDqpN6GQDTsSqQ
requirepass ZJFDqpN6GQDTsSqQ
rename-command KEYS ""
maxmemory 26843545600
maxmemory-policy volatile-lru
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
cluster-enabled yes
cluster-config-file nodes-7001.conf
cluster-node-timeout 15000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events Ex
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
```
