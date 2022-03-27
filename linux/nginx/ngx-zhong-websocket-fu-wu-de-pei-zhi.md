# Ngx 中websocket服务的配置



### websocket的连接有怎样的要求？

websocket需要请求头和响应头都设置 `Upgrade: WebSocket` 和 `Connection: Upgrade` 。

```bash
# 例如
location /chat/ {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

对代理服务器的请求中“Connection”标头字段的值取决于客户端请求标头中“Upgrade”字段的存在：

```bash
http {
    map $http_upgrade $connection_upgrade {
        default upgrade;
        ''      close;
    }
    server {
        ...

        location /chat/ {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
        }
    }
```
