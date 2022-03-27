# 集群扩容缩容

> 将某个节点从集群中剔除，该集群的分片会立刻迁移到别的节点，一直到0

```
curl -X PUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
{
  "transient" : {
    "cluster.routing.allocation.exclude._ip" : "10.0.0.1"
  }
}
'
```

> 检查分片情况
>
> 确认分片数量为0后，即可登入到需要扩容节点的系统中停止elasticsearch服务并关机。
>
> 然后实施重装系统或挂载磁盘等操作，完成后重新启动elasticsearch服务即可。

```
curl -X GET "localhost:9200/_cat/allocation?v"
```

> node重新加入集群后并不会自动同步分片，因为上面已经将它的IP剔除了，此时需要执行以下命令

```
curl -X PUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
{
  "transient" : {
    "cluster.routing.allocation.include._ip" : "10.0.0.*"
  }
}
'
```

```
如果是新节点添加到集群，那么可直接部署好es 修改好配置文件，重启集群即可加入。
```

