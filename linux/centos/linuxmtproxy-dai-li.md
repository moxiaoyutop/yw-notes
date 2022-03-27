# Linux-MTProxy代理



### 什么是MTProxy

> MTProxy是一款专门属于`Telegram`的代理服务，用户只需要在自己国外服务器上搭建这个服务，就可以不扶墙而去使用`Telegram`
>
> * 项目地址：`https://github.com/TelegramMessenger/MTProxy`

```bash
yum install openssl-devel gcc git -y
git clone https://github.com/TelegramMessenger/MTProxy
cd MTProxy
make && cd objs/bin
export MTProxyDir=$PWD
export SECRET=$(head -c 16 /dev/urandom | xxd -ps)
export PORT=9999

# 获取与Telegram服务器的连接、获取配置文件、获取秘钥
curl -s https://core.telegram.org/getProxySecret -o proxy-secret
curl -s https://core.telegram.org/getProxyConfig -o proxy-multi.conf


cat > /usr/lib/systemd/system/MTProxy.service << EOF
[Unit]
Description=MTProxy
After=network.target

[Service]
Type=simple
WorkingDirectory=${MTProxyDir}
ExecStart=${MTProxyDir}/mtproto-proxy -u nobody -p 8888 -H ${PORT} -S ${SECRET} --aes-pwd proxy-secret proxy-multi.conf -M 1
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload && systemctl start MTProxy && systemctl enable MTProxy
```

```bash
# 浏览器访问： tg://proxy?server=SERVER_NAME&port=PORT&secret=SECRET
'''
    SERVER_NAME：你服务器的ip
    PORT：你设置的端口
    SECRET：你上面获取的秘钥
'''
```

### 一键脚本

```bash
yum install wget -y
wget https://raw.githubusercontent.com/TyrantJoy/One-click-shell_script/master/MTProxy/MTProxy.sh
chmod +x MTProxy.sh
./MTProxy.sh
```
