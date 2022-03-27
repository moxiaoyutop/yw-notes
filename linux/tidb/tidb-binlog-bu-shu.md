# tidb binlog部署

```
#扩容之前先做同步
#备份整个数据库 全备
/root/tidb-toolkit-v4.0.7-linux-amd64/bin/mydumper --ask-password -h 192.168.0.34 -P 4000 -u root --threads=4 --chunk-filesize=64 --skip-tz-utc --regex '^(?!(mysql\.|information_schema\.|performance_schema\.))'  -o /mfw_rundata/dump/ --verbose=3
#打包发给mysql机器

#mysql机器安装myloader
yum install -y cmake gcc gcc-c++ git make glib2-devel mysql-devel openssl-devel pcre-devel zlib-devel
yum install -y https://github.com/maxbube/mydumper/releases/download/v0.10.3/mydumper-0.10.3-1.el7.x86_64.rpm
 
#使用myloader 恢复到mysql内
myloader -d /root/dump/ -h 192.168.0.13 -u root -p Top123456\!\@\# -P 3306 -t 4

#查看 pos 并且在mysql里面的bing_log里修改备份后的pos
cat /root/dump/metadata

#添加扩容机器和组件
[root@localhost ~]# cat scale-out.yaml
pump_servers:
  - host: 192.168.0.34
  - host: 192.168.0.35
  - host: 192.168.0.36
drainer_servers:
  - host: 192.168.0.13
    config:
      syncer.db-type: "mysql"
      syncer.to.host: "192.168.0.13"
      syncer.to.user: "root"
      syncer.to.password: "Top123456!@#"
      syncer.to.port: 3306
 
 #扩容
 tiup cluster scale-out tidb-dev scale-out.yaml -p
 
 #在线编辑配置文件
 tiup cluster edit-config tidb-dev
 server_configs:
  tidb:
    binlog.enable: true
    binlog.ignore-error: true
 
 #加载配置文件   
 tiup cluster reload tidb-dev
 
 #重启tidb集群
 tiup cluster reload tidb-dev
 
 
 #数据库操作
 #查看 TiDB 是否开启 binlog，0 代表关闭，1 代表开启
 show variables like "log_bin";
 #查看 Pump/Drainer 状态 online：正常运行中 pausing：暂停中 paused：已暂停 closing：下线中 offline：已下线
 show pump status;
 show drainer status;
```
