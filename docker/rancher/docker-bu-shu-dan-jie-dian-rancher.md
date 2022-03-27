# docker部署单节点rancher

```
docker run -d --restart=unless-stopped --privileged -p 8088:80 -p 8443:443 rancher/rancher:v2.5.12
```
