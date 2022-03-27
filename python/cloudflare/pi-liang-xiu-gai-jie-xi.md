# 批量修改解析

```
import requests

url = "https://api.cloudflare.com/client/v4/zones"
# 密码
org_name="123456"
# 账户id
org_id="账户ID"
headers = {'content-type': 'application/json',
            'X-Auth-Key': 'apikey',# 秘钥
            'X-Auth-Email': 'email'} #账号

domain_list = []
domain_id = []
domain_dnsid = []

def CreateDomain():
    with open('domain.txt', mode='r', encoding='utf-8') as f:
        for line in f:
            domain_list.append(line.strip('\n'))
            urls = url + "?name=" + line.strip()
            # 添加域名
            # requestDomainData = {"account": {"id": org_id}, "name": line.strip('\n'), "jump_start": True}
            # urls2 = url
            # rets2 = requests.post(urls2, json=requestDomainData, headers=headers).json()
            # print(rets2)

            # 获取域名ID
            ret = requests.get(urls, headers=headers).json()['result']  # 获取域名信息 取result字段
            for rets in ret:
                domain_id.append(rets['id'])

        # 循环取域名id 然后 过滤出区域id 和 A记录id
        for domain_ids in domain_id:
            urls = url + "/" + domain_ids + "/dns_records"
            rer = requests.get(urls, headers=headers).json()['result']
            for rers in rer:
                domain_dnsid.append({rers['zone_id']:rers['id']})
    return
#启动 获取域名各种ID 和添加域名函数
CreateDomain()

# 有解析值 就删除重新解析，如果没有那么添加新的解析
def Delete_Domain():
    for domain_dnsids in domain_dnsid:
        lst1 = "".join(list(domain_dnsids.keys()))
        lst2 = "".join(list(domain_dnsids.values()))

        urls = url + "/" + lst1 + "/dns_records/" + lst2
        ret = requests.delete(urls, headers=headers)
        print("删除解析中")

def CreateDomains(CONTENT_IP):
    Delete_Domain()
    for domain_ids in domain_id:
        urls = url + "/" + domain_ids + "/dns_records"
        # 添加解析
        name_list = ['@', 'www', 'm', 'app']
        for name_lists in name_list:
            requestDomainsData = {"type": "CNAME", "name": name_lists, "content": CONTENT_IP, "ttl": 1, "priority": 10,"proxied": True}
            requests.post(urls, json=requestDomainsData, headers=headers).json()
#证书cname添加
            #requestDomainsSslData = {"type":"CNAME","name":"_A872AF32F3B454DFF7FD7B8B34A7E79A","content":"CBB594AB6577CF938548F41AD990A386.EAA59F62BF16A307EAD43D21B20CBACD.EC98D1D9FFCB48C.COMODOCA.COM","ttl":1,"priority":10,"proxied":False}
            #requests.post(urls, json=requestDomainsSslData, headers=headers)
        def domainSsl(valid):
            # valid values: off, flexible, full, strict
            # 有效值：关闭，灵活，完整，严格
            requestSslData = {"value": valid}
            #print(domain_ids)
            urls = url + "/" + domain_ids + "/settings/ssl"
            rer = requests.patch(urls, json=requestSslData, headers=headers).json()

        def domainHttps(valid):
            #on 开启             off 关闭
            requestHttpsData = {"id": "always_use_https", "value":valid}
            urls = url + "/" + domain_ids + "/settings/always_use_https"
            #print(urls)
            rer = requests.patch(urls, json=requestHttpsData, headers=headers).json()
            #print(rer)

        domainHttps('on')
        domainSsl('flexible')
CreateDomains('baidu.com')
```
