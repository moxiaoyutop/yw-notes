# fastdfs三集群部署

参考网址[https://github.com/happyfish100/fastdfs/wiki](https://github.com/happyfish100/fastdfs/wiki)

### 编译环境

```
yum install git gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel wget vim -y
mkdir /home/dfs #创建数据存储目录
cd /usr/local/src #切换到安装目录准备下载安装包
```

### 安装libfastcommon

```
git clone https://github.com/happyfish100/libfastcommon.git --depth 1
cd libfastcommon/
./make.sh && ./make.sh install #编译安装
```

### 安装fastdfs

```
git clone https://github.com/happyfish100/fastdfs.git --depth 1
cd fastdfs/
./make.sh && ./make.sh install #编译安装
#配置文件准备
cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf #客户端文件，测试用
cp /usr/local/src/fastdfs/conf/http.conf /etc/fdfs/ #供nginx访问使用
cp /usr/local/src/fastdfs/conf/mime.types /etc/fdfs/ #供nginx访问使用
```

### 安装fastdfs-nginx-module

```
cd
git clone https://github.com/happyfish100/fastdfs-nginx-module.git --depth 1
cp /usr/local/src/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs
```

### 安装nginx

```
wget http://nginx.org/download/nginx-1.15.4.tar.gz #下载nginx压缩包
tar -zxvf nginx-1.15.4.tar.gz #解压
cd nginx-1.15.4/
#添加fastdfs-nginx-module模块
./configure --add-module=/usr/local/src/fastdfs-nginx-module/src/ 
make && make install #编译安装
```

### 分布式部署

```
#tracker配置
vim /etc/fdfs/tracker.conf
port=22122  # tracker服务器端口（默认22122,一般不修改）
base_path=/home/dfs  # 存储日志和数据的根目录

#storage配置
vim /etc/fdfs/storage.conf
#需要修改的内容如下
port=23000  # storage服务端口（默认23000,一般不修改）
base_path=/home/dfs  # 数据和日志文件存储根目录
store_path0=/home/dfs  # 第一个存储目录
tracker_server=192.168.52.2:22122  # 服务器1
tracker_server=192.168.52.3:22122  # 服务器2
tracker_server=192.168.52.4:22122  # 服务器3
http.server_port=8888  # http访问文件的端口(默认8888,看情况修改,和nginx中保持一致)

#client测试
vim /etc/fdfs/client.conf
#需要修改的内容如下
base_path=/home/dfs
tracker_server=192.168.52.2:22122  # 服务器1
tracker_server=192.168.52.3:22122  # 服务器2
tracker_server=192.168.52.4:22122  # 服务器3
#保存后测试,返回ID表示成功 如：group1/M00/00/00/xx.tar.gz
fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/nginx-1.15.4.tar.gz

#配置nginx访问
vim /etc/fdfs/mod_fastdfs.conf
#需要修改的内容如下
tracker_server=192.168.52.2:22122  # 服务器1
tracker_server=192.168.52.3:22122  # 服务器2
tracker_server=192.168.52.4:22122  # 服务器3
url_have_group_name=true
store_path0=/home/dfs

#配置nginx.config
vim /usr/local/nginx/conf/nginx.conf
#添加如下配置
server {
    listen       8888;    ## 该端口为storage.conf中的http.server_port相同
    server_name  localhost;
    location ~/group[0-9]/ {
        ngx_fastdfs_module;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    root   html;
    }
}
```

### 启动

```
tracker
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start
storage
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf start

#保存后测试,返回ID表示成功 如：group1/M00/00/00/xx.tar.gz
fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/nginx-1.15.4.tar.gz
```

### 配置nginx访问

```
#配置nginx访问
upstream gamefile {
    server 152.32.185.40:8888 weight=1 max_fails=2 fail_timeout=30s;
    server 152.32.191.135:8888 weight=1 max_fails=2 fail_timeout=30s;
    server 128.1.132.11:8888 weight=1 max_fails=2 fail_timeout=30s;
}
server {
    listen     80;
    server_name 152.32.185.40;
 
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
        add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
        proxy_pass http://gamefile;
        proxy_set_header Host $host;
        proxy_read_timeout 300;
        proxy_send_timeout 300;
        proxy_buffer_size 64k;
        proxy_buffers   4 64k;
        proxy_busy_buffers_size 128k;
        proxy_temp_file_write_size 128k;
        client_max_body_size    150m;
    }
}
```
