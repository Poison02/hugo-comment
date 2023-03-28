---
title: "CentOS7安装MySQL8"
date: 2023-03-28T15:32:56+08:00
draft: false
tags: ['Linux', 'MySQL']
category: '环境配置'
---

此教程是在网上搜集资料，写的一篇在CentOS7上安装MySQL8的教程，用于自己学习。

# 1、卸载mariadb

## 1.1检查是否有mariadb

```shell
rpm -qa|grep mariadb
```

## 1.2卸载

```shell
rpm -e --nodeps 文件名
```

## 1.3检查是否卸载

```
rpm -qa|grep mariadb
```

# 2、下载安装

## 2.1进入到目录下

```shell
cd /usr/local
```

## 2.2使用wget下载

```shell
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.28-linux-glibc2.12-x86_64.tar.xz
```

## 2.3检查mysql安装

```shell
rpm -qa|grep mysql
```

如果有输出，显示的不是想要的MySQL版本，则通过以下命令删除：

```shell
# 停止服务
systemctl stop mysqld

# 卸载
rpm -e --nodeps mysql文件名
```

## 2.4解压安装

解压指令：

```shell
# .tar.gz 后缀
tar -zxvf 文件名

# .tar.xz 后缀名
tar -Jxvf
```

解压完成：在 `/usr/local`下会生成MySQL的文件夹

# 3、配置

## 3.1基本设置：

文件夹重命名：通常命名为 `mysql版本号`

```shell
# 重命名（也可通过xftp修改）
mv 原文件夹名 mysql8

# 软连接
ln -s 文件夹名 mysql8
```

## 3.2添加path变量：可在全局使用MySQL：

### 3.2.1临时生效：

`export`命令（连接会话关闭后失效，通常用于测试环境）

```shell
export PATH=$PATH:/usr/local/mysql8/bin
```

### 3.2.2永久生效：修改配置文件

首先进入配置文件中

```shell
vim /etc/profile
```

在最后添加以下两行，主要是自己安装MySQL的路径

```shell
export PATH=$PATH:/usr/local/mysql8/bin
export PATH=$PATH:/usr/local/mysql8/support-files
```

使配置文件生效

```shell
source /etc/profile
```

### 3.2.3执行，确认安装

查看版本号：

```shell
mysql --version
```

删除MySQL安装包：

```shell
rm -rf 压缩包名
```

## 3.3创建用户组、用户

在 `/usr/local`下操作

### 3.3.1、创建用户组：`groupadd`

### 3.3.2、创建用户：`useradd`（-r 创建系统用户，-g指定用户组）

```shell
groupadd mysql8
useradd -r -g mysql8 mysql
```

数据目录

1、创建目录

```shell
mkdir -p /data/mysql8_data
```

2、赋予权限

```shell
# 更改属主和数组
chown -R mysql:mysql8 /data/mysql8_data

# 更改模式
chmod -R 750 /data/mysql8_data
```

# 4、初始化，启动

## 4.1在 `/usr/local/etc`下创建 `My.cnf`配置文件，用于初始化MySQL

配置方式：

通过 vim 配置：

```shell
vim /usr/local/etc/my.cnf
```

按 `i`进入编辑模式，按 `esc`保存，`:wq`退出

配置内容：

```shell
[mysql]
# 默认字符集
default-character-set=utf8mb4
[client]
port       = 3306
socket     = /tmp/mysql.sock
[mysqld]
port       = 3306
server-id  = 3306
user       = mysql
socket     = /tmp/mysql.sock
# 安装目录
basedir    = /usr/local/mysql8
# 数据存放目录
datadir    = /data/mysql8_data/mysql
log-bin    = /data/mysql8_data/mysql/mysql-bin
innodb_data_home_dir      =/data/mysql8_data/mysql
innodb_log_group_home_dir =/data/mysql8_data/mysql
# 日志及进程数据的存放目录
log-error =/data/mysql8_data/mysql/mysql.log
pid-file  =/data/mysql8_data/mysql/mysql.pid
# 服务端字符集
character-set-server=utf8mb4
lower_case_table_names=1
autocommit =1
##### 以上涉及文件夹名，注意修改
skip-external-locking
key_buffer_size = 256M
max_allowed_packet = 1M
table_open_cache = 1024
sort_buffer_size = 4M
net_buffer_length = 8K
read_buffer_size = 4M
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 64M
thread_cache_size = 128
#query_cache_size = 128M
tmp_table_size = 128M
explicit_defaults_for_timestamp = true
max_connections = 500
max_connect_errors = 100
open_files_limit = 65535
binlog_format=mixed
binlog_expire_logs_seconds =864000
# 创建表时使用的默认存储引擎
default_storage_engine = InnoDB
innodb_data_file_path = ibdata1:10M:autoextend
innodb_buffer_pool_size = 1024M
innodb_log_file_size = 256M
innodb_log_buffer_size = 8M
innodb_flush_log_at_trx_commit = 1
innodb_lock_wait_timeout = 50
transaction-isolation=READ-COMMITTED
[mysqldump]
quick
max_allowed_packet = 16M
[myisamchk]
key_buffer_size = 256M
sort_buffer_size = 4M
read_buffer = 2M
write_buffer = 2M
[mysqlhotcopy]
interactive-timeout
```

## 4.2初始化

在 `/usr/local/mysql8/bin`下

初始化命令：

```
--defaults-file 指定配置文件（放在--initialize前面）
--user 指定用户
--basedir 指定安装目录
--datadir 指定初始化数据目录
--initialize-insecure 初始化无密码（否则生成随机密码）
```

### 4.2.1使用以下命令初始化

```shell
mysqld --defaults-file=/usr/local/etc/my.cnf --basedir=/usr/local/mysql8 --datadir=/data/mysql8_data/mysql --user=mysql --initialize-insecure
```

启动MySQL

查看 `usr/local/mysql8/bin`下是否包含  `mysqld_safe`，用于后台安全启动MySQL

### 4.2.2启动服务

1、安全后台启动MySQL

```shell
# 完整命令
/usr/local/mysql8/bin/mysqld_safe --defaults-file=/usr/local/etc/my.cnf & 

# 若添加了PATH变量，可省略如下
mysqld_safe --defaults-file=/usr/local/etc/my.cnf & 
```

2、检查是否启动

```shell
ps -ef|grep mysql
```

### 4.2.3登录

- 无密码：若以 `--initialize-insecure`初始化，首次登陆跳过密码

  ```shell
  mysql -u root --skip-password
  ```

  

- 有密码：若初始化设置了随机密码，在 `/data/mysql8_data/mysql/mysql.log`查看

  ```shell
  mysql -u root -p
  ```

登录成功之后即可进入MySQL的命令行，和在Windows下一样的。

若有报错：检查MySQL服务是否开启

### 4.2.4修改密码：

首次修改

```sql
# 修改密码
alter user 'root'@'localhost' identified with mysql_native_password by '新密码';

# 刷新权限
flush privileges;
```

平时修改：

Linux命令行：

```shell
mysqladmin -u用户名 -p旧密码 password 新密码;
```

MySQL命令行：

```sql
# 设置密码
set password for '用户名'@'localhost' = password('密码');

# 刷新权限
flush privileges;
```

### 4.2.5退出、关闭服务

退出MySQL命令行：

```sql
exit;
quit;
```

关闭MySQL服务：

```sql
shutdown;
```

# 5、远程连接MySQL

## 5.1、创建远程连接用户

选择mysql数据库，查看当前用户

```sql
use mysql;
select user,host,plugin,authentication_string from user;
```

## 5.2、host字段表示可以访问当前数据库的主机，若为localhost则只有本地可以访问

## 5.3、创建用户，任意远程访问

```sql
-- 创建用户
create user 'root'@'%';

-- 设置首次密码
alter user 'root'@'%' identified with mysql_native_password by '123456';

# 授权用户所有权限，刷新权限
grant all privileges on *.* to 'root'@'%';
flush privileges;
```

## 5.4、查看用户

```sql
select user,host,plugin,authentication_string from user;
```

已经创建一个可以被任意远程主机访问的root用户。

远程连接MySQL

### 5.4.1、启动MySQL服务：

```shell
mysqld_safe --defaults-file=/usr/local/etc/my.cnf & 
```

### 5.4.2、开放端口：默认端口是3306

- 查看端口状态：no表示未开启 `firewall-cmd --query-port=3306/tcp`
- 永久开放端口：`firewall-cmd --add-port=3306/tcp --permanent`
- 重启防火墙：`systemctl restart firewalld`

### 5.4.3、远程连接

用navicat 输入对应的ip，用户名、密码即可连接。

# 6、设置MySQL开机自启

## 6.1、创建配置文件

```shell
touch /usr/lib/systemd/system/mysql.service
```

## 6.2、使用 `systemctl start mysql` 启动MySQL

## 6.3、查看MySQL进程 `ps -ef|grep mysql`

## 6.4、设置开机自启 `systemctl enable mysql`
