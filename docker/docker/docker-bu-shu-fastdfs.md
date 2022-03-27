# docker部署fastdfs

```
# pull镜像
docker pull delron/fastdfs

# 构建Tracker容器
docker run -d --network=host --name tracker -v /Users/zzs/develop/temp/tracker:/var/fdfs delron/fastdfs tracker

# 构建Storage容器
docker run -d --network=host --name storage -e TRACKER_SERVER=ip:22122 -v /Users/zzs/develop/temp/storage:/var/fdfs -e GROUP_NAME=group1 delron/fastdfs storage

# 进入storage容器测试
docker exec -it m3u8 /bin/bash
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf weixin.jpg
```
