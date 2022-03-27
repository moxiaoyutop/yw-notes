# EFLK-nginx日志部署

## nginx机器下载

### 下载安装包和java

```
Nginx机器下载：
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.8.2.rpm
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.8.2-x86_64.rpm
yum install -y java-1.8.0-openjdk-devel.x86_64
yum install *.rpm -y
```

### 编辑logstash配置文件

```
vim /etc/logstash/conf.d/nginx_log.conf

input {
   #从filebeat取数据，端口与filebeat配置文件一致
   beats {
     port => 5044
   }
}

filter{
        json {
        source => "message"
        remove_field => "message"
        remove_field => ["[agent][ephemeral_id]","[host][id]","[agent][id]","[host][hostname]","[agent][type]","[agent][version]","[host][name]","[host][os][codename]","[host][os][family]","[host][os][kernel]","[host][os][name]","[host][os][version]","[host][os][platform]","tags","url","[log][offset]","[log][file][path]","[input][type]","[ecs][version]","[_type]","[beat][hostname]","[offset]","[host][containerized]","[host][architecture]"]
      }
}

output {
       # 输出es，这的filetype就是在filebeat那边新增的自定义字段名
       if [filetype] == "gateway-json" {
         elasticsearch {
            hosts => ["165.154.46.51:9200"]
            index => "gateway-json.log-%{+YYYY.MM.dd}"
        }
       }
}
```

### 当前nginx的json日志格式

```
log_format json '{"@timestamp":"$time_iso8601",'
	'"real_ip": "$real_ip",'
	'"remote_addr": "$remote_addr", '
	'"geoip2_data_country_code":"$geoip2_data_country_code",'
	'"remote_user": "$remote_user", '
	'"body_bytes_sent":"$body_bytes_sent",'
	'"status":"$status",'
	'"request_uri": "$request_uri",'
	'"request_method": "$request_method",'
	'"http_referrer": "$http_referer", '
	'"http_host":"$host",'
	'"http_x_forwarded_for": "$http_x_forwarded_for", '
	'"server_ip":"$server_addr",'
	'"http_user_agent": "$http_user_agent",'
	'"total_bytes_sent":"$bytes_sent",'
	'"upstream_addr":"$upstream_addr",'
	'"request_time":"$request_time",'
	'"upstream_response_time":"$upstream_response_time",'
	'"bytes_sent":"$bytes_sent",'
	'"upstream_status":"$upstream_status"}';
    access_log logs/access.log access;
```

### 编辑filebeat配置文件

```
vim /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  backoff: "1s"
  tail_files: false
  paths:
    - /usr/local/nginx/logs/gateway-json.log
  fields:
    filetype: gateway-json
  fields_under_root: true
- type: log
  enabled: true
  backoff: "1s"
  tail_files: false
  paths:
    - /usr/local/nginx/logs/error.log
  fields:
    filetype: log_error
  fields_under_root: true
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
setup.template.settings:
  index.number_of_shards: 3
setup.kibana:
output.logstash:
  hosts: ["192.168.18.150:5044"]
processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
```

![](<../.gitbook/assets/image (3).png>)

![](<../.gitbook/assets/image (8).png>)

#### 启动 logstash and filebeat

systemctl start logstash filebeat

systemctl enable logstash filebeat

netstat -tnlp #出现 5044 9600 为正常状态

## ELK部署

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.2.rpm
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.8.2-x86_64.rpm
yum install -y java-1.8.0-openjdk-devel.x86_64
yum install *.rpm -y

#修改elasticsearch配置文件
mkdir /etc/systemd/system/elasticsearch.service.d/
cat >> /etc/systemd/system/elasticsearch.service.d/override.conf << EOF
[Service]
LimitMEMLOCK=infinity
EOF

systemctl daemon-reload

sed -i 's/#network.host: 192.168.0.1/network.host: 0.0.0.0/' /etc/elasticsearch/elasticsearch.yml
sed -i 's/#http.port: 9200/http.port: 9200/' /etc/elasticsearch/elasticsearch.yml

#启动elasticsearch
systemctl start elasticsearch
systemctl enable elasticsearch
systemctl status elasticsearch
#有的机器配置低 启动需要时间 等几分钟查看端口 9200 9300 出现代表启动成功
netstat -tnlp

#部署kibana
sed -i 's/#server.port: 5601/server.port: 5601/' /etc/kibana/kibana.yml
sed -i 's/#server.host: "localhost"/server.host: "0.0.0.0"/' /etc/kibana/kibana.yml
sed -i 's/#elasticsearch.hosts: \["http:\/\/localhost:9200"\]/elasticsearch.hosts: \["http:\/\/localhost:9200"\]/' /etc/kibana/kibana.yml
systemctl start kibana
systemctl enable kibana
systemctl status kibana
#有的机器配置低 启动需要时间 等几分钟查看端口 5601 出现代表启动成功

netstat -tnlp

#登入kibana
165.154.46.51:5601
# 查看kibana连接elasticsearch的状态 nodes1 为一个节点
返回部署 nginx里面的logstash

开启日志
```
