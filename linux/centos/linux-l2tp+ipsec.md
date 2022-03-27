# Linux-L2TP+IPSec

**一 使用一键脚本搭建L2TP+IPSec**

> 更多参考： [https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients-zh.md#windows-%E9%94%99%E8%AF%AF-809](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients-zh.md#windows-%E9%94%99%E8%AF%AF-809)

```bash
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/across/master/l2tp.sh
chmod +x l2tp.sh
./l2tp.sh
```

可以使用如下命令管理连接账户：

* l2tp -a : 增加一个连接账户
* l2tp -d : 删除一个连接账户
* l2tp -l : 展示现有的账户
* l2tp -m : 修改账户的密码

{% hint style="info" %}
注意： 如果你的云服务器有安全组，请放行：500，1701，4500这三个端口的UDP流量，否则会导致无法连接上。
{% endhint %}
