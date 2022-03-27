# Ngx 服务器内核优化

一个完整的内核优化配置

```
net.ipv4.ip_forward = 0
net.ipv4.conf.default.rp_filter =1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
net.ipv4.tcp_syncookies = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
net.ipv4.tcp_max_tw_buckets = 6000
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 16384 4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.netdev_max_backlog = 262144
net.core.somaxconn = 262144
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.ipv4.tcp_fin_timeout = 1
net.ipv4.tcp_keepalive_time = 30
net.ipv4.ip_local_port_range = 1024 65000
```

优化参考详解

```
# 允许系统打开的端口范围。
net.ipv4.tcp_tw_recycle = 1  

# 启用timewait快速回收。          
net.ipv4.tcp_tw_reuse = 1   

# 开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接。         
net.ipv4.tcp_tw_reuse = 1              

# 开启SYN Cookies，当出现SYN等待队列溢出时，启用cookies来处理
net.ipv4.tcp_syncookies = 1

#  # 系统中最多有多少个TCP套接字不被关联到任何一个用户文件句柄上，可防止简单的DoS攻击
net.ipv4.tcp_max_orphans = 262144

# 默认限制128, 而nginx当中的NGX_LISTEN_BACKLOG默认为511
net.core.somaxconn = 262144

# 每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目。
net.core.netdev_max_backlog = 262144


# 记录的那些尚未收到客户端确认信息的连接请求的最大值。对于有128M内存的系统而言，缺省值是1024，小内存的系统则是128。
net.ipv4.tcp_max_syn_backlog = 262144


# 关闭时间戳，避免序列号的卷绕
net.ipv4.tcp_timestamps = 0


# 决定了内核放弃连接之前发送SYN+ACK包的数量。
net.ipv4.tcp_synack_retries = 1

# 在内核放弃建立连接之前发送SYN包的数量。
net.ipv4.tcp_syn_retries = 1


# 缺省值是60秒。2.2 内核的通常值是180秒，即使你的机器是一个轻载的WEB服务器，也有因为大量的死套接字而内存溢出的风险
net.ipv4.tcp_fin_timeout = 1


# 当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时
net.ipv4.tcp_keepalive_time = 30
```
