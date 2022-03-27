# rancher部署单节点nacos

首先配置映射 配置nacos-config

![](<../../.gitbook/assets/image (7).png>)

```
MODE: standalone
  MYSQL_SERVICE_DB_NAME: nacos-test
  MYSQL_SERVICE_HOST: 172.16.90.9
  MYSQL_SERVICE_PASSWORD: Nacos#123
  MYSQL_SERVICE_PORT: "3306"
  MYSQL_SERVICE_USER: nacos
  SPRING_DATASOURCE_PLATFORM: mysql
```

![](<../../.gitbook/assets/image (15).png>)

#### 3.点击启动即可创建

等待创建成功，可以通过节点（node）地址+端口访问到，\
比如：

> [http://172.16.90.34:30848/nacos\
> 默认用户名和密码是：nacos](http://172.16.90.34:30848/nacos%E9%BB%98%E8%AE%A4%E7%94%A8%E6%88%B7%E5%90%8D%E5%92%8C%E5%AF%86%E7%A0%81%E6%98%AF%EF%BC%9Anacos)

访问地址

![](<../../.gitbook/assets/image (18).png>)
