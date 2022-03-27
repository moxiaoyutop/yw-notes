# prometheus部署

```
#下载所需要的组件
wget https://github.com/prometheus/prometheus/releases/download/v2.32.0/prometheus-2.32.0.linux-amd64.tar.gz
wget https://github.com/prometheus/alertmanager/releases/download/v0.23.0/alertmanager-0.23.0.linux-amd64.tar.gz
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.19.0/blackbox_exporter-0.19.0.linux-amd64.tar.gz

#解压
tar xf prometheus-2.32.0.linux-amd64.tar.gz -C /usr/local/
tar xf alertmanager-0.23.0.linux-amd64.tar.gz -C /usr/local/
tar xf blackbox_exporter-0.19.0.linux-amd64.tar.gz -C /usr/local/
mv /usr/local/prometheus-2.32.0.linux-amd64/ /usr/local/prometheus
mv /usr/local/alertmanager-0.23.0.linux-amd64/ /usr/local/alertmanager
mv /usr/local/blackbox_exporter-0.19.0.linux-amd64/ /usr/local/blackbox
```

### 配置prometheus

```
#编辑配置文件
vi /usr/local/prometheus/prometheus.yml

# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ["localhost:9093"]

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "/usr/local/alertmanager/rules/*.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets:
        - "192.168.0.13:9100"
        - "192.168.0.14:9100"
        - "192.168.0.15:9100"
        - "192.168.0.16:9100"

  - job_name: 'redis-node'
    static_configs:
      - targets:
        - "192.168.0.67:9100"
        - "192.168.0.68:9100"
        - "192.168.0.69:9100"
        - "192.168.0.10:9100"
        - "192.168.0.11:9100"
        - "192.168.0.12:9100"

  - job_name: 'nginx-node'
    static_configs:
      - targets:
        - "192.168.0.72:9100"
        - "192.168.0.73:9100"
        - "192.168.0.150:9100"
        - "192.168.0.151:9100"

  - job_name: 'es-node'
    static_configs:
      - targets:
        - "192.168.0.80:9100"
        - "192.168.0.81:9100"
        - "192.168.0.82:9100"
        - "192.168.0.83:9100"
        - "192.168.0.84:9100"

  - job_name: 'tidb-node'
    static_configs:
      - targets:
        - "192.168.0.74:9100"
        - "192.168.0.75:9100"
        - "192.168.0.76:9100"
        - "192.168.0.159:9100"
        - "192.168.0.160:9100"
        - "192.168.0.162:9100"
        - "192.168.0.163:9100"

  - job_name: "icmp_ping"
    metrics_path: /probe
    params:
      module: [icmp]
    file_sd_configs:
    - refresh_interval: 10s
      files:
      - "/usr/local/blackbox_exporter/data/ping*.yml"
    relabel_configs:
    - source_labels: [__address__]
      regex: (.*)(:80)?
      target_label: __param_target
      replacement: ${1}
    - source_labels: [__param_target]
      target_label: instance
    - source_labels: [__param_target]
      regex: (.*)
      target_label: ping
      replacement: ${1}
    - source_labels: []
      regex: .*
      target_label: __address__
      replacement: 192.168.0.71:9115

  - job_name: 'redis_exporter_targets'
    static_configs:
      - targets:
        - redis://192.168.0.67:7000
        - redis://192.168.0.12:7011
        - redis://192.168.0.68:7002
        - redis://192.168.0.67:7001
        - redis://192.168.0.68:7003
        - redis://192.168.0.69:7004
        - redis://192.168.0.10:7006
        - redis://192.168.0.69:7005
        - redis://192.168.0.10:7007
        - redis://192.168.0.11:7008
        - redis://192.168.0.12:7010
        - redis://192.168.0.11:7009
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.0.67:9121

  - job_name: 'elasticsearch'
    static_configs:
      - targets: ['192.168.0.84:9114']

  - job_name: 'rocketmq'
    static_configs:
      - targets: ['192.168.0.65:5557']

  - job_name: 'nginx'
    static_configs:
    - targets: ['192.168.0.72:9913']
    - targets: ['192.168.0.73:9913']
    - targets: ['192.168.0.150:9913']
    - targets: ['192.168.0.151:9913']

  - job_name: "check_get"
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    file_sd_configs:
    - refresh_interval: 1m
      files:
      - "/usr/local/blackbox_exporter/data/service_get.yml"
    relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: 192.168.0.71:9115
```

### 配置blackbox

```
cat /usr/local/blackbox_exporter/data/ping_status.yml
- targets: ['192.168.0.10','192.168.0.11','192.168.0.12']
  labels:
    group: '机房 正式 网络监控'

cat /usr/local/blackbox_exporter/data/service_get.yml
- targets:
  - https://baidu.com
  - https://google.com
  labels:
    group: 'service'
```

### 配置alertmanager

```
cd /usr/local/alertmanager 
mkdir {data,rules}

vim /usr/local/alertmanager/rules/os-linux.yml

groups:
- name: 主机状态-监控告警
  rules:
  - alert: 主机状态
    expr: up == 0
    for: 15s
    labels:
      status: 非常严重
    annotations:
      summary: "{{$labels.instance}}:服务器宕机"
      description: "{{$labels.instance}}:服务器延时超过5分钟"

  - alert: CPU使用情况
    expr: 100-(avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) by(instance)* 100) > 80
    for: 15s
    labels:
      status: 一般告警
    annotations:
      summary: "{{$labels.mountpoint}} CPU使用率过高！"
      description: "{{$labels.mountpoint }} CPU使用大于80%(目前使用:{{$value}}%)"

  - alert: 内存使用
    expr: (1 - (node_memory_MemAvailable_bytes / (node_memory_MemTotal_bytes)))* 100 > 90
    for: 15s
    labels:
      status: 严重告警
    annotations:
      summary: "{{$labels.mountpoint}} 内存使用率过高！"
      description: "{{$labels.mountpoint }} 内存使用大于90%(已使用:{{$value}}%)"

  - alert: IO性能
    #expr: 100-(avg(irate(node_disk_io_time_seconds_total[1m])) by(instance)* 100) < 60
    expr: (avg(irate(node_disk_io_time_seconds_total[1m])) by(instance) * 100) > 80
    for: 15s
    labels:
      status: 严重告警
    annotations:
      summary: "{{$labels.mountpoint}} 流入磁盘IO使用率过高！"
      description: "{{$labels.mountpoint }} 流入磁盘IO大于80%(目前使用:{{$value}})"

  - alert: 网络入口
    expr: sum by (instance) (irate(node_network_receive_bytes_total{device=~"em1",instance=~'192.168.18.72:9100|192.168.18.73:9100|192.168.18.150:9100|192.168.18.151:9100' }[1m]) / 100000) > 20
    for: 15s
    labels:
      status: 严重告警
    annotations:
      summary: "{{$labels.mountpoint}} 流入网络带宽过高！"
      description: "{{$labels.mountpoint }}流入网络带宽持续15秒高于10M. 带宽使用率{{$value}}"

  - alert: 网络出口
    expr: sum by (instance) (irate(node_network_transmit_bytes_total{device=~"em1",instance=~'192.168.18.72:9100|192.168.18.73:9100|192.168.18.150:9100|192.168.18.151:9100' }[1m]) / 100000) > 20
    for: 15s
    labels:
      status: 严重告警
    annotations:
      summary: "{{$labels.mountpoint}} 流出网络带宽过高！"
      description: "{{$labels.mountpoint }}流出网络带宽持续15秒高于20M. 带宽使用率{{$value}}"

  - alert: TCP会话
    expr: node_netstat_Tcp_CurrEstab > 30000
    for: 15s
    labels:
      status: 严重告警
    annotations:
      summary: "{{$labels.mountpoint}} TCP_ESTABLISHED过高！"
      description: "{{$labels.mountpoint }} TCP_ESTABLISHED大于3000%(目前使用:{{$value}}%)"

  - alert: 磁盘容量
    expr: 100-(node_filesystem_free_bytes{fstype=~"ext4|xfs"}/node_filesystem_size_bytes {fstype=~"ext4|xfs"}*100) > 80
    for: 15s
    labels:
      status: 严重告警
    annotations:
      summary: "{{$labels.mountpoint}} 磁盘分区使用率过高！"
      description: "{{$labels.mountpoint }} 磁盘分区使用大于80%(目前使用:{{$value}}%)"

  - alert: ping主机
    expr: probe_success == 0
    for: 15s
    labels:
      status: 严重告警
    annotations:
      summary: "{{$labels.mountpoint}} 主机ping不通了！"
      description: "{{$labels.mountpoint }} 非要逼我报警么{{$value}}%)"
```

### 配置各服务启动脚本

```
# 配置blackbox 启动脚本
vim /usr/lib/systemd/system/blackbox.service
[Unit]
Description=Prometheus
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/usr/local/blackbox
ExecStart=/usr/local/blackbox/blackbox_exporter --config.file=/usr/local/blackbox/blackbox.yml

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

#启动
systemctl start blackbox.service
systemctl status blackbox.service


#prometheus启动脚本
vim /usr/lib/systemd/system/prometheus.service
[Unit]
Description=Prometheus
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/usr/local/prometheus
ExecStart=/usr/local/prometheus/prometheus --config.file=/usr/local/prometheus/prometheus.yml

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

systemctl start prometheus.service
systemctl status prometheus.service


#配置alertmanager启动脚本
[Unit]
Description=alertmanager
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/usr/local/alertmanager
ExecStart=/usr/local/alertmanager/alertmanager --config.file=/usr/local/alertmanager/alertmanager.yml

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

systemctl start alertmanager.service
systemctl status alertmanager.service
```

### 安装granfana

```
cat >> /etc/yum.repos.d/grafana.repo << EOF
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF

额外安装依赖
yum install fontconfig freetype* urw-fonts -y
yum install grafana -y
systemctl start grafana-server
systemctl enable grafana-server

常用代码
8919
9965
2949
10990
10477
```

### 监控端node部署

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar xf node_exporter-1.3.1.linux-amd64.tar.gz -C /usr/local/
mv /usr/local/node_exporter-1.3.1.linux-amd64 /usr/local/node_exporter

vim /usr/lib/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/node_exporter/node_exporter

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

chmod +x /usr/lib/systemd/system/node_exporter.service
systemctl start node_exporter.service
systemctl enable node_exporter.service
systemctl status node_exporter.service
```

### 配置telegram告警

```
# 安装python3环境
yum install python36 -y
cd /usr/local/alertmanager
pip3 install packages.txt
python3 bot.py &

vim /usr/local/alertmanager/bot.py

import datetime
import telegram, json, logging
from dateutil import parser
from flask import Flask
from flask import request
from flask_basicauth import BasicAuth

app = Flask(__name__)
app.secret_key = 'jkprometheus_bot'
basic_auth = BasicAuth(app)

# Yes need to have -, change it!
chatID = "-570167"

# Authentication conf, change it!
app.config['BASIC_AUTH_FORCE'] = False
app.config['BASIC_AUTH_USERNAME'] = 'XXXUSERNAME'
app.config['BASIC_AUTH_PASSWORD'] = 'XXXPASSWORD'

# Bot token, change it!
bot = telegram.Bot(token="169:AAHQCpDdUSZsm-P4iJm6gA")


@app.route('/alert', methods=['POST'])
def postAlertmanager():

    try:
        content = json.loads(request.get_data())
        print(content)
        for alert in content['alerts']:
            message = "状态情况: "+alert['status']+"\n"
            if 'name' in alert['labels']:
                message += "故障主机："+alert['labels']['instance']+"("+alert['labels']['name']+")\n"
            else:
                message += "故障主机："+alert['labels']['instance']+"\n"
            if 'info' in alert['annotations']:
                message += "Info: "+alert['annotations']['info']+"\n"
            if 'summary' in alert['annotations']:
                message += "故障标题："+alert['annotations']['summary']+"\n"
            if 'description' in alert['annotations']:
                message += "故障事件："+alert['annotations']['description']+"\n"
            if alert['status'] == "resolved":
                correctDate = (parser.parse(alert['endsAt'])+datetime.timedelta(hours=8)).strftime('%Y-%m-%d %H:%M:%S')
                message += "恢复时间："+correctDate
            elif alert['status'] == "firing":
                correctDate = (parser.parse(alert['startsAt'])+datetime.timedelta(hours=8)).strftime('%Y-%m-%d %H:%M:%S')
                message += "故障时间："+correctDate
            bot.sendMessage(chat_id=chatID, text=message)
            return "Alert OK", 200
    except Exception as error:
        bot.sendMessage(chat_id=chatID, text="Error to read json: "+str(error))
        app.logger.info("\t%s", error)
        return "Alert fail", 200


if __name__ == '__main__':
    logging.basicConfig(level=logging.INFO)
    app.run(host='0.0.0.0', port=9119)


```

![](<../../.gitbook/assets/image (14).png>)

> 完了后再修改一下

```
vim /usr/local/alertmanager/alertmanager.yml

global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:9119/alert'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```
