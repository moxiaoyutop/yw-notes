# Linux-openvpn

```
# 制作VPN
wget https://git.io/vpn -O openvpn-install.sh && bash openvpn-install.sh

# 编辑配置文件
vi /etc/openvpn/server/server.conf
   AES下方添加
duplicate-cn
max-clients 1000
log         openvpn.log
log-append    /var/log/openvpn.log
verb 4

# 重启
systemctl restart openvpn-server@server.service
```
