# 批量域名检测

```
from whois import whois

def Update_expir(dm):
    global expir_time   #设置全局变量
    try:
        res = whois(dm)
        if res.expiration_date is not None:
            if isinstance(res.expiration_date, list):
                expir_time = res.expiration_date[0]
            else:
                expir_time = res.expiration_date
            return expir_time
        return False
    except:
        return False

if __name__ == '__main__':
    with open('domian.txt', 'r') as urls:
        for dm in urls.read().splitlines():
            Update_expir(dm)
            print(dm, expir_time)
```
