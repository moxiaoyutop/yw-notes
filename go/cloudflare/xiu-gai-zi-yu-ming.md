# 修改子域名

```
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"strings"
)

const (
	username   = "邮箱"
	apikey = "key"
)

func jiance(ip string)  {
	apiUrl := "https://api.cloudflare.com/client/v4/zones"
	reader := strings.NewReader("{\n\t\"type\": \"A\",\n\t\"name\": \"test1\",\n\t\"content\": \""+ip+"\",\n\t\"ttl\": 1,\n\t\"proxied\": false\n}")
	req, err := http.NewRequest("PUT", apiUrl+"/域名id/dns_records/子域名id",reader)
	if err != nil {
		fmt.Println(err)
	}
	//req.Header.Set("content-type","application/json")
	req.Header.Set("X-Auth-Key",apikey)
	req.Header.Set("X-Auth-Email",username)

	resp, err := (&http.Client{}).Do(req)
	if err != nil {
		fmt.Println(err)
	}
	defer resp.Body.Close()
	conteent, err := ioutil.ReadAll(resp.Body)
	respBody := string(conteent)
	fmt.Println(respBody)
}

func main() {
	jiance("192.168.0.0")
}
```
