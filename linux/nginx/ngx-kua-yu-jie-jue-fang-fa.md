---
description: Ngx跨域解决方法
---

# Ngx 跨域解决方法

### 一个完整的内核优化配置[#yi-ge-wan-zheng-de-nei-he-you-hua-pei-zhi](ngx-kua-yu-jie-jue-fang-fa.md#yi-ge-wan-zheng-de-nei-he-you-hua-pei-zhi "mention")

**JavaScript JS 跨域问题**

HTTP 错误 405 - 用于访问该页的 HTTP 动作未被许可

HTTP 错误 405.0 - Method Not Allowed

```
Method = "OPTIONS" | "GET" | "HEAD" | "POST" | "PUT" | "DELETE" | "TRACE" | "CONNECT" | extension-method
extension
```

在Nginx配置文件nginx.conf加入如下代码：

```
location / {

        # 跨域设置
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Credentials' 'true'; 
       
        if ( $request_method = 'OPTIONS' ) { return 200; }

 }
```

或者

```
location / {
        if (!-e $request_filename){
            rewrite  ^(.*)$  /index.php?s=$1  last;   break;
        }
        
        if ($request_method = 'OPTIONS') { 
        　　add_header Access-Control-Allow-Origin *; 
        　　add_header Access-Control-Allow-Credentials true; 
        　　add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS'; 
        　　#add_header '*'; 
        　　return 200; 
        }

        if ($request_method = 'POST') {
        　　add_header 'Access-Control-Allow-Origin' *; 
        　　add_header 'Access-Control-Allow-Credentials' 'true'; 
        　　add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS'; 
        　　#add_header '*';
        }

        if ($request_method = 'GET') {
        　　add_header 'Access-Control-Allow-Origin' *; 
        　　add_header 'Access-Control-Allow-Credentials' 'true'; 
        　　add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS'; 
        　　#add_header '*';
        }
 
```
