# 删除索引脚本

```
#/bin/bash

#七天前
DATA=`date -d "1 days ago" +%Y.%m.%d`

#当前时间
NOW=`date`

#删除七天前的索引
curl -XDELETE http://127.0.0.1:9200/*-log-$DATA.*

if [ $? -eq 0 ];then
echo $NOW"----->del $DATE log success.." >> /tmp/es-index-clear.log
else
echo $NOW"----->del $DATE log fail.." >> /tmp/es-index-clear.log
fi
```
