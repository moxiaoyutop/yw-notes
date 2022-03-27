# 基础操作

```
#查看集群状态
curl -X GET "localhost:9200/_cat/health?v"
curl -XGET 'http://localhost:9200/_cluster/health?pretty'

#查看所有索引
curl 'localhost:9200/_cat/indices?v'

#删除单天日志
curl -XDELETE 'http://127.0.0.1:9200/*-log-2020.02.*'

#查看各个节点的分片数量
curl -X GET "localhost:9200/_cat/allocation?v"

#显示每个节点的线程池统计信息
#active 表示当前正在执行任务的线程数量
#queue 表示队列中等待处理的请求数量
#rejected 表示由于队列已满，请求被拒绝的次数。
curl -X GET "localhost:9200/_cat/thread_pool"

#返回所有节点的统计数据
curl -X GET "localhost:9200/_nodes/stats"

#返回特定节点的统计数据
curl -X GET "localhost:9200/_nodes/nodeId1, nodeId2/stats"

#返回集群中一个或全部节点的热点线程
curl -X GET "localhost:9200/_nodes/hot_threads"
curl -X GET "localhost:9200/nodes/nodeId1, nodeId2/hot_threads"
```
