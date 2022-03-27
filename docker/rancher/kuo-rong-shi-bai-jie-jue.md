# 扩容失败解决

> &#x20;报错信息
>
> Failed to reconcile etcd plane: Failed to add etcd member \[etcd-node2] to etcd cluster

解决：

```
# 查看当前etcd集群是否可以用，找到不可用的删掉
docker exec etcd etcdctl member list
docker exec etcd etcdctl member remove 61ef31d196b83635
```
