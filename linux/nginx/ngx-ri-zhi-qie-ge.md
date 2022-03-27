# Ngx 日志切割

方法1

```
#nginx日志切割

# Nginx pid
NGINX_PID=$(cat /var/logs/nginx.pid)

# 多个日志文件
LOGS=(xxx.access.log xxx.access.log)

# Nginx日志路径目录
BASH_PATH="/www/wwwlogs"

# xxxx年xx月
lOG_PATH=$(date -d yesterday +"%Y%m")

# 昨天日期
DAY=$(date -d yesterday +"%d")

# 循环移动
for log in ${LOGS[@]}
do
	# 先判断日志目录是否存在
    [[ ! -d "${BASH_PATH}/${lOG_PATH}" ]] && mkdir -p ${BASH_PATH}/${lOG_PATH}

    # 进入日志目录
    cd ${BASH_PATH}
    mv ${log} ${lOG_PATH}/${DAY}-${log}
    #kill -USR1 `ps axu | grep "nginx: master process" | grep -v grep | awk '{print $2}'`
    kill -USR1 ${NGINX_PID}

    # 删除30天的备份,最好是移动到其他位置，不建议 rm -fr
    #find ${BASH_PATH}/${lOG_PATH} -mtime +30 -name "." -exec rm -fr {} \;
done
```

方法2

```
#!/bin/bash

#set the path to nginx log files
log_files_path="/usr/local/nginx/logs/"
log_files_dir=${log_files_path}$(date -d "yesterday" +"%Y")/$(date -d "yesterday" +"%m")
#set nginx log files you want to cut
log_files_name=(gamefile)
#set the path to nginx.
nginx_sbin="/usr/local/nginx/sbin/nginx"
#Set how long you want to save
save_days=10

############################################
#Please do not modify the following script #
############################################
mkdir -p $log_files_dir

log_files_num=${#log_files_name[@]}

#cut nginx log files
for((i=0;i<$log_files_num;i++));do
mv ${log_files_path}${log_files_name[i]}.log ${log_files_dir}/${log_files_name[i]}_$(date -d "yesterday" +"%Y%m%d").log
done

#delete 30 days ago nginx log files
find $log_files_path -mtime +$save_days -exec rm -rf {} \; 

$nginx_sbin -s reload
```
