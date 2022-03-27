# Ngx 算法|Rewrite规则|优先级



### 环境信息

> 操作系统：CentOS 7.4 IP：192.168.0.111 在 Nginx 下，提供了 ngx\_http\_auth\_basic\_module 模块实现让用户只有输入正确的用户名密码才允许访问web内容。默认情况下，Nginx 已经安装了该模块。

1、先用第三方工具设置用户名、密码（其中密码已经加过密），然后保存到文件中， 2、接着在 Nginx 配置文件中根据之前事先保存的文件开启访问验证。

### 如何配置？

```bash
yum -y install httpd-tools
mkdir /usr/local/nginx/auth
htpasswd -bc /usr/local/nginx/auth/passwd admin 123456
cat /usr/local/nginx/auth/passwd
# /usr/local/nginx/auth/passwd 是生成密码后的文件保存路径(passwdfile)，admin是用户名（username）, 123456是密码
```

```bash
vi /usr/local/nginx/conf/nginx.conf
'''
	''' server 中添加
	auth_basic "Please input password";
	auth_basic_user_file /usr/local/nginx/auth/passwd;
'''


/usr/local/nginx/sbin/nginx -t
/usr/local/nginx/sbin/nginx -s reload
```

***

### 参数详解

* htpasswd命令选项参数说明 -c 创建一个加密文件; -n 不更新加密文件，只将htpasswd命令加密后的用户名密码显示在屏幕上; -m 默认htpassswd命令采用MD5算法对密码进行加密; -d htpassswd命令采用CRYPT算法对密码进行加密; -p htpassswd命令不对密码进行进行加密，即明文密码; -s htpassswd命令采用SHA算法对密码进行加密; -b htpassswd命令行中一并输入用户名和密码而不是根据提示输入密码; -D 删除指定的用户。

```bash
# 新增用户
htpasswd -b [passwdfile] [username] [passwd]

# 删除用户
htpasswd -D /usr/local/nginx/auth/passwd test

# 创建文件，添加用户（注意密码文件，否则已存在文件会覆盖原内容）
htpasswd -bc /usr/local/nginx/auth/passwd Test 123
```

> > Nginx算法、Rewrite规则、优先级
>
> ### Nginx 五种算法
>
> * 轮询、权重、ip\_hash、fair、url\_hash
>
> ```bash
> upstream test_app {
> 	server 192.168.8.2:8080 weight=1 max_fails=2 fail_timeout=30s;
> 	server 192.168.8.3:8080 weight=1 max_fails=2 fail_timeout=30s;
> }
>
>
> # test_app均衡两台后端JAVA服务，在30秒内nginx会与后端的某个server通信检测，如果检测连接失败2次，则Nginx会认为该server已经失效，然后踢出转发列表，然后在接下来的30s内，nginx不再讲请求转发给失效的server。
> # 另外，fail_timeout设置的时间对响应时间没影响，这个响应时间是用proxy_connect_timeout和proxy_read_timeout来控制的。
> # proxy_connect_timeout : Nginx与后端服务器连接的超时时间，发起握手等候响应超时时间。
> # proxy_read_timeout：连接成功后_等候后端服务器响应时间，其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）。
> # proxy_send_timeout :后端服务器数据回传时间，在规定时间之内后端服务器必须传完所有的数据。
> # keepalive_timout：一个http产生的tcp连接在传送完最后一个响应后，还需要等待多少秒后，才关闭这个连接。
> ```
>
> ***
>
> ### Rewrite规则
>
> last ： 相当于Apache里德(L)标记，表示完成rewrite <浏览器地址栏URL地址不变> break； 本条规则匹配完成后，终止匹配，不再匹配后面的规则 <浏览器地址栏URL地址不变> redirect： 返回302临时重定向，浏览器地址会显示跳转后的URL地址 permanent： 返回301永久重定向，浏览器地址栏会显示跳转后的URL地址
>
> \* 代表前面0或更多个字符 + 代表前面1或更多个字符 ? 代表前面0或1个字符 ^ 代表字符串的开始位置 $ 代表字符串结束的位置 . 为通配符，代表任何字符
>
> **根据路径条件**
>
> ```bash
> # www.test.com 跳转 www.test.com/new.index.html
> rewrite ^/$ http://www.test.com/index01.html permanent;
> ```
>
> **根据域名跳转**
>
> ```bash
> if ( $host = 'www.baidu.com' ) {
> 	rewrite ^/(.*)$ http://baidu.com/$1 permanent;
> }
> ```
>
> ```bash
> ## 当访问的文件和目录不存在时，重定向到某个php文件
> if( !-e $request_filename ){
> 	rewrite ^/(.*)$ index.php last;
> }
> ```
>
> **目录对换**
>
> > /xxxx/123456 --to--> /xxxx?id=123456
>
> ```bash
> rewrite ^/(.+)/(\d+) /$2?id=$1 last;
> ```
>
> **浏览器请求头跳转**
>
> ```bash
> if( $http_user_agent ~ MSIE) {
> 	rewrite ^(.*)$ /ie/$1 break;
> }
> ```
>
> **禁止访问指定后缀**
>
> > 禁止访问以.sh,.flv,.mp3为文件后缀名的文件
>
> ```bash
> location ~ .*\.(sh|flv|mp3|xml|rar|zip)$ {
> 	return 403;
> }
> ```
>
> **匹配浏览器信息**
>
> ```bash
> if ( $http_user_agent ~* "(Android)|(iPhone)|(Mobile)|(WAP)|(UCWEB)" ){
> 	rewrite ^/$  http://m.jfedu.net/ permanent;
> }
> ```
>
> **BBS rewrite 配置**
>
> > BBS论坛rewrite规则配置
>
> ```bash
> rewrite ^([^\.]*)/group-([0-9]+)-([0-9]+)\.html$ $1/forum.php?mod=group&fid=$2&page=$3 last;
> ```
>
> ***
>
> ### location 表达式
>
> \= 进行普通字符匹配。也就是完全匹配 ^\~ 表示普通字符匹配。使用前缀匹配，如果匹配成功，则不再匹配其他location \~ 表示执行一个正则匹配，区分大小写 \~\* 表示执行一个正则匹配，不区分大小写 @ 定义一个命名的location，使用内部定向时，例如 error\_page, try\_files
>
> ### location优先级
>
> 正location表达式的类型有关。相同类型的表达式，字符串长的会优先匹配。 第一优先级：等号类型（=）的优先级最高。一旦匹配成功，则不再查找其他匹配项 第二优先级：^~~类型表达式。一旦匹配成功，则不再查找其他匹配项。 第三优先级：正则表达式类型（~~ \~\*）的优先级次之。如果有多个location的正则能匹配的话，则使用正则表达式最长的那个。 第四优先级：常规字符串匹配类型
>
> ```bash
> location = / {
>     # 仅仅匹配请求 /
>     [ configuration A ]
> }
>
> location / {
>     # 匹配所有以 / 开头的请求。但是如果有更长的同类型的表达式，则选择更长的表达式。如果有正则表达式可以匹配，则优先匹配正则表达式。
>     [ configuration B ]
> }
>
> location /documents/ {
>     # 匹配所有以 /documents/ 开头的请求。但是如果有更长的同类型的表达式，则选择更长的表达式。如果有正则表达式可以匹配，则优先匹配正则表式 /
>     [ configuration C ]
> }
>
>
> location ^~ /images/ {
>     # 匹配所有以 /images/ 开头的表达式，如果匹配成功，则停止匹配查找。所以，即便有符合的正则表达式location，也不会被使用
>     [ configuration D ]
> }
>
>
> location ~* \.(gif|jpg|jpeg)$ {
>     # 匹配所有以 gif jpg jpeg结尾的请求。但是 以 /images/开头的请求，将使用 Configuration D
>     [ configuration E ]
> }
>
> ```
>
> **请求匹配示例**
>
> ```bash
> / -> configuration A/index.html -> configuration B/documents/document.html -> configuration C/images/1.gif -> configuration D/documents/1.jpg -> configuration E注意，以上的匹配和在配置文件中定义的顺序无关。
> ```
>
> **优先级别顺序**
>
> > (location =) > (location 完整路径 >) >(location ^\~ 路径) >(location \~\* 正则) >(location 路径)
