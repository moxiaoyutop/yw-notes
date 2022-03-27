# Prometheus监控redis

> 一、安装reids\_exporter

```
wget https://github.com/oliver006/redis_exporter/releases/download/v1.11.1/redis_exporter-v1.11.1.linux-amd64.tar.gz
tar -zxvf redis_exporter-v1.11.1.linux-amd64.tar.gz
# 这里只指定了集群中的一台redis， 它会自动收集Cluster 中所有redis实例的。
# 启动后端口为9121
nohup ./redis_exporter -redis.addr 192.168.0.10:7000 &
```

> 二、修改prometheus

```
vi prometheus.yml

- job_name: 'redis_exporter_targets'
    static_configs:
      - targets:
        - redis://192.168.0.10:7000
        - redis://192.168.0.10:7001
        - redis://192.168.0.10:7002
        - redis://192.168.0.11:7000
        - redis://192.168.0.11:7001
        - redis://192.168.0.11:7002
        - redis://192.168.0.12:7000
        - redis://192.168.0.12:7001
        - redis://192.168.0.12:7002
    params:
      check-keys: ["metrics:*"]
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.0.10:9121
  - job_name: 'redis_exporter'
    static_configs:
    - targets:
      - 192.168.0.10:9121

# 192.168.0.10:9121 为 redis_export 的地址， 为了收集redis集群健康状况。
# redis:// 为 redis集群redis地址。


grafana的redis代码
69602
```

> 配置告警信息

```
cd /usr/local/alertmanager/rules/
vim redis.yml

groups:
- name: redisAlert
  rules:
    - alert: Redis挂了
    expr: redis_up  == 0
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Redis down (instance {{ $labels.instance }})"
      description: "Redis 挂了啊，mmp\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: OutOfMemory
    expr: redis_memory_used_bytes / redis_memory_max_bytes * 100 > 80
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "Out of memory (instance {{ $labels.instance }})"
      description: "Redis is running out of memory (> 80%)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: 复制中断
    expr: delta(redis_connected_slaves[1m]) < 0
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "复制中断 Replication broken (instance {{ $labels.instance }})"
      description: "Redis instance lost a slave\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: TooManyConnections
    expr: redis_connected_clients > 5500
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "Too many connections (instance {{ $labels.instance }})"
      description: "Redis instance has too many connections( >5500 )\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: NotEnoughConnections
    expr: redis_connected_clients < 5
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "Not enough connections (instance {{ $labels.instance }})"
      description: "Redis instance should have more connections (> 5)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: 拒绝的连接数
    expr: increase(redis_rejected_connections_total[1m]) > 0
    for: 5m
    labels:
      severity: error
    annotations:
      summary: "Rejected connections (instance {{ $labels.instance }})"
      description: "Some connections to Redis has been rejected\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: redis节点堵塞
    expr: redis_memory_used_bytes < 1024
    for: 3s
    labels:
      severity: error
    annotations:
      summary: "redis 集群节点 (instance {{ $labels.instance }}) 堵塞"
      description: "redis 集群节点内存使用值为\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

