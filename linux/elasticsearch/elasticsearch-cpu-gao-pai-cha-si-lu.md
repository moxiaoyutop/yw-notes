# Elasticsearch CPU高排查思路

### 一、可能导致ES CPU高的原因： <a href="#slide-0" id="slide-0"></a>

1、复杂的query查询

> 举例：我这边出现过200个组合wildcard query导致集群down掉的情况；

2、有大量的reindex操作

3、ES版本较低

### 二、排查思路 <a href="#slide-1" id="slide-1"></a>

#### 1、业务场景排查 <a href="#slide-2" id="slide-2"></a>

问自己几个问题？

* 1）集群中数据类型是怎么样的？
* 2）集群中有多少数据？
* 3）集群中有多少节点数、分片数？
* 4）当前集群索引和检索的速率如何？
* 5）当前在执行哪种类型的查询或者其他操作？

#### 2、建议Htop观察，结合ElaticHQ 观察CPU曲线 <a href="#slide-3" id="slide-3"></a>

#### 3、CPU高的时候，建议看一下ES节点的日志，看看是不是有大量的GC。 <a href="#slide-4" id="slide-4"></a>

#### 4、查看hot\_threads。 <a href="#slide-5" id="slide-5"></a>

```
GET _nodes/hot_threads

::: {test}{ikKuXkFvRc-qFCqG99smGg}{VE-uqoiARoONJwomfPwRBw}{127.0.0.1}{127.0.0.1:9300}{ml.machine_memory=8481566720, ml.max_open_jobs=20, ml.enabled=true}
   Hot threads at 2018-04-09T15:58:21.117Z, interval=500ms, busiestThreads=3, ignoreIdleThreads=true:

    0.0% (0s out of 500ms) cpu usage by thread 'Attach Listener'
     unique snapshot
     unique snapshot
     unique snapshot
     unique snapshot
     unique snapshot
     unique snapshot
     unique snapshot
     unique snapshot
     unique snapshot
     unique snapshot
```

### 三、解决方案： <a href="#slide-6" id="slide-6"></a>

#### 3.1、集群负载高，增加新节点以缓解负载。 <a href="#slide-7" id="slide-7"></a>

#### 3.2、增加堆内存到系统内存的1半，最大31GB（理论上线32GB）. <a href="#slide-8" id="slide-8"></a>

> 如果机器内存不够，那就加大内存吧。\
> [https://github.com/elastic/elasticsearch/issues/10437](https://github.com/elastic/elasticsearch/issues/10437)\
> [https://discuss.elastic.co/t/es-high-cpu-usage-when-idle/87950/4](https://discuss.elastic.co/t/es-high-cpu-usage-when-idle/87950/4)

#### 3.3、插入数据的时候，副本数设置为0. <a href="#slide-9" id="slide-9"></a>

分片数不可以修改，副本数是可以修改的。

注意：分片过多，会导致：堆内存压力大。

#### 3.4、配置优化 <a href="#slide-10" id="slide-10"></a>

```
# 强制锁定所有内存，强制 JVM 从不交换
bootstrap.mlockall: true

# 线程池设置
# 搜索池
threadpool.search.type: fixed
threadpool.search.size: 20
threadpool.search.queue_size: 200

# Bulk pool
threadpool.bulk.type: fixed
threadpool.bulk.size: 60
threadpool.bulk.queue_size: 3000

# Index pool
threadpool.index.type: fixed
threadpool.index.size: 20
threadpool.index.queue_size: 1000

# Indices settings
indices.memory.index_buffer_size: 30%
indices.memory.min_shard_index_buffer_size: 12mb
indices.memory.min_index_buffer_size: 96mb

# Cache Sizes
indices.fielddata.cache.size: 30%
#indices.fielddata.cache.expire: 6h #will be depreciated & Dev recomend not to use it
indices.cache.filter.size: 30%
#indices.cache.filter.expire: 6h #will be depreciated & Dev recomend not to use it

# 写入的索引设置
index.refresh_interval: 30s
#index.translog.flush_threshold_ops: 50000
#index.translog.flush_threshold_size: 1024mb
index.translog.flush_threshold_period: 5m
index.merge.scheduler.max_thread_count: 1
```

\
