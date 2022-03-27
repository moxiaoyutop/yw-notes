# 单机部署

一、下载解压

```
wget https://dlcdn.apache.org/rocketmq/4.9.2/rocketmq-all-4.9.2-bin-release.zip

unzip rocketmq-all-4.7.1-bin-release.zip -d /usr/local/

mv rocketmq-all-4.7.1-bin-release/ rocketmq-all-4.7.1
```

二、修改配置

```
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn125m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
bash
修改tools.sh、runserver.sh、runbroker.sh 中JAVA启动内存参数，默认的数值比较大，可能会启动不起来。

cat >> /usr/local/rocketmq-all-4.7.1/conf/broker.conf << EOF
# 配置当前broker监听的地址（一般是公网IP），防止阿里云ECS无法公网访问MQ
brokerIP1=xx.xx.xx.xx
# 配置namesrv地址
namesrvAddr=127.0.0.1:9876
EOF
broker启动默认会取第一个网卡的IP作为监听IP，一般为内网IP。
```

三、启动

```
# 启动namesvr，监听9876端口
nohup sh bin/mqnamesrv &
# 启动mqbroker，监听10911、10912端口
nohup sh bin/mqbroker -n localhost:9876 -c ./conf/broker.conf autoCreateTopicEnable=true &
```

四、验证

```
export NAMESRV_ADDR=localhost:9876
# 生产者
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
# 消费者
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
```
