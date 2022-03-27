# Ngx 反代参数

```
#如果不想改变请求头“Host”的值，可以这样来设置：
proxy_set_header Host  $http_host;

#保留代理之前的host
proxy_set_header Host $host;

#保留代理之前的真实客户端ip
proxy_set_header X-Real-IP $remote_addr;

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

#在多级代理的情况下，记录每次代理之前的客户端真实ip
proxy_set_header HTTP_X_FORWARDED_FOR $remote_addr;

#指定修改被代理服务器返回的响应头中的location头域跟refresh头域数值
proxy_redirect default;
```
