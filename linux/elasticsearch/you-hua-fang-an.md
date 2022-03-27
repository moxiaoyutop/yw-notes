---
description: ES是一个P2P类型的分布式系统，使用gossip协议
---

# 优化方案

### CPU 配置 <a href="#cpu-pei-zhi" id="cpu-pei-zhi"></a>

一般说来，CPU 繁忙的原因有以下几个：

* 线程中有无限空循环、无阻塞、正则匹配或者单纯的计算；
* 发生了频繁的 GC；
* 多线程的上下文切换；

大多数 Elasticsearch 部署往往对 CPU 要求不高。因此，相对其它资源，具体配置多少个（CPU）不是那么关键。你应该选择具有多个内核的现代处理器，常见的集群使用 2 到 8 个核的机器。如果你要在更快的 CPUs 和更多的核数之间选择，选择更多的核数更好。多个内核提供的额外并发远胜过稍微快一点点的时钟频率。

### **内存分配**

当机器内存小于 64G 时，遵循通用的原则，50% 给 ES，50% 留给 lucene。

当机器内存大于 64G 时，遵循以下原则：

* 如果主要的使用场景是全文检索，那么建议给 ES Heap 分配 4\~32G 的内存即可；其它内存留给操作系统，供 lucene 使用（segments cache），以提供更快的查询性能。
* 如果主要的使用场景是聚合或排序，并且大多数是 numerics，dates，geo\_points 以及 not\_analyzed 的字符类型，建议分配给 ES Heap 分配 4\~32G 的内存即可，其它内存留给操作系统，供 lucene 使用，提供快速的基于文档的聚类、排序性能。
* 如果使用场景是聚合或排序，并且都是基于 analyzed 字符数据，这时需要更多的 heap size，建议机器上运行多 ES 实例，每个实例保持不超过 50% 的 ES heap 设置（但不超过 32 G，堆内存设置 32 G 以下时，JVM 使用对象指标压缩技巧节省空间），50% 以上留给 lucene。

### 内存锁定

```
bootstrap.memory_lock：true允许JVM锁住内存，禁止操作系统交换出去。
```

### zen.discovery

> Elasticsearch默认被配置为使用单播发现，以防止节点无意中加入集群。组播发现应该永远不被使用在生产环境了，否则你得到的结果就是一个节点意外的加入到了你的生产环境，仅仅是因为他们收到了一个错误的组播信号。
>
>
>
> ES是一个P2P类型的分布式系统，使用gossip协议，集群的任意请求都可以发送到集群的任一节点，然后ES内部会找到需要转发的节点，并且与之进行通信。
>
>
>
> 在ES1.x的版本，ES默认是开启组播，启动ES之后，可以快速将局域网内集群名称，默认端口的相同实例加入到一个大的集群，后续再ES2.x之后，都调整成了单播，避免安全问题和网络风暴。
>
>
>
> 单播 discovery.zen.ping.unicast.hosts，建议写入集群内所有的节点及端口，如果新实例加入集群，新实例只需要写入当前集群的实例，即可自动加入到当前集群，之后再处理原实例的配置即可，新实例加入集群，不需要重启原有实例；\
>
>
> 节点zen相关配置：discovery.zen.ping\_timeout：判断master选举过程中，发现其他node存活的超时设置，主要影响选举的耗时，参数仅在加入或者选举 master 主节点的时候才起作用discovery.zen.join\_timeout：节点确定加入到集群中，向主节点发送加入请求的超时时间，默认为3sdiscovery.zen.minimum\_master\_nodes：参与master选举的最小节点数，当集群能够被选为master的节点数量小于最小数量时，集群将无法正常选举。

### 故障检测(fault detection)

> 两种情况下会进行故障检测：
>
> * 第一种是由master向集群的所有其他节点发起ping，验证节点是否处于活动状态；
> * 第二种是：集群每个节点向master发起ping，判断master是否存活，是否需要发起选举。故障检测需要配置以下设置使用 形如：discovery.zen.fd.ping\_interval节点被ping的频率，默认为1s。discovery.zen.fd.ping\_timeout 等待ping响应的时间，默认为 30s，运行的集群中，master 检测所有节点，以及节点检测 master 是否正常。discovery.zen.fd.ping\_retries ping失败/超时多少导致节点被视为失败，默认为3。

{% embed url="https://www.elastic.co/guide/en/elasticsearch/reference/6.x/modules-discovery-zen.html" %}
官方链接
{% endembed %}

### 队列数量

> 不建议盲目加大ES的队列数量，如果是偶发的因为数据突增，导致队列阻塞，加大队列size可以使用内存来缓存数据；如果是持续性的数据阻塞在队列，加大队列size除了加大内存占用，并不能有效提高数据写入速率，反而可能加大ES宕机时候，在内存中可能丢失的上数据量。\
>
>
> 哪些情况下，加大队列size呢？GET /\_cat/thread\_pool，观察api中返回的queue和rejected，如果确实存在队列拒绝或者是持续的queue，可以酌情调整队列size。

{% embed url="https://www.elastic.co/guide/en/elasticsearch/reference/6.x/modules-threadpool.html" %}
官方链接
{% endembed %}

### **禁止 swap**

禁止 swap，一旦允许内存与磁盘的交换，会引起致命的性能问题。可以通过在 elasticsearch.yml 中 bootstrap.memory\_lock: true，以保持 JVM 锁定内存，保证 ES 的性能。

### **GC 设置**

[老的版本中官方文档  (opens new window)](https://www.elastic.co/guide/cn/elasticsearch/guide/current/dont-touch-these-settings.html)中推荐默认设置为：Concurrent-Mark and Sweep（CMS），给的理由是当时G1 还有很多 BUG。

原因是：已知JDK 8附带的HotSpot JVM的早期版本存在一些问题，当启用G1GC收集器时，这些问题可能导致索引损坏。受影响的版本早于JDK 8u40随附的HotSpot版本。来源于[官方说明  (opens new window)](https://www.elastic.co/guide/en/elasticsearch/reference/current/\_g1gc\_check.html)

实际上如果你使用的JDK8较高版本，或者JDK9+，我推荐你使用G1 GC； 因为我们目前的项目使用的就是G1 GC，运行效果良好，对Heap大对象优化尤为明显。修改jvm.options文件，将下面几行:

```
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly

```

更改为

```
-XX:+UseG1GC
-XX:MaxGCPauseMillis=50

```

其中 -XX:MaxGCPauseMillis是控制预期的最高GC时长，默认值为200ms，如果线上业务特性对于GC停顿非常敏感，可以适当设置低一些。但是 这个值如果设置过小，可能会带来比较高的cpu消耗。

G1对于集群正常运作的情况下减轻G1停顿对服务时延的影响还是很有效的，但是如果是你描述的GC导致集群卡死，那么很有可能换G1也无法根本上解决问题。 通常都是集群的数据模型或者Query需要优化。

### 磁盘 <a href="#ci-pan" id="ci-pan"></a>

硬盘对所有的集群都很重要，对大量写入的集群更是加倍重要（例如那些存储日志数据的）。硬盘是服务器上最慢的子系统，这意味着那些写入量很大的集群很容易让硬盘饱和，使得它成为集群的瓶颈。

**在经济压力能承受的范围下，尽量使用固态硬盘（SSD）**。固态硬盘相比于任何旋转介质（机械硬盘，磁带等），无论随机写还是顺序写，都会对 IO 有较大的提升。

> 1. 如果你正在使用 SSDs，确保你的系统 I/O 调度程序是配置正确的。当你向硬盘写数据，I/O 调度程序决定何时把数据实际发送到硬盘。大多数默认 \*nix 发行版下的调度程序都叫做 cfq（完全公平队列）。
> 2. 调度程序分配时间片到每个进程。并且优化这些到硬盘的众多队列的传递。但它是为旋转介质优化的：机械硬盘的固有特性意味着它写入数据到基于物理布局的硬盘会更高效。
> 3. 这对 SSD 来说是低效的，尽管这里没有涉及到机械硬盘。但是，deadline 或者 noop 应该被使用。deadline 调度程序基于写入等待时间进行优化，noop 只是一个简单的 FIFO 队列。

**这个简单的更改可以带来显著的影响。仅仅是使用正确的调度程序，我们看到了 500 倍的写入能力提升**。

如果你使用旋转介质（如机械硬盘），尝试获取尽可能快的硬盘（高性能服务器硬盘，15k RPM 驱动器）。

**使用 RAID0 是提高硬盘速度的有效途径，对机械硬盘和 SSD 来说都是如此**。没有必要使用镜像或其它 RAID 变体，因为 Elasticsearch 在自身层面通过副本，已经提供了备份的功能，所以不需要利用磁盘的备份功能，同时如果使用磁盘备份功能的话，对写入速度有较大的影响。

最后，**避免使用网络附加存储（NAS）**。人们常声称他们的 NAS 解决方案比本地驱动器更快更可靠。除却这些声称，我们从没看到 NAS 能配得上它的大肆宣传。NAS 常常很慢，显露出更大的延时和更宽的平均延时方差，而且它是单点故障的。

### 内存使用

> 设置indices的内存熔断相关参数，根据实际情况进行调整，防止写入或查询压力过高导致OOM：

```
indices.breaker.total.limit：50%，集群级别的断路器，默认为jvm堆的70%；

indices.breaker.request.limit：10%，单个request的断路器限制，默认为jvm堆的60%；

indices.breaker.fielddata.limit：10%，fielddata breaker限制，默认为jvm堆的60%。
```

{% embed url="https://www.elastic.co/guide/en/elasticsearch/reference/6.x/circuit-breaker.html" %}

> 根据实际情况调整查询占用cache，避免查询cache占用过多的jvm内存，参数为静态的，需要在每个数据节点配置。indices.queries.cache.size: 5%，控制过滤器缓存的内存大小，默认为10%。接受百分比值，5%或者精确值，例如512mb。

{% embed url="https://www.elastic.co/guide/en/elasticsearch/reference/6.x/query-cache.html" %}

### 创建shard

> 如果集群规模较大，可以阻止新建shard时扫描集群内全部shard的元数据，提升shard分配速度。

```
cluster.routing.allocation.disk.include_relocations: false，默认为true。
```

{% embed url="https://www.elastic.co/guide/en/elasticsearch/reference/6.x/disk-allocator.html" %}
