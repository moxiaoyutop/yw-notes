# 基本操作

```
#查看集群node节点信息
cluster nodes

#查看集群信息
cluster info

#获取key的值
get name

#修改值
set key value

#查看key类型
type keys

#查看当前数据库有多少key
dbsize

#查看 hash类型值
hgetall keys

#注意0和1之间是空格；这个命令和pop命令不一样，不会删除里面的值
lrange list 0 1

#所有的
lrange list 0 -1

#命令行导出
redis-cli -a ZJFDqpN6GQDTsSqQ -c  -h 192.168.18.67 -p 7000 lrange keys:keys 0 662 >> panda.txt

#查看 maxmemory 参数
CONFIG GET maxmemory

#设置maxmemory参数
CONFIG SET maxmemory 12GB
```
