# 批量证书检测

```
from urllib3.contrib import pyopenssl as reqs
from datetime import datetime
import whois

def get_expire_time(url):
    cert = reqs.OpenSSL.crypto.load_certificate(reqs.OpenSSL.crypto.FILETYPE_PEM,reqs.ssl.get_server_certificate((url, 443)))
    notafter = datetime.strptime(cert.get_notAfter().decode()[0:-1], '%Y%m%d%H%M%S')  # 获取到的时间戳格式是ans.1的，需要转换
    remain_days = notafter - datetime.now()  # 用证书到期时间减去当前时间
    domain = whois.whois(url)
    print(url, domain["expiration_date"][0], remain_days.days)
    print(url, domain["expiration_date"][0])
    #print(notafter)   #获取证书到期时间
    # print(remain_days.days)  #获取剩余天数

if __name__ == '__main__':
    with open('domian.txt', 'r') as urls:
        for url in urls.read().splitlines():
            get_expire_time(url)
```
