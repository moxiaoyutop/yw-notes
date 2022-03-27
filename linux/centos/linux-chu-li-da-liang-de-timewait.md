# Linux-处理大量的 TIME\_WAIT

```bash
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
```

然后执行 /sbin/sysctl -p 让参数生效。

`net.ipv4.tcp_syncookies = 1 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；`&#x20;

`net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将time-wait sockets重新用于新的TCP连接，默认为0，表示关闭；`&#x20;

`net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。`&#x20;

`net.ipv4.tcp_fin_timeout 修改系統默认的 TIMEOUT 时间`
