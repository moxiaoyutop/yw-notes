# 备份还原

### dumpling备份

```

#!/bin/bash
#当天日期
date_time=`date +%Y%m%d`

#sql备份
dumpling   -u root   -P 4000   -H 127.0.0.1 -pK71SKrkjxn  --filetype sql   --threads 32   -o /backup/all$date_time   -F $(( 1024 * 1024 * 256 ))

#打包
cd /backup
tar zcvf all$date_time.tar.gz all$date_time && rm -rf all$date_time

echo "备份完毕" `date` >> /tmp/backup.log
```

### tidb-ligthting还原

```
vim tidb-lightning.toml

[lightning]
# 日志
level = "info"
file = "tidb-lightning.log"

[tikv-importer]
# 选择使用的 local 后端
backend = "local"
# 设置排序的键值对的临时存放地址，目标路径需要是一个空目录
sorted-kv-dir = "/mnt/ssd/sorted-kv-dir"

[mydumper]
# 源数据目录。
data-source-dir = "/root/all20220105/"

# 配置通配符规则，默认规则会过滤 mysql、sys、INFORMATION_SCHEMA、PERFORMANCE_SCHEMA、METRICS_SCHEMA、INSPECTION_SCHEMA 系统数据库下的所有表
# 若不配置该项，导入系统表时会出现“找不到 schema”的异常
filter = ['*.*', '!mysql.*', '!sys.*', '!INFORMATION_SCHEMA.*', '!PERFORMANCE_SCHEMA.*', '!METRICS_SCHEMA.*', '!INSPECTION_SCHEMA.*']
[tidb]
# 目标集群的信息
host = "118.193.46.154"
port = 4000
user = "root"
password = "top123456"
# 表架构信息在从 TiDB 的“状态端口”获取。
status-port = 10080
# 集群 pd 的地址
pd-addr = "118.193.46.154:2379"
```

> 可以强制还原数据&#x20;
>
> \--check-requirements=false

![](<../../.gitbook/assets/image (16).png>)
