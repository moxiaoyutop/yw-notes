# 问题定位

当Elasticsearch本身的配置没有明显的问题之后，发现ES使用还是非常慢，这个时候，就需要我们去定位ES本身的问题了，首先祭出定位问题的第一个命令：

### hot\_threads

```
GET /_nodes/hot_threads&interval=30s
```

> 抓取30s的节点上占用资源的热线程，并通过排查占用资源最多的TOP线程来判断对应的资源消耗是否正常。一般情况下，bulk，search类的线程占用资源都可能是业务造成的，但是如果是merge线程占用了大量的资源，就应该考虑是不是创建index或者刷磁盘间隔太小，批量写入size太小造成的。

{% embed url="https://www.elastic.co/guide/en/elasticsearch/reference/6.x/cluster-nodes-hot-threads.html" %}
官方链接
{% endembed %}

### pending\_tasks

```
GET /_cluster/pending_tasks
```

> 有一些任务只能由主节点去处理，比如创建一个新的索引或者在集群中移动分片，由于一个集群中只能有一个主节点，所以只有这一master节点可以处理集群级别的元数据变动。
>
>
>
> 在99.9999%的时间里，这不会有什么问题，元数据变动的队列基本上保持为零。在一些罕见的集群里，元数据变动的次数比主节点能处理的还快，这会导致等待中的操作会累积成队列。\
>
>
> 这个时候可以通过pending\_tasks api分析当前什么操作阻塞了ES的队列，比如，集群异常时，会有大量的shard在recovery，如果集群在大量创建新字段，会出现大量的put\_mappings的操作，所以正常情况下，需要禁用动态mapping。

{% embed url="https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-pending.html" %}
官方链接
{% endembed %}

### 字段存储

> 当前es主要有doc\_values，fielddata，storefield三种类型，大部分情况下，并不需要三种类型都存储，可根据实际场景进行调整：
>
> * 当前用得最多的就是**doc\_values**，列存储，对于不需要进行分词的字段，都可以开启doc\_values来进行存储（且只保留keyword字段），节约内存，当然，开启doc\_values会对查询性能有一定的影响，但是，这个性能损耗是比较小的，而且是值得的；
>
>
>
> * **fielddata**构建和管理 100% 在内存中，常驻于 JVM 内存堆，所以可用于快速查询，但是这也意味着它本质上是不可扩展的，有很多边缘情况下要提防，如果对于字段没有分析需求，可以关闭fielddata；
>
>
>
> * **storefield**主要用于\_source字段，默认情况下，数据在写入es的时候，es会将doc数据存储为\_source字段，查询时可以通过\_source字段快速获取doc的原始结构，如果没有update，reindex等需求，可以将\_source字段disable；
>
>
>
> * **\_all**，ES在6.x以前的版本，默认将写入的字段拼接成一个大的字符串，并对该字段进行分词，用于支持整个doc的全文检索，在知道doc字段名称的情况下，建议关闭掉该字段，节约存储空间，也避免不带字段key的全文检索；
>
>
>
> * **norms**：搜索时进行评分，日志场景一般不需要评分，建议关闭。

### tranlog

> Elasticsearch 2.0之后为了保证不丢数据，每次 index、bulk、delete、update 完成的时候，一定触发刷新 translog 到磁盘上，才给请求返回 200 OK。这个改变在提高数据安全性的同时当然也降低了一点性能。如果你不在意这点可能性，还是希望性能优先，可以在 index template 里设置如下参数：

```
{

    "index.translog.durability": "async"

}
```

> **index.translog.sync\_interval：**
>
>
>
> 对于一些大容量的偶尔丢失几秒数据问题也并不严重的集群，使用异步的 fsync 还是比较有益的。\
>
>
> 比如，写入的数据被缓存到内存中，再每5秒执行一次 fsync ，默认为5s。小于的值100ms是不允许的。\
>
>
> **index.translog.flush\_threshold\_size：**\
>
>
> translog存储尚未安全保存在Lucene中的所有操作。虽然这些操作可用于读取，但如果要关闭并且必须恢复，则需要重新编制索引。\
>
>
> 此设置控制这些操作的最大总大小，以防止恢复时间过长。达到设置的最大size后，将发生刷新，生成新的Lucene提交点，默认为512mb。

### refresh\_interval

> 执行刷新操作的频率，这会使索引的最近更改对搜索可见，默认为1s，可以设置-1为禁用刷新，对于写入速率要求较高的场景，可以适当的加大对应的时长，减小磁盘io和segment的生成。

### 禁止动态mapping

> 动态mapping的坏处：
>
> * 造成集群元数据一直变更，导致集群不稳定；
> * 可能造成数据类型与实际类型不一致；
> * 对于一些异常字段或者是扫描类的字段，也会频繁的修改mapping，导致业务不可控。
>
>
>
> 动态mapping配置的可选值及含义如下：
>
> * **true**：支持动态扩展，新增数据有新的字段属性时，自动添加对于的mapping，数据写入成功；
> * **false**：不支持动态扩展，新增数据有新的字段属性时，直接忽略，数据写入成功 ；
> * **strict**：不支持动态扩展，新增数据有新的字段时，报错，数据写入失败。

