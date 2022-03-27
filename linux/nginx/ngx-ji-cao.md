# Ngx 基操



### **常规配置模板**

```
#user  www www;
worker_processes auto;
error_log  /home/wwwlogs/nginx_error.log crit;
#pid        /usr/local/nginx/nginx.pid;

#Specifies the value for maximum file descriptors that can be opened by this process.
worker_rlimit_nofile 655350;

events
	{
		use epoll;
		worker_connections 655350;
	}

http
	{
		include       mime.types;
		default_type  application/octet-stream;

		server_names_hash_bucket_size 128;
		client_header_buffer_size 32k;
		large_client_header_buffers 4 32k;
		client_max_body_size 50m;
		server_tokens off;

		sendfile on;
		tcp_nopush     on;
		tcp_nodelay on;
		keepalive_timeout 60;
		

		fastcgi_connect_timeout 300;
		fastcgi_send_timeout 300;
		fastcgi_read_timeout 300;
		fastcgi_buffer_size 64k;
		fastcgi_buffers 4 64k;
		fastcgi_busy_buffers_size 128k;
		fastcgi_temp_file_write_size 256k;
		proxy_connect_timeout 600;
		proxy_read_timeout 600;
		proxy_send_timeout 600;
		proxy_buffer_size 64k;
		proxy_buffers   4 32k;
		proxy_busy_buffers_size 64k;
		proxy_temp_file_write_size 64k;
		
		gzip on;
		gzip_min_length  1k;
		gzip_buffers     4 16k;
		gzip_http_version 1.0;
		gzip_comp_level 2;
		gzip_types 	 text/plain application/x-javascript text/css application/xml text/xml application/json;
		gzip_vary on;
		log_format  access '$remote_addr - $remote_user [$time_local] $host '
                                   '"$request" $status $body_bytes_sent $request_time '
                                   '"$http_referer" "$http_user_agent" "$http_x_forwarded_for"'
                                   '$upstream_addr $upstream_status $upstream_response_time' ;
                                   
 #设置Web缓存区名称为cache_one，内存缓存空间大小为256MB，1天没有被访问的内容自动清除，硬盘缓存空间大小为30GB。
		proxy_temp_path   /home/proxy_temp_dir;
		proxy_cache_path  /home/proxy_cache_path levels=1:2 keys_zone=cache_one:256m inactive=1d max_size=30g;
		
        server {
            listen 80;
            server_name  _;
            return 403;
        }
        include vhost/*.conf;
}
```

以“exmaple.org”为例，如下为基于upstream负载均衡模式的配置：

```
upstream example_backend {
        server   127.0.0.1:9080;
        server   192.168.1.198:9080;
        server 172.16.0.4:80 weight=5 max_fails=3 fail_timeout=10s; # 权重、健康监测
        server 172.16.0.5:8080 backup; # 备份节点,
        ip_hash;  #调度算法
 }
 

server {
        listen 80;
        server_name www.example.org example.com .example.org;
    
    location / {
# 如果后端服务器出现502 或504错误代码的话nginx就不会请求某台服务器了,当后端服务器又工作正常了,nginx继续请求,这样一来达到了后端服务器健康状况检测的功能,
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
        proxy_pass http://example_backend;
        proxy_http_version 1.1;
        proxy_set_header        Host    $host;
        proxy_set_header        X-Real-IP       $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # 设置后端连接超时时间
        proxy_connect_timeout   600;  
        proxy_read_timeout  600;  
        proxy_send_timeout  600;  
        proxy_buffer_size   8k;  
        proxy_temp_file_write_size  64k;  
        
        # nginx本地cache开启
        proxy_cache cache_one;
        proxy_cache_valid 200 304 30d;
        proxy_cache_valid 301 302 404 1m;
        proxy_cache_valid any 1m;
        proxy_cache_key $host$request_uri;
        
        # 客户端缓存，在header中增加“Expires”
        expires 30d;
        add_header Cache-Control public;
        add_header X-Proxy-Cache $upstream_cache_status;

        proxy_set_header If-Modified-Since $http_if_modified_since;
        if_modified_since before;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|ico|swf|xml|css|js)$ {
        proxy_pass      http://example_backend;
        proxy_set_header        Host    $host;
        proxy_set_header        X-Real-IP       $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        expires 15d;
    }
    location ~ .*\.(jhtml)$ {
        proxy_pass      http://example_backend;
        proxy_set_header        Host    $host;
        proxy_set_header        X-Real-IP       $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        expires -1;
    }
    access_log  logs/www.example.org.log;
}
```

其他的proxy配置：

```
proxy_set_header :在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息。
proxy_connect_timeout:配置Nginx与后端代理服务器尝试建立连接的超时时间。
proxy_read_timeout : 配置Nginx向后端服务器组发出read请求后，等待相应的超时时间。
proxy_send_timeout：配置Nginx向后端服务器组发出write请求后，等待相应的超时时间。
proxy_redirect :用于修改后端服务器返回的响应头中的Location和Refresh。
```

### 用户认证

```
# 命令行
htpasswd -bc /usr/local/nginx/auth/passwd admin 123456

vim /usr/local/nginx/conf/vhost/www.conf 
server {  
        listen   80;   
        server_name www.com;     
        index  index.html index.htm; 
        root /www;   
location / { 
         auth_basic "test"; 
         auth_basic_user_file /usr/local/nginx/auth/passwd; 
} 

 
systemctl restart nginx
```

比如说访问某站点的路径为/forum/ , 此时想使用/bbs来访问此站点需要做url重写如下

```
location / {
	rewrite ^/forum/?$ /bbs/ permanent; 
}
```

比如说某站点有个图片服务器(10.0.0.1/p\_w\_picpaths/ ) 此时访问某站点上/p\_w\_picpaths/的资源时希望访问到图片服务器上的资源

```
location / {
	rewrite ^/p_w_picpaths/(.*\.jpg)$  /p_w_picpaths2/$1 break;
} 
```

### 域名跳转

```
server { 
    listen 80; 
    server_name www.com; 
    rewrite ^/ http://www.www.com/; 
    # return 301 http://www.andy.com/;
} 
```

### 域名镜像

```
server { 
    listen 80; 
    server_name  www.com; 
    rewrite ^/(.*)$ http://www.www.com/$1 last; 
} 
```

判断表达式

```
-f 和 !-f 用来判断是否存在文件
-d 和 !-d 用来判断是否存在目录
-e 和 !-e 用来判断是否存在文件或目录
-x 和 !-x 用来判断文件是否可执行
```

### 防盗链

```
location ~* \.(gif|jpg|png|swf|flv)$ { 
  valid_referers none blocked www.com; 
  if ($invalid_referer) { 
    rewrite ^/ http://www.com/403.html; 
  } 
```

### 黑白名单

```
if ($remote_addr !~ ^(222.65.126.38|100.110.15.17|100.110.15.18|127.0.0.1)) {
     rewrite ^/(.*) http://www.baidu.com/$1 permanent;
     #return 404；
  }
```

### **会话保持**

#### **ip\_hash**

```
# ip_hash使用源地址哈希算法，将同一客户端的请求只发往同一个后端服务器（除非该服务器不可用）。
# 问题： 当后端服务器宕机后，session会话丢失；同一客户端会被转发到同一个后端服务器，可能导致负载失衡；
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
}
```

sticky\_cookie\_insert

```
# 使用sticky_cookie_insert 启用会话亲缘关系，会导致来自同一客户端的请求被传递到一组服务器的同一台服务器。与ip_hash不同之处在于，它不是基于IP来判断客户端的，而是基于cookie来判断。因此可以避免上述ip_hash中来自同一客户端导致负载失衡的情况(需要引入第三方模块才能实现)。
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    sticky_cookie_insert srv_id expires=1h domain=3evip.cn path=/;
}
server {
    listen 80;
    server_name 3evip.cn;
    location / {
    	proxy_pass http://backend;
    }
}
```

expires：设置浏览器中保持cookie的时间 domain：定义cookie的域 path：为cookie定义路径

### 动静分离

为加快网站解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。 在动静分离的tomcat的时候比较明显，因为tomcat解析静态很慢，简单来说，是使用正则表达式匹配过滤，交给不同的服务器。

1、准备环境

```
192.168.62.159 代理服务器
192.168.62.157 动态资源
192.168.62.155 静态资源
```

2、配置nginx反向代理upstream

```bash
[root@nginx-server conf.d]# cat upstream.conf 

upstream static {
    server 192.168.62.155:80 weight=1 max_fails=1 fail_timeout=60s;
}

upstream phpserver {
    server 192.168.62.157:80 weight=1 max_fails=1 fail_timeout=60s;
}

server {
   listen      80;
   server_name     localhost;

   #动态资源加载
   location ~ \.(php|jsp)$ {
       proxy_pass http://phpserver;
       proxy_set_header Host $host:$server_port;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

   #静态资源加载
   location ~ .*\.(html|gif|jpg|png|bmp|swf|css|js)$ {
       proxy_pass http://static;
       proxy_set_header Host $host:$server_port;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

3、192.168.62.155 静态资源

```bash
#静态资源配置 
server {
    listen 80;
    server_name     localhost;

    location ~ \.(html|jpg|png|js|css|gif|bmp|jpeg) {
        root /home/www/nginx;
        index index.html index.htm;
    }
}
```

4、192.168.62.157 动态资源

```bash
server {
    listen      80;
    server_name     localhost;
    
    location ~ \.php$ {
        root           /home/nginx/html;  #指定网站目录
        fastcgi_pass   127.0.0.1:9000;    #指定访问地址
        fastcgi_index  index.php;       #指定默认文件
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name; #站点根目录，取决于root配置项
        include        fastcgi_params;  #包含nginx常量定义
    }
}
```

### 日志切割

方法一：

```bash
#!/bin/bash
# 适合单个网站日志文件

LOGS_PATH=/home/wwwroot/yunwei/logs
yesterday=`date  +"%F" -d  "-1 days"`
mv ${LOGS_PATH}/yunwei.log  ${LOGS_PATH}/yunwei-${yesterday}.log
kill -USR1 $(cat /var/logs/nginx.pid)
```

方法二：

```bash
#nginx日志切割

# Nginx pid
NGINX_PID=$(cat /var/logs/nginx.pid)

# 多个日志文件
LOGS=(xxx.access.log xxx.access.log)

# Nginx日志路径目录
BASH_PATH="/www/wwwlogs"

# xxxx年xx月
lOG_PATH=$(date -d yesterday +"%Y%m")

# 昨天日期
DAY=$(date -d yesterday +"%d")

# 循环移动
for log in ${LOGS[@]}
do
	# 先判断日志目录是否存在
    [[ ! -d "${BASH_PATH}/${lOG_PATH}" ]] && mkdir -p ${BASH_PATH}/${lOG_PATH}

    # 进入日志目录
    cd ${BASH_PATH}
    mv ${log} ${lOG_PATH}/${DAY}-${log}
    #kill -USR1 `ps axu | grep "nginx: master process" | grep -v grep | awk '{print $2}'`
    kill -USR1 ${NGINX_PID}

    # 删除30天的备份,最好是移动到其他位置，不建议 rm -fr
    #find ${BASH_PATH}/${lOG_PATH} -mtime +30 -name "." -exec rm -fr {} \;
done
```

方法三：

```bash
#!/bin/bash

#set the path to nginx log files
log_files_path="/usr/local/nginx/logs/"
log_files_dir=${log_files_path}$(date -d "yesterday" +"%Y")/$(date -d "yesterday" +"%m")
#set nginx log files you want to cut
log_files_name=(gw20 gw20-json)
#set the path to nginx.
nginx_sbin="/usr/bin/nginx"
#Set how long you want to save
save_days=10

############################################
#Please do not modify the following script #
############################################
mkdir -p $log_files_dir

log_files_num=${#log_files_name[@]}

#cut nginx log files
for((i=0;i<$log_files_num;i++));do
mv ${log_files_path}${log_files_name[i]}.log ${log_files_dir}/${log_files_name[i]}_$(date -d "yesterday" +"%Y%m%d").log
done

#delete 30 days ago nginx log files
find $log_files_path -mtime +$save_days -exec rm -rf {} \;

$nginx_sbin -s reload
```

方法四：

```bash
server{

    if ($time_iso8601 ~ '(\d{4}-\d{2}-\d{2})') {
        set $day $1;
    }

	access_log  /www/wwwlogs/xxx.com-$day.log main;
	error_log  /www/wwwlogs/xxx.com.error.log;
}
```

### location优先级

```bash
当有多条 location 规则时，nginx 有一套比较复杂的规则，优先级如下：
`精确匹配=
`前缀匹配^~（立刻停止后续的正则搜索）
`按文件中顺序的正则匹配~或~*
`匹配不带任何修饰的前缀匹配。

这个规则大体的思路是
`先精确匹配，没有则查找带有^~的前缀匹配，没有则进行正则匹配，最后才返回前缀匹配的结果（如果有的话）
```

### **HTTPS** **使用自颁发证书实现**

```bash
# 建立存放https证书的目录 
mkdir -pv /usr/local/nginx/conf/.sslkey 
# 生成网站私钥文件 
cd /usr/local/nginx/conf/.sslkey 
openssl genrsa -out https.key 1024 
# 生存网站证书文件,需要注意的是在生成的过程中需要输入一些信息根据自己的需要输入,但Common Name 项输入的必须是访问网站的FQDN 
openssl req -new -x509 -key https.key -out https.crt 
# 为了安全起见,将存放证书的目录权限设置为400 
chmod -R 400 /usr/local/nginx/conf/.sslkey 
 
#vim /usr/local/nginx/conf/vhost/www.conf 
server {  
        listen   443;   
        server_name www.com;   
        index  index.html index.htm;  
        root /www;   
        ssl                 on; 
        ssl_protocols       SSLv3 TLSv1 TLSv1.1 TLSv1.2; 
        ssl_ciphers         AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5; 
        ssl_certificate     /usr/local/nginx/conf/.sslkey/https.crt; 
        ssl_certificate_key /usr/local/nginx/conf/.sslkey/https.key; 
        ssl_session_cache   shared:SSL:10m; 
        ssl_session_timeout 10m; 
}   
 
#重新启动nginx服务
service nginx restar
```

### **基于HTTPS配置核心配置**

```bash
upstream example_backend {
    server   127.0.0.1:9080;
    server   192.168.1.198:9080;
 }
server {
        listen 80;
        server_name www.example.org example.org;
        rewrite ^/(.*)$  https://$host/$1 last;
 }
server {
    listen 443;
    server_name www.example.org example.org;

    ssl on;
    ssl_certificate      /usr/local/nginx/conf/ssl/example.crt;
    ssl_certificate_key  /usr/local/nginx/conf/ssl/example.key;
    ssl_session_timeout  5m;
    ssl_protocols  SSLv3 TLSv1 TLSv1.2 TLSv1.1;
    ssl_ciphers HIGH:!aNULL:!MD5:!EXPORT56:!EXP;
    ssl_prefer_server_ciphers   on;

    location / {
        ...
    }
}
```

### 根据页面不存在则自定义跳转

```bash
if (!-f $request_filename) { 
      rewrite ^(/.*)$ http://www.com permanent; 
} 
```

### 根据文件类型设置过期时间

```bash
location ~* \.(js|css|jpg|jpeg|gif|png|swf)$ {
    if (-f $request_filename) {
        expires 1h;
        break;
    }
}
```

### 根据域名自定义跳转

```bash
if ( $host = 'www.baidu.com' ) {
	rewrite ^/(.*)$ http://baidu.com/$1 permanent;
}
```

### 手机|PC端,作出相应的跳转

```bash
# 根据浏览器头部 URL 重写到指定目录
if ($http_user_agent ~* MSIE) { 
	rewrite  ^(.*)$  /msie/$1  break; 
} 

# 判断是否是手机端
if ( $http_user_agent ~* "(Android|iPhone|Windows Phone|UC|Kindle)" ) {
	rewrite ^/(.*)$ http://m.qp.com$uri redirect;
}
```

### 禁止访问目录|文件

```bash
location ~* \.(txt|doc)${
    root /data/www/wwwroot/linuxtone/test;
    deny all;
}
```

如果前端是反向代理的情况下：

```bash
location /admin/ {
	allow 192.168.1.0/24;
	deny all;
}

# 后端
# set $allow false;
# if ($allow = false) { return 403;}
if ($http_x_forwarded_for !~* ^192\.168\.1\.*) {
	return 403;
}
```

### 添加模块--支持Websock模块

Nginx动态添加模块

> 版本平滑升级，和添加模块操作类似

#### 准备模块

> 这里以 **nginx-push-stream-module** 为例,模块我放在 **/data/module** 下，你也可以放在其他位置

```bash
mkdir -p /data/module && cd /data/module/
git clone http://github.com/wandenberg/nginx-push-stream-module.git
```

#### 查看Nginx已安装模块

```bash
/usr/local/nginx/sbin/nginx -V
--prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-http_stub_status_module --with-http_gzip_static_module --with-http_flv_module --with-http_mp4_module --with-pcre
```

#### 备份源执行文件

> 备份原来的 nginx 可执行文件

```bash
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx_bak
#　有必要的话，可以再备份下配置文件，以防万一
```

#### 下载源码编译

> 下载相同版本的Nginx源码包编译（以前安装时的源码包），如果已经删除了可重新下载，版本相同即可

```bash
wget http://nginx.org/download/nginx-1.16.1.tar.gz
tar xf nginx-1.16.1.tar.gz
cd nginx-1.16.1
 ./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-http_stub_status_module --with-http_gzip_static_module --with-http_flv_module --with-http_mp4_module --with-pcre --add-module=/data/module/nginx-push-stream-module

# 编译Nginx（千万不要make install，不然就真的覆盖了）
make
mv objs/nginx /usr/local/nginx/sbin/
```

#### 查看是否安装

```bash
/usr/local/nginx/sbin/nginx -V
--prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-http_stub_status_module --with-http_gzip_static_module --with-http_flv_module --with-http_mp4_module --with-pcre --add-module=/data/module/nginx-push-stream-module
```

### 添加模块--支持健康检查模块

#### 缺陷？

自带健康检查的缺陷：

* Nginx只有当有访问时后，才发起对后端节点探测。
* 如果本次请求中，节点正好出现故障，Nginx依然将请求转交给故障的节点,然后再转交给健康的节点处理。所以不会影响到这次请求的正常进行。但是会影响效率,因为多了一次转发
* 自带模块无法做到预警
* 被动健康检查

使用第三访模块nginx\_upstream\_check\_module：

* 区别于nginx自带的非主动式的心跳检测，淘宝开发的tengine自带了一个提供主动式后端服务器心跳检测模块
* 若健康检查包类型为http，在开启健康检查功能后，nginx会根据设置的间隔向指定的后端服务器端口发送健康检查包，并根据期望的HTTP回复状态码来判断服务是否健康。
* 后端真实节点不可用，则请求不会转发到故障节点
* 故障节点恢复后，请求正常转发

#### 准备模块

```bash
yum install patch git -y
cd /usr/local/src
git clone https://github.com/yaoweibin/nginx_upstream_check_module.git
```

```bash
# 需进入源码包打补丁
# 个人习惯，源码放在 /usr/local/src 
# 例如我的 nginx 源码包存放： /usr/local/src/nginx-1.16.1 , 若源码已经删除，那么去官网上再下载同版本
cd /usr/local/src/nginx-1.16.1
patch -p1 < ../nginx_upstream_check_module/check_1.16.1+.patch 
```

#### 重新编译

```bash
nginx -V
# configure arguments: --prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-http_stub_status_module --add-module=/root/nginx_upstream_check_module/

# 在运行中的 nginx 添加模块； 首先一点: 修改东西之前要先备份
mv /usr/loca/nginx/sbin/nginx{,_bak}


./configure --prefix=/usr/local/nginx \
--user=www --group=www \
--with-http_ssl_module \
--with-http_stub_status_module \
--add-module=../nginx_upstream_check_module
make


# **别手贱， 千万不要 make install**
```

```bash
cp objs/nginx /usr/local/nginx/sbin/
```

#### 查看模块

```bash
nginx -V
configure arguments: --prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-http_stub_status_module --add-module=../nginx_upstream_check_module
```

#### 如何使用？

```bash
http {

    upstream cluster {
        server 192.168.0.1:80;
        server 192.168.0.2:80;
	    server 127.0.0.1:80;
        check interval=5000 rise=1 fall=3 timeout=4000;

        #check interval=3000 rise=2 fall=5 timeout=1000 type=ssl_hello;
        #check interval=3000 rise=2 fall=5 timeout=1000 type=http;
        #check_http_send "HEAD / HTTP/1.0\r\n\r\n";
        #check_http_expect_alive http_2xx http_3xx;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://cluster;
        }

        location /status {
            # 默认html，请求方式： check_status html|json|xml;
            # allow 允许的IP地址
            check_status;
            access_log   off;
            allow SOME.IP.ADD.RESS;
            deny all;
       }
    }

}

# kill -USER2 `cat /usr/local/nginx/logs/nginx.pid` #热升级nginx,如果当前nginx不是用绝对路径下的nginx命令启动的话，热升级无效。
# 只能 `nginx -s stop` && /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf`
```

```bash
# curl http://127.0.0.1/status?format=http
# curl http://127.0.0.1/status?format=xml
curl http://127.0.0.1/status?format=json
"""
{"servers": {
  "total": 3,
  "generation": 2,
  "server": [
    {"index": 0, "upstream": "cluster", "name": "192.168.0.1:80", "status": "down", "rise": 0, "fall": 432, "type": "tcp", "port": 0},
    {"index": 1, "upstream": "cluster", "name": "192.168.0.2:80", "status": "down", "rise": 0, "fall": 432, "type": "tcp", "port": 0},
    {"index": 2, "upstream": "cluster", "name": "127.0.0.1:80", "status": "up", "rise": 4, "fall": 0, "type": "tcp", "port": 0}
  ]
}}
"""
```

#### 注意事项

> 如果后端是基于域名访问，可使用check\_http\_send "GET /xxx HTTP/1.0\r\n HOST www.xxx.com\r\n\r\n";方式在请求时添加请求头信息

#### 参数详解

* interval： 检测间隔3秒
* fall: 连续检测失败次数5次时，认定relaserver is down
* rise: 连续检测成功2次时，认定relaserver is up
* timeout: 超时1秒
* default\_down: 初始状态为down,只有检测通过后才为up
* type: 检测类型方式 tcp
  * tcp :tcp 套接字,不建议使用，后端业务未100%启动完成,前端已经放开访问的情况
  * ssl\_hello： 发送hello报文并接收relaserver 返回的hello报文
  * http: 自定义发送一个请求，判断上游relaserver 接收并处理
  * mysql: 连接到mysql服务器，判断上游relaserver是否还存在
  * ajp: 发送AJP Cping数据包，接收并解析AJP Cpong响应以诊断上游relaserver是否还存活(AJP tomcat内置的一种协议)
  * fastcgi: php程序是否存活

#### GIthub 地址

> https://github.com/yaoweibin/nginx\_upstream\_check\_module

### 添加模块-支持国家城市模块

#### 安装依赖 libmaxminddb

> 因为需要读取在GeoIP2的IP数据库库，需要使用到libmaxminddb中的一个C库

```bash
wget https://github.com/maxmind/libmaxminddb/releases/download/1.3.2/libmaxminddb-1.3.2.tar.gz
tar zxvf libmaxminddb-1.3.2.tar.gz
cd libmaxminddb-1.3.2
./configure
make
make  install

# 添加库路径并更新库
sh -c "echo /usr/local/lib  >> /etc/ld.so.conf.d/local.conf"
ldconfig

# 或者
yum install libmaxminddb-devel -y
```

#### 下载GeoIP源码

```bash
wget https://github.com/leev/ngx_http_geoip2_module/archive/3.2.tar.gz
tar zxvf 3.2.tar.gz
```

#### Nginx重新编译

```bash
 ./configure --prefix=/usr/local/nginx --add-module=../ngx_http_geoip2_module-3.2
make && make install
```

#### 下载GeoLite

> 这个库是为了将IP地址翻译成具体的地址信息，下载需要注册... URL: https://www.maxmind.com/en/accounts/current/people/current 账号： xxxx@qq.com 密码： xxx..

```bash
#wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz     已作废
#wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.mmdb.gz  已作废
gunzip GeoLite2-City.mmdb.gz
gunzip GeoLite2-Country.mmdb.gz
mkdir /data/geoip
mv GeoLite2-City.mmdb  /data/geoip/city.mmdb
mv GeoLite2-Country.mmdb /data/geoip/country.mmdb
```

#### 启用 GeoIP

```bash
vim /usr/local/nginx/conf/nginx.conf
http {
    geoip2 /data/geoip/country.mmdb {
        $geoip2_data_country_code default=CN country iso_code;
        $geoip2_data_country_name country names en;
    }
    geoip2 /data/geoip/city.mmdb {
        $geoip2_data_city_name default=Shenzhen city names en;
    }

    server {
        listen       80;
        server_name  localhost;
        location / {
            add_header geoip2_data_country_code $geoip2_data_country_code;
            add_header geoip2_data_city_name $geoip2_data_city_name;
            if ($geoip2_data_country_code = CN){
                root /data/webroot/cn;
            }
            if ($geoip2_data_country_code = US){
                root /data/webroot/us;
            }
        }
｝
```

另外一种方式

```
http {
geoip2 /usr/local/nginx/geoip/GeoLite2-Country.mmdb {
            $geoip2_data_country_code source=$real_ip country iso_code;
            $geoip2_data_country_name country names en;
    }

    geoip2 /usr/local/nginx/geoip/GeoLite2-City.mmdb {
      $geoip2_metadata_city_build metadata build_epoch;
      #城市英文名，大多是拼音，有重复情况
      $geoip2_city_name_en source=$real_ip city names en;
      #城市中文名，部分城市没有中文名
      $geoip2_city_name_cn source=$real_ip city names zh-CN;
      #城市id，maxmaind 库里的id，非国际标准
      $geoip2_data_city_code source=$real_ip city geoname_id;

      $geoip2_subdivisions_name_en source=$real_ip subdivisions 0 names en;
    }

    map  $geoip2_data_country_code $allowed_country {
       default no;
       CN yes;
       HK yes;
       PH yes;
    }

    map  $geoip2_subdivisions_name_en $allowed_region {
       default no;
       Chongqing yes;
       Guangdong yes;
    }
    
    server {
    listen     80;
    server_name  localhost;
    location / {
       if ($allowed_country = no) {
               return 403;
          }
	if ($allowed_region = no) {
   	        return 403;
   	  }
    }
}
```

#### 检查 GeoIP

```bash
mkdir /data/webroot/us
mkdir /data/webroot/cn
echo "US Site" > /data/webroot/us/index.html
echo "CN Site" > /data/webroot/cn/index.html
```

```bash
curl 试一试
```

### 内置变量

```bash
http://wiki.nginx.org/HttpCoreModule#Variables   官方文档
$arg_PARAMETER
$args
$binary_remote_addr
$body_bytes_sent
$content_length
$content_type
$cookie_COOKIE
$document_root
$document_uri
$host
$hostname
$http_HEADER
$sent_http_HEADER
$is_args
$limit_rate
$nginx_version
$query_string
$remote_addr
$remote_port
$remote_user
$request_filename
$request_body
$request_body_file
$request_completion
$request_method
$request_uri
$scheme
$server_addr
$server_name
$server_port
$server_protocol
$uri
```
