# 集群部署

>

| 角色              | 作用                                                                                                                                                                                                        |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Producer`      | 消息发送者，将消息发送到 `Broker`。无状态，其与`NameServer`集群中的一个节点建立长连接，定期从`NameServer`获取`Topic`路由信息，并与提供`Topic`服务的`Master`建立连接，定时向`Master`发送心跳                                                                             |
| `Consumer`      | 消息消费者，从 `Broker` 中获取消息消费。其与`NameServer`集群中的一个节点建立长连接，定期从`NameServer`获取`Topic`路由信息，并与提供`Topic`服务的`Master`，`Slave`建立长连接，定时向`Master`，`Slave`发送心跳。`Consumer`既可以从`Master`订阅消息，也可以从`Slave`订阅消息，订阅规则由`Broker`配置  |
| `Broker`        | 存储和传输消息。分为`Master`和`Slave`，并且`Master`和`Slave`是一对多的关系，`Master`与 `Slave`对应关系通过相同的`BrokerName`，不同的`BrokerId`来定义（为`0`表示`Master`，非`0`表示`Slave`）每个`Broker`与`NameServer`集群中的所有节点建立长连接，定时注册`Topic`到所有`NameServer` |
| `NameServer`    | 负责管理`Broker`，比如生产者发送到哪个 `Broker`，由 `NameServer` 告知。集群中节点之间无信息同步，是无状态节点                                                                                                                                    |
| `Topic`         | 消息类别，一个发送者可以发送消息给一个或多个`Topic`，一个消息的接受者可以订阅一个或者多个 `Topic`                                                                                                                                                  |
| `Message Queue` | 用于并行发送和接收消息                                                                                                                                                                                               |

### 前提背景

> 3台服务器如下：
>
> 192.168.0.40 简称服务器A
>
> 192.168.0.41 简称服务器B
>
> 192.168.0.42 简称服务器C
>
> rocketmq-console访问地址：192.168.0.100:3033
>
> 每台服务器必须环境：
>
> JDK 本次采用：jdk1.8.0\_161
>
> Maven 本次采用：apache-maven-3.6.3
>
> rocketmq 本次采用：rocketmq-all-4.7.1-bin-release

### jdk maven安装

```
#下载java安装包 mvn安装包
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie"  http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
wget https://dlcdn.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz --no-check-certificate

#解压到路径
tar xf jdk-8u131-linux-x64.tar.gz -C /usr/local/
tar xf apache-maven-3.6.3-bin.tar.gz -C /usr/local/

#替换名字
mv /usr/local/jdk1.8.0_131/ /usr/local/java
mv /usr/local/apache-maven-3.6.3/ /usr/local/maven-3.6.3

#添加java环境变量
#在最后添加以下内容
vim /etc/profile
JAVA_HOME=/usr/local/java
PATH=$PATH:$JAVA_HOME/bin
export JAVA_HOME PATH
export MAVEN_HOME=/usr/local/maven-3.6.3
export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin

#加载以添加的环境变量
source /etc/profile

#制作软连接
ln -s /usr/local/java/bin/java /usr/bin/

#查看版本
java -version
mvn -v
which java
```

### 部署rocketmq

```
#下载所需安装包
wget https://archive.apache.org/dist/rocketmq/4.7.1/rocketmq-all-4.7.1-bin-release.zip
unzip rocketmq-all-4.7.1-bin-release.zip -d /usr/local/
mv /usr/local/rocketmq-all-4.7.1-bin-release/ /usr/local/rocketmq

#配置环境变量
vim /etc/profile
export ROCKETMQ_HOME=/usr/local/rocketmq
export PATH=$ROCKETMQ_HOME/bin:$PATH
source /etc/profile

#创建3m3s目录
mkdir /usr/local/rocketmq/conf/3m-3s-sync
```

### 分别修改各服务器配置文件

> 192.168.0.40 A

<mark style="color:red;">**测试环境**</mark>

```
# 集群名，不同broker节点集群名是一样的
brokerClusterName=It-Test-RocketmqCluster

# broker名字，不同broker节点brokerName是唯一的
brokerName=broker-a

# 0表示Master，大于0表示 Slave
brokerId=0

# 删除文件时间点，默认凌晨 4点
deleteWhen=04

# 文件保留时间，默认 48 小时
fileReservedTime=48

# 当前节点角色，ASYNC_MASTER=异步复制Master模式，SYNC_MASTER=同步双写Master模式
brokerRole=SYNC_MASTER

# 刷盘模式，ASYNC_FLUSH=异步刷盘，SYNC_FLUSH=同步刷盘
flushDiskType=SYNC_FLUSH

#nameServer地址，多个用英文分号分割
namesrvAddr=192.168.0.40:9876;192.168.0.41:9876;192.168.0.42:9876

# broker对外服务的监听端口，默认10911
listenPort=10911

# 是否允许 Broker 自动创建Topic，建议关闭
autoCreateTopicEnable=false

# 是否允许 Broker 自动创建订阅组，建议关闭
autoCreateSubscriptionGroup=false

#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824

#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000

#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88

#存储路径
storePathRootDir=/usr/local/rocketmq/store/rootdir-a-m

#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog-a-m

#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue-a-m

#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index-a-m

#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint-a-m

#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort-a-m

#限制的消息大小
maxMessageSize=65536
#############################
```

### 生产环境三台服务器配置文件

<mark style="color:red;">**192.168.0.40 - a**</mark>

```
vim broker-a.properties

rocketmqHome=/usr/local/rocketmq
# 如果是一个nameserver，直接写一个地址即可
namesrvAddr=192.168.0.40:9876;192.168.0.41:9876;192.168.0.42:9876
brokerName=borker-a
brokerClusterName=It-Pre-RocketmqCluster
brokerId=0
brokerPermission=6
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=8
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=false
clusterTopicEnable=true
brokerTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=false
sendMessageThreadPoolNums=1
pullMessageThreadPoolNums=20
queryMessageThreadPoolNums=10
adminBrokerThreadPoolNums=16
clientManageThreadPoolNums=32
consumerManageThreadPoolNums=32
heartbeatThreadPoolNums=2
endTransactionThreadPoolNums=12
flushConsumerOffsetInterval=5000
flushConsumerOffsetHistoryInterval=60000
rejectTransactionMessage=false
fetchNamesrvAddrByAddressServer=false
sendThreadPoolQueueCapacity=10000
pullThreadPoolQueueCapacity=100000
queryThreadPoolQueueCapacity=20000
clientManagerThreadPoolQueueCapacity=1000000
consumerManagerThreadPoolQueueCapacity=1000000
heartbeatThreadPoolQueueCapacity=50000
endTransactionPoolQueueCapacity=100000
filterServerNums=0
longPollingEnable=true
shortPollingTimeMills=1000
notifyConsumerIdsChangedEnable=true
highSpeedMode=false
commercialEnable=true
commercialTimerCount=1
commercialTransCount=1
commercialBigCount=1
commercialBaseCount=1
transferMsgByHeap=true
maxDelayTime=40
regionId=DefaultRegion
registerBrokerTimeoutMills=6000
slaveReadEnable=false
disableConsumeIfConsumerReadSlowly=false
consumerFallbehindThreshold=17179869184
brokerFastFailureEnable=true
waitTimeMillsInSendQueue=200
waitTimeMillsInPullQueue=5000
waitTimeMillsInHeartbeatQueue=31000
waitTimeMillsInTransactionQueue=3000
startAcceptSendRequestTimeStamp=0
traceOn=true
enableCalcFilterBitMap=false
expectConsumerNumUseFilter=32
maxErrorRateOfBloomFilter=20
filterDataCleanTimeSpan=86400000
filterSupportRetry=false
enablePropertyFilter=false
compressedRegister=false
forceRegister=true
registerNameServerPeriod=30000
transactionTimeOut=6000
transactionCheckMax=15
transactionCheckInterval=60000
#Broker 对外服务的监听端口
listenPort=10911
serverWorkerThreads=8
serverCallbackExecutorThreads=0
serverSelectorThreads=3
serverOnewaySemaphoreValue=256
serverAsyncSemaphoreValue=64
serverChannelMaxIdleTimeSeconds=120
serverSocketSndBufSize=131072
serverSocketRcvBufSize=131072
serverPooledByteBufAllocatorEnable=true
useEpollNativeSelector=false
clientWorkerThreads=16
clientCallbackExecutorThreads=2
clientOnewaySemaphoreValue=65535
clientAsyncSemaphoreValue=65535
connectTimeoutMillis=3000
channelNotActiveInterval=60000
clientChannelMaxIdleTimeSeconds=120
clientSocketSndBufSize=131072
clientSocketRcvBufSize=131072
clientPooledByteBufAllocatorEnable=false
clientCloseSocketIfTimeout=false
useTLS=false
storePathRootDir=/usr/local/rocketmq/store/rootdir-a-m
storePathCommitLog=/usr/local/rocketmq/store/commitlog-a-m
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue-a-m
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index-a-m
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint-a-m
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort-a-m
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存60W条，根据业务情况调整,
mapedFileSizeConsumeQueue=3000000
enableConsumeQueueExt=false
mappedFileSizeConsumeQueueExt=50331648
bitMapLengthConsumeQueueExt=64
flushIntervalCommitLog=500
commitIntervalCommitLog=200
useReentrantLockWhenPutMessage=false
flushCommitLogTimed=false
flushIntervalConsumeQueue=1000
cleanResourceInterval=10000
deleteCommitLogFilesInterval=100
deleteConsumeQueueFilesInterval=100
destroyMapedFileIntervalForcibly=120000
redeleteHangedFileInterval=120000
#删除文件时间点，默认凌晨 2点
deleteWhen=02
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=75
#文件保留时间，默认 72 小时
fileReservedTime=128
putMsgIndexHightWater=600000
#限制的消息大小
maxMessageSize=65536
checkCRCOnRecover=true
flushCommitLogLeastPages=4
commitCommitLogLeastPages=4
flushLeastPagesWhenWarmMapedFile=4096
flushConsumeQueueLeastPages=2
flushCommitLogThoroughInterval=10000
commitCommitLogThoroughInterval=200
flushConsumeQueueThoroughInterval=60000
maxTransferBytesOnMessageInMemory=262144
maxTransferCountOnMessageInMemory=32
maxTransferBytesOnMessageInDisk=65536
maxTransferCountOnMessageInDisk=8
accessMessageInMemoryMaxRatio=40
messageIndexEnable=true
maxHashSlotNum=5000000
maxIndexNum=20000000
maxMsgsNumBatch=64
messageIndexSafe=false
haSendHeartbeatInterval=5000
haHousekeepingInterval=20000
haTransferBatchSize=32768
haMasterAddress=
haSlaveFallbehindMax=268435456
brokerRole=SYNC_MASTER
flushDiskType=SYNC_FLUSH
syncFlushTimeout=5000
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
flushDelayOffsetInterval=10000
cleanFileForciblyEnable=true
warmMapedFileEnable=false
offsetCheckInSlave=false
debugLockEnable=false
duplicationEnable=false
diskFallRecorded=true
osPageCacheBusyTimeOutMills=1000
defaultQueryMaxNum=32
transientStorePoolEnable=false
transientStorePoolSize=5
fastFailIfNoBufferInStorePool=false
```

<mark style="color:red;">**192.168.0.40 -b-s**</mark>

```
vim broker-b-s.properties

rocketmqHome=/usr/local/rocketmq
# 如果是一个nameserver，直接写一个地址即可
namesrvAddr=192.168.0.40:9876;192.168.0.41:9876;192.168.0.42:9876
brokerName=borker-b
brokerClusterName=It-Pre-RocketmqCluster
brokerId=1
brokerPermission=6
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=8
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=false
clusterTopicEnable=true
brokerTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=false
sendMessageThreadPoolNums=1
pullMessageThreadPoolNums=20
queryMessageThreadPoolNums=10
adminBrokerThreadPoolNums=16
clientManageThreadPoolNums=32
consumerManageThreadPoolNums=32
heartbeatThreadPoolNums=2
endTransactionThreadPoolNums=12
flushConsumerOffsetInterval=5000
flushConsumerOffsetHistoryInterval=60000
rejectTransactionMessage=false
fetchNamesrvAddrByAddressServer=false
sendThreadPoolQueueCapacity=10000
pullThreadPoolQueueCapacity=100000
queryThreadPoolQueueCapacity=20000
clientManagerThreadPoolQueueCapacity=1000000
consumerManagerThreadPoolQueueCapacity=1000000
heartbeatThreadPoolQueueCapacity=50000
endTransactionPoolQueueCapacity=100000
filterServerNums=0
longPollingEnable=true
shortPollingTimeMills=1000
notifyConsumerIdsChangedEnable=true
highSpeedMode=false
commercialEnable=true
commercialTimerCount=1
commercialTransCount=1
commercialBigCount=1
commercialBaseCount=1
transferMsgByHeap=true
maxDelayTime=40
regionId=DefaultRegion
registerBrokerTimeoutMills=6000
slaveReadEnable=false
disableConsumeIfConsumerReadSlowly=false
consumerFallbehindThreshold=17179869184
brokerFastFailureEnable=true
waitTimeMillsInSendQueue=200
waitTimeMillsInPullQueue=5000
waitTimeMillsInHeartbeatQueue=31000
waitTimeMillsInTransactionQueue=3000
startAcceptSendRequestTimeStamp=0
traceOn=true
enableCalcFilterBitMap=false
expectConsumerNumUseFilter=32
maxErrorRateOfBloomFilter=20
filterDataCleanTimeSpan=86400000
filterSupportRetry=false
enablePropertyFilter=false
compressedRegister=false
forceRegister=true
registerNameServerPeriod=30000
transactionTimeOut=6000
transactionCheckMax=15
transactionCheckInterval=60000
#Broker 对外服务的监听端口
listenPort=10921
serverWorkerThreads=8
serverCallbackExecutorThreads=0
serverSelectorThreads=3
serverOnewaySemaphoreValue=256
serverAsyncSemaphoreValue=64
serverChannelMaxIdleTimeSeconds=120
serverSocketSndBufSize=131072
serverSocketRcvBufSize=131072
serverPooledByteBufAllocatorEnable=true
useEpollNativeSelector=false
clientWorkerThreads=16
clientCallbackExecutorThreads=2
clientOnewaySemaphoreValue=65535
clientAsyncSemaphoreValue=65535
connectTimeoutMillis=3000
channelNotActiveInterval=60000
clientChannelMaxIdleTimeSeconds=120
clientSocketSndBufSize=131072
clientSocketRcvBufSize=131072
clientPooledByteBufAllocatorEnable=false
clientCloseSocketIfTimeout=false
useTLS=false
storePathRootDir=/usr/local/rocketmq/store/rootdir-b-s
storePathCommitLog=/usr/local/rocketmq/store/commitlog-b-s
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue-b-s
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index-b-s
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint-b-s
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort-b-s
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存60W条，根据业务情况调整,
mapedFileSizeConsumeQueue=3000000
enableConsumeQueueExt=false
mappedFileSizeConsumeQueueExt=50331648
bitMapLengthConsumeQueueExt=64
flushIntervalCommitLog=500
commitIntervalCommitLog=200
useReentrantLockWhenPutMessage=false
flushCommitLogTimed=false
flushIntervalConsumeQueue=1000
cleanResourceInterval=10000
deleteCommitLogFilesInterval=100
deleteConsumeQueueFilesInterval=100
destroyMapedFileIntervalForcibly=120000
redeleteHangedFileInterval=120000
#删除文件时间点，默认凌晨 2点
deleteWhen=04
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=75
#文件保留时间，默认 72 小时
fileReservedTime=48
putMsgIndexHightWater=600000
#限制的消息大小
maxMessageSize=65536
checkCRCOnRecover=true
flushCommitLogLeastPages=4
commitCommitLogLeastPages=4
flushLeastPagesWhenWarmMapedFile=4096
flushConsumeQueueLeastPages=2
flushCommitLogThoroughInterval=10000
commitCommitLogThoroughInterval=200
flushConsumeQueueThoroughInterval=60000
maxTransferBytesOnMessageInMemory=262144
maxTransferCountOnMessageInMemory=32
maxTransferBytesOnMessageInDisk=65536
maxTransferCountOnMessageInDisk=8
accessMessageInMemoryMaxRatio=40
messageIndexEnable=true
maxHashSlotNum=5000000
maxIndexNum=20000000
maxMsgsNumBatch=64
messageIndexSafe=false
haSendHeartbeatInterval=5000
haHousekeepingInterval=20000
haTransferBatchSize=32768
haMasterAddress=
haSlaveFallbehindMax=268435456
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
syncFlushTimeout=5000
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
flushDelayOffsetInterval=10000
cleanFileForciblyEnable=true
warmMapedFileEnable=false
offsetCheckInSlave=false
debugLockEnable=false
duplicationEnable=false
diskFallRecorded=true
osPageCacheBusyTimeOutMills=1000
defaultQueryMaxNum=32
transientStorePoolEnable=false
transientStorePoolSize=5
fastFailIfNoBufferInStorePool=false
```

<mark style="color:red;">**192.168.0.41 -b**</mark>

```
vim /usr/local/rocketmq/conf/3m-3s-sync/broker-b.properties

rocketmqHome=/usr/local/rocketmq
# 如果是一个nameserver，直接写一个地址即可
namesrvAddr=192.168.0.40:9876;192.168.0.41:9876;192.168.0.42:9876
brokerName=borker-b
brokerClusterName=It-Pre-RocketmqCluster
brokerId=0
brokerPermission=6
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=8
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=false
clusterTopicEnable=true
brokerTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=false
sendMessageThreadPoolNums=1
pullMessageThreadPoolNums=20
queryMessageThreadPoolNums=10
adminBrokerThreadPoolNums=16
clientManageThreadPoolNums=32
consumerManageThreadPoolNums=32
heartbeatThreadPoolNums=2
endTransactionThreadPoolNums=12
flushConsumerOffsetInterval=5000
flushConsumerOffsetHistoryInterval=60000
rejectTransactionMessage=false
fetchNamesrvAddrByAddressServer=false
sendThreadPoolQueueCapacity=10000
pullThreadPoolQueueCapacity=100000
queryThreadPoolQueueCapacity=20000
clientManagerThreadPoolQueueCapacity=1000000
consumerManagerThreadPoolQueueCapacity=1000000
heartbeatThreadPoolQueueCapacity=50000
endTransactionPoolQueueCapacity=100000
filterServerNums=0
longPollingEnable=true
shortPollingTimeMills=1000
notifyConsumerIdsChangedEnable=true
highSpeedMode=false
commercialEnable=true
commercialTimerCount=1
commercialTransCount=1
commercialBigCount=1
commercialBaseCount=1
transferMsgByHeap=true
maxDelayTime=40
regionId=DefaultRegion
registerBrokerTimeoutMills=6000
slaveReadEnable=false
disableConsumeIfConsumerReadSlowly=false
consumerFallbehindThreshold=17179869184
brokerFastFailureEnable=true
waitTimeMillsInSendQueue=200
waitTimeMillsInPullQueue=5000
waitTimeMillsInHeartbeatQueue=31000
waitTimeMillsInTransactionQueue=3000
startAcceptSendRequestTimeStamp=0
traceOn=true
enableCalcFilterBitMap=false
expectConsumerNumUseFilter=32
maxErrorRateOfBloomFilter=20
filterDataCleanTimeSpan=86400000
filterSupportRetry=false
enablePropertyFilter=false
compressedRegister=false
forceRegister=true
registerNameServerPeriod=30000
transactionTimeOut=6000
transactionCheckMax=15
transactionCheckInterval=60000
#Broker 对外服务的监听端口
listenPort=10911
serverWorkerThreads=8
serverCallbackExecutorThreads=0
serverSelectorThreads=3
serverOnewaySemaphoreValue=256
serverAsyncSemaphoreValue=64
serverChannelMaxIdleTimeSeconds=120
serverSocketSndBufSize=131072
serverSocketRcvBufSize=131072
serverPooledByteBufAllocatorEnable=true
useEpollNativeSelector=false
clientWorkerThreads=16
clientCallbackExecutorThreads=2
clientOnewaySemaphoreValue=65535
clientAsyncSemaphoreValue=65535
connectTimeoutMillis=3000
channelNotActiveInterval=60000
clientChannelMaxIdleTimeSeconds=120
clientSocketSndBufSize=131072
clientSocketRcvBufSize=131072
clientPooledByteBufAllocatorEnable=false
clientCloseSocketIfTimeout=false
useTLS=false
storePathRootDir=/usr/local/rocketmq/store/rootdir-b-m
storePathCommitLog=/usr/local/rocketmq/store/commitlog-b-m
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue-b-m
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index-b-m
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint-b-m
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort-b-m
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存60W条，根据业务情况调整,
mapedFileSizeConsumeQueue=3000000
enableConsumeQueueExt=false
mappedFileSizeConsumeQueueExt=50331648
bitMapLengthConsumeQueueExt=64
flushIntervalCommitLog=500
commitIntervalCommitLog=200
useReentrantLockWhenPutMessage=false
flushCommitLogTimed=false
flushIntervalConsumeQueue=1000
cleanResourceInterval=10000
deleteCommitLogFilesInterval=100
deleteConsumeQueueFilesInterval=100
destroyMapedFileIntervalForcibly=120000
redeleteHangedFileInterval=120000
#删除文件时间点，默认凌晨 2点
deleteWhen=04
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=75
#文件保留时间，默认 72 小时
fileReservedTime=48
putMsgIndexHightWater=600000
#限制的消息大小
maxMessageSize=65536
checkCRCOnRecover=true
flushCommitLogLeastPages=4
commitCommitLogLeastPages=4
flushLeastPagesWhenWarmMapedFile=4096
flushConsumeQueueLeastPages=2
flushCommitLogThoroughInterval=10000
commitCommitLogThoroughInterval=200
flushConsumeQueueThoroughInterval=60000
maxTransferBytesOnMessageInMemory=262144
maxTransferCountOnMessageInMemory=32
maxTransferBytesOnMessageInDisk=65536
maxTransferCountOnMessageInDisk=8
accessMessageInMemoryMaxRatio=40
messageIndexEnable=true
maxHashSlotNum=5000000
maxIndexNum=20000000
maxMsgsNumBatch=64
messageIndexSafe=false
haSendHeartbeatInterval=5000
haHousekeepingInterval=20000
haTransferBatchSize=32768
haMasterAddress=
haSlaveFallbehindMax=268435456
brokerRole=SYNC_MASTER
flushDiskType=SYNC_FLUSH
syncFlushTimeout=5000
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
flushDelayOffsetInterval=10000
cleanFileForciblyEnable=true
warmMapedFileEnable=false
offsetCheckInSlave=false
debugLockEnable=false
duplicationEnable=false
diskFallRecorded=true
osPageCacheBusyTimeOutMills=1000
defaultQueryMaxNum=32
transientStorePoolEnable=false
transientStorePoolSize=5
fastFailIfNoBufferInStorePool=false
```

<mark style="color:red;">192.168.0.41 - c-s</mark>

```
vim /usr/local/rocketmq/conf/3m-3s-sync/broker-c-s.properties

rocketmqHome=/usr/local/rocketmq
# 如果是一个nameserver，直接写一个地址即可
namesrvAddr=192.168.0.40:9876;192.168.0.41:9876;192.168.0.42:9876
brokerName=borker-c
brokerClusterName=It3-Pre-RocketmqCluster
brokerId=1
brokerPermission=6
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=8
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=false
clusterTopicEnable=true
brokerTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=false
sendMessageThreadPoolNums=1
pullMessageThreadPoolNums=20
queryMessageThreadPoolNums=10
adminBrokerThreadPoolNums=16
clientManageThreadPoolNums=32
consumerManageThreadPoolNums=32
heartbeatThreadPoolNums=2
endTransactionThreadPoolNums=12
flushConsumerOffsetInterval=5000
flushConsumerOffsetHistoryInterval=60000
rejectTransactionMessage=false
fetchNamesrvAddrByAddressServer=false
sendThreadPoolQueueCapacity=10000
pullThreadPoolQueueCapacity=100000
queryThreadPoolQueueCapacity=20000
clientManagerThreadPoolQueueCapacity=1000000
consumerManagerThreadPoolQueueCapacity=1000000
heartbeatThreadPoolQueueCapacity=50000
endTransactionPoolQueueCapacity=100000
filterServerNums=0
longPollingEnable=true
shortPollingTimeMills=1000
notifyConsumerIdsChangedEnable=true
highSpeedMode=false
commercialEnable=true
commercialTimerCount=1
commercialTransCount=1
commercialBigCount=1
commercialBaseCount=1
transferMsgByHeap=true
maxDelayTime=40
regionId=DefaultRegion
registerBrokerTimeoutMills=6000
slaveReadEnable=false
disableConsumeIfConsumerReadSlowly=false
consumerFallbehindThreshold=17179869184
brokerFastFailureEnable=true
waitTimeMillsInSendQueue=200
waitTimeMillsInPullQueue=5000
waitTimeMillsInHeartbeatQueue=31000
waitTimeMillsInTransactionQueue=3000
startAcceptSendRequestTimeStamp=0
traceOn=true
enableCalcFilterBitMap=false
expectConsumerNumUseFilter=32
maxErrorRateOfBloomFilter=20
filterDataCleanTimeSpan=86400000
filterSupportRetry=false
enablePropertyFilter=false
compressedRegister=false
forceRegister=true
registerNameServerPeriod=30000
transactionTimeOut=6000
transactionCheckMax=15
transactionCheckInterval=60000
#Broker 对外服务的监听端口
listenPort=10921
serverWorkerThreads=8
serverCallbackExecutorThreads=0
serverSelectorThreads=3
serverOnewaySemaphoreValue=256
serverAsyncSemaphoreValue=64
serverChannelMaxIdleTimeSeconds=120
serverSocketSndBufSize=131072
serverSocketRcvBufSize=131072
serverPooledByteBufAllocatorEnable=true
useEpollNativeSelector=false
clientWorkerThreads=16
clientCallbackExecutorThreads=2
clientOnewaySemaphoreValue=65535
clientAsyncSemaphoreValue=65535
connectTimeoutMillis=3000
channelNotActiveInterval=60000
clientChannelMaxIdleTimeSeconds=120
clientSocketSndBufSize=131072
clientSocketRcvBufSize=131072
clientPooledByteBufAllocatorEnable=false
clientCloseSocketIfTimeout=false
useTLS=false
storePathRootDir=/usr/local/rocketmq/store/rootdir-c-s
storePathCommitLog=/usr/local/rocketmq/store/commitlog-c-s
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue-c-s
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index-c-s
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint-c-s
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort-c-s
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存60W条，根据业务情况调整,
mapedFileSizeConsumeQueue=3000000
enableConsumeQueueExt=false
mappedFileSizeConsumeQueueExt=50331648
bitMapLengthConsumeQueueExt=64
flushIntervalCommitLog=500
commitIntervalCommitLog=200
useReentrantLockWhenPutMessage=false
flushCommitLogTimed=false
flushIntervalConsumeQueue=1000
cleanResourceInterval=10000
deleteCommitLogFilesInterval=100
deleteConsumeQueueFilesInterval=100
destroyMapedFileIntervalForcibly=120000
redeleteHangedFileInterval=120000
#删除文件时间点，默认凌晨 2点
deleteWhen=04
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=75
#文件保留时间，默认 72 小时
fileReservedTime=48
putMsgIndexHightWater=600000
#限制的消息大小
maxMessageSize=65536
checkCRCOnRecover=true
flushCommitLogLeastPages=4
commitCommitLogLeastPages=4
flushLeastPagesWhenWarmMapedFile=4096
flushConsumeQueueLeastPages=2
flushCommitLogThoroughInterval=10000
commitCommitLogThoroughInterval=200
flushConsumeQueueThoroughInterval=60000
maxTransferBytesOnMessageInMemory=262144
maxTransferCountOnMessageInMemory=32
maxTransferBytesOnMessageInDisk=65536
maxTransferCountOnMessageInDisk=8
accessMessageInMemoryMaxRatio=40
messageIndexEnable=true
maxHashSlotNum=5000000
maxIndexNum=20000000
maxMsgsNumBatch=64
messageIndexSafe=false
haSendHeartbeatInterval=5000
haHousekeepingInterval=20000
haTransferBatchSize=32768
haMasterAddress=
haSlaveFallbehindMax=268435456
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
syncFlushTimeout=5000
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
flushDelayOffsetInterval=10000
cleanFileForciblyEnable=true
warmMapedFileEnable=false
offsetCheckInSlave=false
debugLockEnable=false
duplicationEnable=false
diskFallRecorded=true
osPageCacheBusyTimeOutMills=1000
defaultQueryMaxNum=32
transientStorePoolEnable=false
transientStorePoolSize=5
fastFailIfNoBufferInStorePool=false
```

<mark style="color:red;">**192.168.0.42 - c**</mark>

```
vim /usr/local/rocketmq/conf/3m-3s-sync/broker-c.properties

rocketmqHome=/usr/local/rocketmq
# 如果是一个nameserver，直接写一个地址即可
namesrvAddr=192.168.18.64:9876;192.168.18.65:9876;192.168.18.66:9876
brokerName=borker-c
brokerClusterName=It3-Pre-RocketmqCluster
brokerId=0
brokerPermission=6
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=8
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=false
clusterTopicEnable=true
brokerTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=false
sendMessageThreadPoolNums=1
pullMessageThreadPoolNums=20
queryMessageThreadPoolNums=10
adminBrokerThreadPoolNums=16
clientManageThreadPoolNums=32
consumerManageThreadPoolNums=32
heartbeatThreadPoolNums=2
endTransactionThreadPoolNums=12
flushConsumerOffsetInterval=5000
flushConsumerOffsetHistoryInterval=60000
rejectTransactionMessage=false
fetchNamesrvAddrByAddressServer=false
sendThreadPoolQueueCapacity=10000
pullThreadPoolQueueCapacity=100000
queryThreadPoolQueueCapacity=20000
clientManagerThreadPoolQueueCapacity=1000000
consumerManagerThreadPoolQueueCapacity=1000000
heartbeatThreadPoolQueueCapacity=50000
endTransactionPoolQueueCapacity=100000
filterServerNums=0
longPollingEnable=true
shortPollingTimeMills=1000
notifyConsumerIdsChangedEnable=true
highSpeedMode=false
commercialEnable=true
commercialTimerCount=1
commercialTransCount=1
commercialBigCount=1
commercialBaseCount=1
transferMsgByHeap=true
maxDelayTime=40
regionId=DefaultRegion
registerBrokerTimeoutMills=6000
slaveReadEnable=false
disableConsumeIfConsumerReadSlowly=false
consumerFallbehindThreshold=17179869184
brokerFastFailureEnable=true
waitTimeMillsInSendQueue=200
waitTimeMillsInPullQueue=5000
waitTimeMillsInHeartbeatQueue=31000
waitTimeMillsInTransactionQueue=3000
startAcceptSendRequestTimeStamp=0
traceOn=true
enableCalcFilterBitMap=false
expectConsumerNumUseFilter=32
maxErrorRateOfBloomFilter=20
filterDataCleanTimeSpan=86400000
filterSupportRetry=false
enablePropertyFilter=false
compressedRegister=false
forceRegister=true
registerNameServerPeriod=30000
transactionTimeOut=6000
transactionCheckMax=15
transactionCheckInterval=60000
#Broker 对外服务的监听端口
listenPort=10911
serverWorkerThreads=8
serverCallbackExecutorThreads=0
serverSelectorThreads=3
serverOnewaySemaphoreValue=256
serverAsyncSemaphoreValue=64
serverChannelMaxIdleTimeSeconds=120
serverSocketSndBufSize=131072
serverSocketRcvBufSize=131072
serverPooledByteBufAllocatorEnable=true
useEpollNativeSelector=false
clientWorkerThreads=16
clientCallbackExecutorThreads=2
clientOnewaySemaphoreValue=65535
clientAsyncSemaphoreValue=65535
connectTimeoutMillis=3000
channelNotActiveInterval=60000
clientChannelMaxIdleTimeSeconds=120
clientSocketSndBufSize=131072
clientSocketRcvBufSize=131072
clientPooledByteBufAllocatorEnable=false
clientCloseSocketIfTimeout=false
useTLS=false
storePathRootDir=/usr/local/rocketmq/store/rootdir-c-m
storePathCommitLog=/usr/local/rocketmq/store/commitlog-c-m
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue-c-m
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index-c-m
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint-c-m
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort-c-m
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存60W条，根据业务情况调整,
mapedFileSizeConsumeQueue=3000000
enableConsumeQueueExt=false
mappedFileSizeConsumeQueueExt=50331648
bitMapLengthConsumeQueueExt=64
flushIntervalCommitLog=500
commitIntervalCommitLog=200
useReentrantLockWhenPutMessage=false
flushCommitLogTimed=false
flushIntervalConsumeQueue=1000
cleanResourceInterval=10000
deleteCommitLogFilesInterval=100
deleteConsumeQueueFilesInterval=100
destroyMapedFileIntervalForcibly=120000
redeleteHangedFileInterval=120000
#删除文件时间点，默认凌晨 2点
deleteWhen=04
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=75
#文件保留时间，默认 72 小时
fileReservedTime=48
putMsgIndexHightWater=600000
#限制的消息大小
maxMessageSize=65536
checkCRCOnRecover=true
flushCommitLogLeastPages=4
commitCommitLogLeastPages=4
flushLeastPagesWhenWarmMapedFile=4096
flushConsumeQueueLeastPages=2
flushCommitLogThoroughInterval=10000
commitCommitLogThoroughInterval=200
flushConsumeQueueThoroughInterval=60000
maxTransferBytesOnMessageInMemory=262144
maxTransferCountOnMessageInMemory=32
maxTransferBytesOnMessageInDisk=65536
maxTransferCountOnMessageInDisk=8
accessMessageInMemoryMaxRatio=40
messageIndexEnable=true
maxHashSlotNum=5000000
maxIndexNum=20000000
maxMsgsNumBatch=64
messageIndexSafe=false
haSendHeartbeatInterval=5000
haHousekeepingInterval=20000
haTransferBatchSize=32768
haMasterAddress=
haSlaveFallbehindMax=268435456
brokerRole=SYNC_MASTER
flushDiskType=SYNC_FLUSH
syncFlushTimeout=5000
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
flushDelayOffsetInterval=10000
cleanFileForciblyEnable=true
warmMapedFileEnable=false
offsetCheckInSlave=false
debugLockEnable=false
duplicationEnable=false
diskFallRecorded=true
osPageCacheBusyTimeOutMills=1000
defaultQueryMaxNum=32
transientStorePoolEnable=false
transientStorePoolSize=5
fastFailIfNoBufferInStorePool=false
```

<mark style="color:red;">**192.168.0.42 - a-s**</mark>

```
vim /usr/local/rocketmq/conf/3m-3s-sync/broker-a-s.properties

rocketmqHome=/usr/local/rocketmq
# 如果是一个nameserver，直接写一个地址即可
namesrvAddr=192.168.18.64:9876;192.168.18.65:9876;192.168.18.66:9876
brokerName=borker-a
brokerClusterName=It3-Pre-RocketmqCluster
brokerId=1
brokerPermission=6
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=8
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=false
clusterTopicEnable=true
brokerTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=false
sendMessageThreadPoolNums=1
pullMessageThreadPoolNums=20
queryMessageThreadPoolNums=10
adminBrokerThreadPoolNums=16
clientManageThreadPoolNums=32
consumerManageThreadPoolNums=32
heartbeatThreadPoolNums=2
endTransactionThreadPoolNums=12
flushConsumerOffsetInterval=5000
flushConsumerOffsetHistoryInterval=60000
rejectTransactionMessage=false
fetchNamesrvAddrByAddressServer=false
sendThreadPoolQueueCapacity=10000
pullThreadPoolQueueCapacity=100000
queryThreadPoolQueueCapacity=20000
clientManagerThreadPoolQueueCapacity=1000000
consumerManagerThreadPoolQueueCapacity=1000000
heartbeatThreadPoolQueueCapacity=50000
endTransactionPoolQueueCapacity=100000
filterServerNums=0
longPollingEnable=true
shortPollingTimeMills=1000
notifyConsumerIdsChangedEnable=true
highSpeedMode=false
commercialEnable=true
commercialTimerCount=1
commercialTransCount=1
commercialBigCount=1
commercialBaseCount=1
transferMsgByHeap=true
maxDelayTime=40
regionId=DefaultRegion
registerBrokerTimeoutMills=6000
slaveReadEnable=false
disableConsumeIfConsumerReadSlowly=false
consumerFallbehindThreshold=17179869184
brokerFastFailureEnable=true
waitTimeMillsInSendQueue=200
waitTimeMillsInPullQueue=5000
waitTimeMillsInHeartbeatQueue=31000
waitTimeMillsInTransactionQueue=3000
startAcceptSendRequestTimeStamp=0
traceOn=true
enableCalcFilterBitMap=false
expectConsumerNumUseFilter=32
maxErrorRateOfBloomFilter=20
filterDataCleanTimeSpan=86400000
filterSupportRetry=false
enablePropertyFilter=false
compressedRegister=false
forceRegister=true
registerNameServerPeriod=30000
transactionTimeOut=6000
transactionCheckMax=15
transactionCheckInterval=60000
#Broker 对外服务的监听端口
listenPort=10921
serverWorkerThreads=8
serverCallbackExecutorThreads=0
serverSelectorThreads=3
serverOnewaySemaphoreValue=256
serverAsyncSemaphoreValue=64
serverChannelMaxIdleTimeSeconds=120
serverSocketSndBufSize=131072
serverSocketRcvBufSize=131072
serverPooledByteBufAllocatorEnable=true
useEpollNativeSelector=false
clientWorkerThreads=16
clientCallbackExecutorThreads=2
clientOnewaySemaphoreValue=65535
clientAsyncSemaphoreValue=65535
connectTimeoutMillis=3000
channelNotActiveInterval=60000
clientChannelMaxIdleTimeSeconds=120
clientSocketSndBufSize=131072
clientSocketRcvBufSize=131072
clientPooledByteBufAllocatorEnable=false
clientCloseSocketIfTimeout=false
useTLS=false
storePathRootDir=/usr/local/rocketmq/store/rootdir-a-s
storePathCommitLog=/usr/local/rocketmq/store/commitlog-a-s
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue-a-s
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index-a-s
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint-a-s
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort-a-s
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存60W条，根据业务情况调整,
mapedFileSizeConsumeQueue=3000000
enableConsumeQueueExt=false
mappedFileSizeConsumeQueueExt=50331648
bitMapLengthConsumeQueueExt=64
flushIntervalCommitLog=500
commitIntervalCommitLog=200
useReentrantLockWhenPutMessage=false
flushCommitLogTimed=false
flushIntervalConsumeQueue=1000
cleanResourceInterval=10000
deleteCommitLogFilesInterval=100
deleteConsumeQueueFilesInterval=100
destroyMapedFileIntervalForcibly=120000
redeleteHangedFileInterval=120000
#删除文件时间点，默认凌晨 2点
deleteWhen=04
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=75
#文件保留时间，默认 72 小时
fileReservedTime=48
putMsgIndexHightWater=600000
#限制的消息大小
maxMessageSize=65536
checkCRCOnRecover=true
flushCommitLogLeastPages=4
commitCommitLogLeastPages=4
flushLeastPagesWhenWarmMapedFile=4096
flushConsumeQueueLeastPages=2
flushCommitLogThoroughInterval=10000
commitCommitLogThoroughInterval=200
flushConsumeQueueThoroughInterval=60000
maxTransferBytesOnMessageInMemory=262144
maxTransferCountOnMessageInMemory=32
maxTransferBytesOnMessageInDisk=65536
maxTransferCountOnMessageInDisk=8
accessMessageInMemoryMaxRatio=40
messageIndexEnable=true
maxHashSlotNum=5000000
maxIndexNum=20000000
maxMsgsNumBatch=64
messageIndexSafe=false
haSendHeartbeatInterval=5000
haHousekeepingInterval=20000
haTransferBatchSize=32768
haMasterAddress=
haSlaveFallbehindMax=268435456
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
syncFlushTimeout=5000
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
flushDelayOffsetInterval=10000
cleanFileForciblyEnable=true
warmMapedFileEnable=false
offsetCheckInSlave=false
debugLockEnable=false
duplicationEnable=false
diskFallRecorded=true
osPageCacheBusyTimeOutMills=1000
defaultQueryMaxNum=32
transientStorePoolEnable=false
transientStorePoolSize=5
fastFailIfNoBufferInStorePool=false
```

> <mark style="color:red;">上面两个配置文件一定要注意：两个broker的存储路径不要一样</mark>

> 修改默认broker和nameserver的默认内存配置(实例为 64G服务器)
>
> 根据自己生产机器配置大小而修改

```
vim /usr/local/rocketmq/bin/runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms14g -Xmx14g -Xmn5g"

vim /usr/local/rocketmq/bin/runserver.sh
JAVA_OPT="${JAVA_OPT} -server -Xms7g -Xmx7g -Xmn2g -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=1024m"
```

#### 服务器ABC的broker配置注意

1.注意每个主broker和它的从broker的brokerName需要一致。

2.注意主broker的brokerId=0，从broker的brokerId=1，0表示Master，大于0表示 Slave，不然broker启动会因冲突而失败

#### 启动集群

启动namesrv（每台都要启动）

```
cd /usr/local/rocketmq/bin/
nohup sh mqnamesrv &
tail -f ~/logs/rocketmqlogs/namesrv.log
看到The Name Server boot success...即启动成功
```

启动broker（每台都要启动，此处以服务器A为例

```
nohup sh mqbroker -c /usr/local/rocketmq/conf/3m-3s-async/broker-a.properties &
nohup sh mqbroker -c /usr/local/rocketmq/conf/3m-3s-async/broker-b-s.properties &

# 验证Name Server 是否启动成功
tail -f ~/logs/rocketmqlogs/broker.log
```

查看集群

```
sh mqadmin clusterlist -n 192.168.0.40:9876

#停止
sh mqshutdown broker
sh mqshutdown namesrv
```

### 安装控制台

```
#下载

wget https://github.com/apache/rocketmq-externals/archive/master.zip
unzip master.zip

#编译打包
cd /opt/master
mvn clean package -Dmaven.test.skip=true
bash复制代码
三、启动
mkdir /usr/local/rocketmq-console/
cp target/rocketmq-console-ng-2.0.0.jar /usr/local/rocketmq-console/
cd /usr/local/rocketmq-console/
nohup java -jar rocketmq-console-ng-2.0.0.jar --rocketmq.config.namesrvAddr=192.168.0.40:9876;192.168.0.41:9876;192.168.0.42:9876 &
```
