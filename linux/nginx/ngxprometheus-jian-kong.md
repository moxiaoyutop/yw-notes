# Ngx-prometheus-监控

```
# 普罗米修斯监控
# 下载模块
wget https://github.com/vozlt/nginx-module-vts/archive/v0.1.18.tar.gz

# 编译加载模块
./auto/configure --with-http_ssl_module --with-stream --with-http_realip_module --http-client-body-temp-path=/tmp --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module --add-module=../ngx_healthcheck_module --add-module=../ngx_http_geoip2_module --add-module=../ngx_req_status-master

# 替换nginx
cd /usr/local/nginx/sbin/
nginx -s stop
rm -rf nginx
mv /root/nginx-1.14.2/objs/nginx .
nginx

# 下载被监控端
wget  https://github.com/hnlq715/nginx-vts-exporter/releases/download/v0.9.1/nginx-vts-exporter-0.9.1.linux-amd64.tar.gz
tar -zxvf nginx-vts-exporter-0.9.1.linux-amd64.tar.gz
mv nginx-vts-exporter-0.9.1.linux-amd64 /usr/local/nginx-vts-exporter

cat >>/usr/lib/systemd/system/nginx_vts_exporter.service<< EOF
[Unit]
Description=prometheus_nginx_vts
After=network.target
 
[Service]
Type=simple
ExecStart=/usr/local/nginx-vts-exporter/nginx-vts-exporter  -nginx.scrape_uri http://192.168.18.151:8081/status/format/json
Restart=on-failure
 
[Install]
WantedBy=multi-user.target
EOF
 
systemctl daemon-reload
systemctl enable  nginx_vts_exporter
systemctl start  nginx_vts_exporter
systemctl status  nginx_vts_exporter

#修改普罗米修斯配置文件
  - job_name: 'nginx'
    static_configs:
    - targets: ['192.168.179.99:9913']

grafana添加 2949
完事

```
