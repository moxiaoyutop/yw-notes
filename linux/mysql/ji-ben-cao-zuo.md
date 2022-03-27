# 基本操作

```
# 创建root用户
CREATE USER 'root'@'%' IDENTIFIED BY 'Top123456#';

# 授权root所有权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Top123456#'WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0;

# 查询库 
SHOW DATABASES; 

# 创建库
CREATE DATABASE MYSQLDATA; 

# 进入某个库
USE MYSQLDATA; 

# 查看表
SHOW TABLES;

# 清空表数据
delete from MYTABLE;

# 查看表结构
DESCRIBE mysql.component; 

# 库名+表名 清空表数据
delete from MYTABLE;

# 更改密码 
set password for root@'localhost'=password('mFs6&HVxtasd');

# 创建用户 
grant REPLICATION CLIENT on . to zabbix@"localhost" identified by "zabbix";

# 删除用户 
drop user zabbix@'localhost';

# 查看用户 
SELECT User, Host FROM mysql.user;

# 刷新授权表 
flush privileges;

# 主查看bin-log存放地址
show master status;

# binglog查看日志
/www/server/mysql/bin/mysqlbinlog mysql-bin.000005 --start-datetime="2019-12-17 07:21:09" --stop-datetime="2019-12-18 07:59:50"  > log.txt


```
