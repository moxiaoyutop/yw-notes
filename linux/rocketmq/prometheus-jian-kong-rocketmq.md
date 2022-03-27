# prometheus监控rocketmq

> &#x20;一、下载及安装

```
git clone https://github.com/apache/rocketmq-exporter
cd rocketmq-exporter
mvn clean install

java -jar rocketmq-exporter-0.0.1-SNAPSHOT.jar --rocketmq.config.namesrvAddr="127.0.0.1:9876"
```

> prometheus配置文件配置

```
- job_name: 'rocketmq'
    static_configs:
      - targets: ['192.168.0.65:5557']
```

> 告警配置

```
cat /usr/local/alertmanager/rules/mq.yml

groups:
- name: GaleraAlerts
  rules:
  - alert: RocketMQClusterProduceHigh
    expr: sum(rocketmq_producer_tps) by (instance,cluster) >= 300
    for: 1m
    labels:
      severity: warning
    annotations:
      description: '集群 {{ $labels.cluster }} 发送的总消息数量(tps) 太高 {{$value}} (大于 300)'
      summary: cluster send tps too high
  - alert: RocketMQClusterProduceLow
    expr: sum(rocketmq_producer_tps) by (instance,cluster) < 1
    for: 1m
    labels:
      severity: warning
    annotations:
      description: '集群 {{ $labels.cluster }} 发送的总消息数量(tps) 太少 {{$value}} (小于 1)'
      summary: cluster send tps too low
  - alert: RocketMQClusterConsumeHigh
    expr: sum(rocketmq_consumer_tps) by (instance,cluster) >= 500
    for: 1m
    labels:
      severity: warning
    annotations:
      description: '集群 {{ $labels.cluster }} 消费的总消息数量(tps) 太高 {{$value}} (大于 500)'
      summary: cluster consume tps too high
  - alert: RocketMQClusterConsumeLow
    expr: sum(rocketmq_consumer_tps) by (instance,cluster) < 1
    for: 1m
    labels:
      severity: warning
    annotations:
      description: '集群 {{ $labels.cluster }} 消费的总消息数量(tps) 太少 {{$value}} (小于 1)'
      summary: cluster consume tps too low
```
