# rpm部署8.0

```
#!/bin/bash
#mysql
MYSQL_HOME=/data
MYSQL_PWD=Top123456
MYSQL_PORT=3306

function install_mysql8_el7()
{

  echo ""
  echo -e "\033[33m***************************************************自动部署mysql8.0**************************************************\033[0m"
  #建用户及目录
  groupadd -r mysql && useradd -r -g mysql mysql -d /home/mysql -m
  mkdir -p $MYSQL_HOME/datafile &&  mkdir -p $MYSQL_HOME/log &&  mkdir -p $MYSQL_HOME/backup
  chown -R mysql:mysql $MYSQL_HOME && chmod -R 755 $MYSQL_HOME

  #关闭selinux
  setenforce 0
  sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

  #下载包
  if [ -f /opt/mysql-8.0.17-1.el7.x86_64.rpm-bundle.tar ];then
      echo "*****存在mysql8安装包，无需下载*****"
  else
      ping -c 4 app.fslgz.com >/dev/null 2>&1
      if [ $? -eq 0 ];then
  wget https://cdn.mysql.com/archives/mysql-8.0/mysql-8.0.17-1.el7.x86_64.rpm-bundle.tar -P /opt/
      else
        echo "please download mysql8 package manual !"
    exit $?
      fi
  fi

  #配yum安装mysql依赖包
  rpm -qa|grep libaio-devel
  if [ $? -eq 1 ];then
    yum install -y gcc gcc-c++ openssl openssl-devel libaio libaio-devel  ncurses  ncurses-devel &>/dev/null
    action "***************安装mysql依赖包完成***************" /bin/true
    else
      action "****************已安装mysql依赖包****************" /bin/false
  fi

  #安装mysql8.0
  ps -ef|grep mysqld |grep -v grep | grep root
  if [ $? -eq 0 ] ;then
     echo "*****************已存在mysql进程*****************"
     exit $?
  else
   # 卸载 mysql
     rpm -qa|grep mysql|xargs -i rpm -e --nodeps {}
     # uninstall mariadb-libs
     rpm -qa|grep mariadb|xargs -i rpm -e --nodeps {}
     # 安装mysql
     action "***************开始安装mysql数据库***************" /bin/true
     cd /opt
     tar -xvf mysql-8.0.17-1.el7.x86_64.rpm-bundle.tar -C /opt/  &>/dev/null
     yum install *.rpm -y
     cd /root/
  fi

  #配置my.cnf
  cp /etc/my.cnf /etc/my.cnf_${DATE}bak &>/dev/null
cat << EOF > /etc/my.cnf
[mysqld]
#解决时区问题
default-time-zone = '+8:00'
#解决mysql日志时间与系统时间不一致问题
log_timestamps=SYSTEM
port=${MYSQL_PORT}
datadir=${MYSQL_HOME}/datafile
log-error=${MYSQL_HOME}/log/mysqld.log
#mysql8默认禁用Symbolic links，无需再去标记禁用
#symbolic-links=0
bind-address=0.0.0.0
lower_case_table_names=1
character_set_server=utf8mb4
max_allowed_packet=500M
#SQL Mode的NO_AUTO_CREATE_USER取消
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
#InnoDB用于缓存数据、索引、锁、插入缓冲、数据字典等
innodb_buffer_pool_size=4G
#InnoDB的log buffer
innodb_log_buffer_size = 64M
#InnoDB redo log大小
innodb_log_file_size = 256M
#InnoDB redo log文件组
innodb_log_files_in_group = 2
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1
#连接数
max_connections=600
max_connect_errors=1000
max_user_connections=400
#设置临时表最大值
max_heap_table_size = 100M
tmp_table_size = 100M
#每个连接都会分配的一些排序、连接等缓冲
sort_buffer_size = 2M
join_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 2M
#mysql8自动关闭query cache
#query_cache_size = 0
#如果是以InnoDB引擎为主的DB，专用于MyISAM引擎的 key_buffer_size 可以设置较小，8MB 已足够,如果是以MyISAM引擎为主，可设置较大，但不能超过4G
key_buffer_size = 8M
#设置慢查询阀值，单位为秒
long_query_time = 60
slow_query_log=1
log_output=table,File    #日志输出会写表，也会写日志文件，为了便于程序去统计，所以最好写表
slow_query_log_file=${MYSQL_HOME}/log/slow.log
#快速预热缓冲池
innodb_buffer_pool_dump_at_shutdown=1
innodb_buffer_pool_load_at_startup=1
#打印deadlock日志
innodb_print_all_deadlocks=1
#二进制配置
server-id = 1
log-bin = ${MYSQL_HOME}/log/mysql-bin.log
log-bin-index =${MYSQL_HOME}/log/binlog.index
log_bin_trust_function_creators=1
binlog_format = row
gtid_mode = ON
enforce_gtid_consistency = ON
#expire-logs-days参数取消，修改成binlog_expire_logs_seconds,单位为秒，以下代表15天
binlog_expire_logs_seconds=1296000
#schedule
event_scheduler = on
#unknown variable 'show_compatibility_56=on'
#show_compatibility_56=on
#处理TIMESTAMP with implicit DEFAULT value is deprecated
explicit_defaults_for_timestamp=true
#MySQL 8.0改了默认加密方式为“caching_sha2_password”，这里改回来
default_authentication_plugin=mysql_native_password
#禁用SSL提高性能
skip_ssl
#timeout
wait_timeout = 3600
interactive_timeout = 3600
net_read_timeout = 3600
net_write_timeout = 3600
#密码有效期
default_password_lifetime=360
EOF

  #启动数据库
  systemctl start mysqld.service
  sleep 3
  MYSQL_TEMP_PWD=$(grep "temporary password" ${MYSQL_HOME}/log/mysqld.log|cut -d "@" -f 2|awk '{print $2}')
  #mysql 8密码策略validate_password_policy 变为validate_password.policy
  #MYSQL 8.0内新增加mysql_native_password函数，通过更改这个函数密码来进行远程连接
  mysql -hlocalhost  -P${MYSQL_PORT}  -uroot -p"${MYSQL_TEMP_PWD}" -e "set global validate_password.policy=0" --connect-expired-password
  mysql -hlocalhost  -P${MYSQL_PORT}  -uroot -p"${MYSQL_TEMP_PWD}" -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '${MYSQL_PWD}'"  --connect-expired-password
  mysql -hlocalhost  -P${MYSQL_PORT}  -uroot -p"${MYSQL_PWD}" -e "CREATE USER 'root'@'%' IDENTIFIED BY '${MYSQL_PWD}'"  --connect-expired-password
  mysql -hlocalhost  -P${MYSQL_PORT}  -uroot -p"${MYSQL_PWD}" -e "grant all privileges on *.* to root@'%'"  --connect-expired-password
  #授权root远程登录需输入以下命令：
  #create user root@'%' identified by 'fswl@1234';
  #grant all privileges on *.* to root@'%';
  echo -e "\033[33m************************************************完成mysql8.0数据库部署***********************************************\033[0m"
cat > /tmp/mysql8.log  << EOF
mysql安装目录：${MYSQL_HOME}
mysql版本：Mysql-8.0
mysql端口：${MYSQL_PORT}
mysql密码：${MYSQL_PWD}
EOF
  cat /tmp/mysql8.log
  echo -e "\e[1;31m 以上信息10秒后消失，保存在/tmp/mysql8.log文件下 \e[0m"
  echo -e "\033[33m*********************************************************************************************************************\033[0m"
  echo ""
  sleep 10
}


install_mysql8_el7
```
