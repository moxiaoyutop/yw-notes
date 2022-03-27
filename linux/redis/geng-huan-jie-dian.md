# 更换节点

> 线上环境为redis六主六从，其中有一台服务器riad坏了，导致无法启动，集群状态是正常的，但是为了不影响业务，新增两个slave节点
>
> 一、部署好新的reids
>
> 部署过程看三主三从部署
>
> 二、查询主从关系(列：)

```
# 当前节点为 192.168.18.67:7000
info replication
```

![](<../../.gitbook/assets/image (13).png>)

> **给master添加新节点为slave**
>
> **因为添加的是slave 比较简单，可以直接添加，如果需要添加master。需要额外分配分片**

```
# 192.168.18.161:7014为新添加节点，192.168.18.67:7001为集群任意节点 最后面的id为master
redis-cli -a ueS4iFLjKij2YlP4 --cluster add-node 192.168.18.161:7014 192.168.18.67:7001 --cluster-slave --cluster-master-id f4ca0215fc0782d71e8b668477b3c1223a5943a7
```

> 剔除 fail挂壁的redis节点

```
cluster forget 38c4e14f178434e3fe8404a558f096c7cfde6dc5
cluster forget 9dac8d5678810f30741f226fa45adf9815683ac8
```
