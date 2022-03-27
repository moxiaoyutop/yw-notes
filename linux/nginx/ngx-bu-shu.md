# Ngx 部署

```
#安装依赖
yum -y install gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel openssl openssl-devel git
 

#下载需要编译安装的包
cd /usr/local/src/
wget https://nginx.org/download/nginx-1.20.2.tar.gz
wget https://github.com/maxmind/libmaxminddb/releases/download/1.6.0/libmaxminddb-1.6.0.tar.gz
git clone https://github.com/leev/ngx_http_geoip2_module.git
git clone https://github.com/zhouchangxun/ngx_healthcheck_module.git
git clone https://github.com/DDdark007/GeoLite2.git

#解压
tar xf libmaxminddb-1.6.0.tar.gz
tar xf nginx-1.16.1.tar.gz
 

#安装 libmaxminddb
cd libmaxminddb-1.6.0
./configure
make
make install
ldconfig
sh -c "echo /usr/local/lib  >> /etc/ld.so.conf.d/local.conf"
ldconfig
 

#安装nginx
cd ../nginx-1.16.1
./configure --with-http_ssl_module --with-stream --with-http_realip_module --http-client-body-temp-path=/tmp --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module --add-module=/usr/local/src/ngx_healthcheck_module --add-module=/usr/local/src/ngx_http_geoip2_module
 
make && make install
 

#软连接
ln -s /usr/local/nginx/sbin/nginx /usr/bin/
mkdir /usr/local/nginx/{geoip,data,ssl}

#部署geoip2 ip数据库
mv /usr/local/src/GeoLite2/GeoLite2-Country.mmdb /usr/local/nginx/geoip/

mkdir -p /usr/local/nginx/conf/vhost/{web,default}
cd /usr/local/nginx/conf/vhost/default
wget https://raw.githubusercontent.com/DDdark007/nginx/main/default.conf

#修改配置文件
mv /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.bak
cd /usr/local/nginx/conf/
wget  https://raw.githubusercontent.com/DDdark007/nginx/main/nginx.conf

#编辑子配置文件
cat >> /usr/local/nginx/conf/vhost/web/web.conf << EOF
server {
    listen     80;
    server_name 20.205.56.98 20.205.124.60;
    charset utf-8;
    access_log logs/web.log access;

    location / {
    root /usr/local/nginx/data/web/dist/;
    index index.html index.htm index.php pc.html;
    try_files $uri $uri/ /index.html;
    limit_rate 10M;
    proxy_set_header Host      $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#    if ($allowed_country = no) {
#      root /usr/local/nginx/data/limit403;
#    }
  }
}
EOF

#反向代理配置
cat >> /usr/local/nginx/conf/vhost/api/api.conf << EOF
upstream gamegw{
    server 20.205.56.98:80;
}

server {
    listen    80;
    #listen     443 ssl http2;
    server_name  20.205.124.60;
    access_log logs/api.log access;
    access_log  logs/api-json.log  json;
    client_max_body_size 200m;

    #ssl_certificate /usr/local/nginx/conf/ssl/xxx.com.crt;
    #ssl_certificate_key /usr/local/nginx/conf/ssl/xxx.com.key;


    location /{

        # http request header param
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # websocket request header param
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # websocket xieyi
        proxy_http_version 1.1;

        # timeout
        proxy_connect_timeout 300;
        proxy_read_timeout 300;
        proxy_send_timeout 300;

        proxy_pass http://gamegw;
        proxy_next_upstream error timeout invalid_header http_503 http_504;
    }
   #if ($allowed_country = no) {
   #   return 403;
   # }
}
EOF

#添加开机自启
chmod +x /etc/rc.d/rc.local
echo nginx >> /etc/rc.local

#在server_name 处添加域名 不需要的域名删除
#没有添加域名在配置文件中 会403 配置文件在 /usr/local/nginx/conf/vhost/default/default.conf

#先以添加虚拟机目录为
/usr/local/nginx/conf/vhost/api
/usr/local/nginx/conf/vhost/web

#如还需添加请在 nginx.conf内添加一行配置文件
```
