# 导入导出数据

## elasticsearch数据导出

```
安装elasticdump 命令
curl -sL https://rpm.nodesource.com/setup_10.x | bash -
yum install -y nodejs 
mkdir esdump && cd esdump
npm init -f
npm install elasticdump
```

### 导出索引分片

```
elasticdump \
  --input=http://192.168.18.57:9200/top_first_deposit_user_info \
  --output=/data/esdump/top_first_deposit_user_info_mapping.json \
  --type=mapping
```

### 导出索引数据

```
elasticdump \
  --input=http://192.168.18.57:9200/top_first_deposit_user_info \
  --output=/data/esdump/top_first_deposit_user_info.json \
  --type=data
```

### 恢复数据分片

```
./bin/elasticdump \
  --output=http://es实例IP:9200/index_name \
  --input=/home/indexdata/roll_vote_mapping.json \ # 导入数据目录
  --type=mapping
```

### 恢复索引数据

```
./bin/elasticdump \
  --output=http:///es实例IP:9200/top_summary_user_info \
  --input=/home/indexdata/roll_vote.json \ 
  --type=data
  --limit 10000
```
