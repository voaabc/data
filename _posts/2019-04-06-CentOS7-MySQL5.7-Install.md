---
layout: post
title: CentOS7安装MySQL5.7
categories: MySQL
tags: MySQL CentOS7 
---

* content
{:toc}

###安装前的检查

####检查Linux系统

```shell
[root@mysql5507 ~]# cat /etc/system-release
CentOS Linux release 7.6.1810 (Core)
[root@mysql5507 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1819         132         153           9        1533        1468
Swap:          2047           0        2047
[root@mysql5507 ~]# cat /proc/cpuinfo| grep "processor"| wc -l
1


```

####检查是否安装了MySQL

```shell
[root@mysql5507 ~]# rpm -qa | grep mysql
[root@mysql5507 ~]# rpm -qa | grep mariadb
mariadb-libs-5.5.60-1.el7_5.x86_64
[root@mysql5507 ~]# systemctl stop mariadb
Failed to stop mariadb.service: Unit mariadb.service not loaded.
[root@mysql5507 ~]# rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
[root@mysql5507 ~]# rpm -qa | grep mariadb
```

####检查是否开启防火墙
```shell
[root@mysql5507 ~]# firewall-cmd --state
running
[root@mysql5507 ~]# systemctl stop firewalld.service
[root@mysql5507 ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@mysql5507 ~]# systemctl enable firewalld.service
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
[root@mysql5507 ~]# systemctl start firewalld
[root@mysql5507 ~]# firewall-cmd --state
running
[root@mysql5507 ~]# firewall-cmd --zone=public --add-port=3306/tcp --permanent
success
[root@mysql5507 ~]# firewall-cmd --reload
success
[root@mysql5507 ~]# firewall-cmd --zone=public --list-ports
3306/tcp
[root@mysql5507 ~]# systemctl stop firewalld
[root@mysql5507 ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@mysql5507 ~]# firewall-cmd --state
not running
```

####检查NUMA

```shell
[root@mysql5507 ~]# rpm -qa | grep libaio
libaio-0.3.109-13.el7.x86_64
[root@mysql5507 ~]# numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0
node 0 size: 2047 MB
node 0 free: 718 MB
node distances:
node   0 
  0:  10 
```

```shell 
###检查ulimit
[root@mysql5507 ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7183
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7183
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

###检查依赖包libaio
```shell
[root@mysql5507 ~]# yum install -y libaio
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.cn99.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Package libaio-0.3.109-13.el7.x86_64 already installed and latest version
Nothing to do
```

###检查selinux
```shell
[root@mysql5507 ~]# getenforce
Enforcing
[root@mysql5507 ~]# sed -i "s#SELINUX=enforcing#SELINUX=disabled#g" /etc/sysconfig/selinux
[root@mysql5507 ~]# more /etc/sysconfig/selinux 

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 


[root@mysql5507 ~]# getenforce
Enforcing
[root@mysql5507 ~]# setenforce 0
[root@mysql5507 ~]# getenforce
Permissive
```



####安装MySQL

###创建用户和目录

```shell
[root@mysql5507 ~]# groupadd mysql
[root@mysql5507 ~]# useradd -g mysql -d /usr/local/mysql -s /sbin/nologin -M mysql
[root@mysql5507 ~]# id mysql
uid=1001(mysql) gid=1001(mysql) groups=1001(mysql)
[root@mysql5507 ~]# mkdir /opt/mysql
[root@mysql5507 ~]# tar zxf mysql-5.7.18-linux-glibc2.5-x86_64.tar.gz -C /opt/mysql
[root@mysql5507 ~]# cd /usr/local
[root@mysql5507 local]# ln -s /opt/mysql/mysql-5.7.18-linux-glibc2.5-x86_64/  mysql
[root@mysql5507 local]# cd mysql/
[root@mysql5507 mysql]# pwd
/usr/local/mysql
[root@mysql5507 mysql]# cd ..
[root@mysql5507 local]# chown -R mysql.mysql mysql
[root@mysql5507 local]# ll
total 0
drwxr-xr-x. 2 root  root   6 Apr 11  2018 bin
drwxr-xr-x. 2 root  root   6 Apr 11  2018 etc
drwxr-xr-x. 2 root  root   6 Apr 11  2018 games
drwxr-xr-x. 2 root  root   6 Apr 11  2018 include
drwxr-xr-x. 2 root  root   6 Apr 11  2018 lib
drwxr-xr-x. 2 root  root   6 Apr 11  2018 lib64
drwxr-xr-x. 2 root  root   6 Apr 11  2018 libexec
lrwxrwxrwx. 1 mysql mysql 46 Apr  6 12:53 mysql -> /opt/mysql/mysql-5.7.18-linux-glibc2.5-x86_64/
drwxr-xr-x. 2 root  root   6 Apr 11  2018 sbin
drwxr-xr-x. 5 root  root  49 Apr  6 10:45 share
drwxr-xr-x. 2 root  root   6 Apr 11  2018 src
[root@mysql5507 local]# cd mysql/
[root@mysql5507 mysql]# ll
total 36
drwxr-xr-x.  2 root root   4096 Apr  6 12:53 bin
-rw-r--r--.  1 7161 31415 17987 Mar 18  2017 COPYING
drwxr-xr-x.  2 root root     55 Apr  6 12:53 docs
drwxr-xr-x.  3 root root   4096 Apr  6 12:52 include
drwxr-xr-x.  5 root root    229 Apr  6 12:53 lib
drwxr-xr-x.  4 root root     30 Apr  6 12:53 man
-rw-r--r--.  1 7161 31415  2478 Mar 18  2017 README
drwxr-xr-x. 28 root root   4096 Apr  6 12:53 share
drwxr-xr-x.  2 root root     90 Apr  6 12:53 support-files
```

###创建配置文件my.cnf
```shell
[root@mysql5507 ~]# vim /etc/my.cnf
#
[client]
port	= 3306
socket	= /data/mysql/mysql.sock

[mysql]
prompt="\u@mysqldb \R:\m:\s [\d]> "
no-auto-rehash

[mysqld]
user	= mysql
port	= 3306
basedir	= /usr/local/mysql
datadir	= /data/mysql
socket	= /data/mysql/mysql.sock
pid-file = mysqldb.pid
character-set-server = utf8mb4
skip_name_resolve = 1

#若你的MySQL数据库主要运行在境外，请务必根据实际情况调整本参数
time_zone = "+8:00"

open_files_limit    = 65535
back_log = 1024
max_connections = 512
max_connect_errors = 1000000
table_open_cache = 1024
table_definition_cache = 1024
table_open_cache_instances = 64
thread_stack = 512K
external-locking = FALSE
max_allowed_packet = 32M
sort_buffer_size = 4M
join_buffer_size = 4M
thread_cache_size = 768
interactive_timeout = 600
wait_timeout = 600
tmp_table_size = 32M
max_heap_table_size = 32M
slow_query_log = 1
log_timestamps = SYSTEM
slow_query_log_file = /data/mysql/slow.log
log-error = /data/mysql/error.log
long_query_time = 0.1
log_queries_not_using_indexes =1
log_throttle_queries_not_using_indexes = 60
min_examined_row_limit = 100
log_slow_admin_statements = 1
log_slow_slave_statements = 1
server-id = 3306
log-bin = /data/mysql/mybinlog
sync_binlog = 1
binlog_cache_size = 4M
max_binlog_cache_size = 2G
max_binlog_size = 1G
expire_logs_days = 7
master_info_repository = TABLE
relay_log_info_repository = TABLE
gtid_mode = on
enforce_gtid_consistency = 1
log_slave_updates
slave-rows-search-algorithms = 'INDEX_SCAN,HASH_SCAN'
binlog_format = row
binlog_checksum = 1
relay_log_recovery = 1
relay-log-purge = 1
key_buffer_size = 32M
read_buffer_size = 8M
read_rnd_buffer_size = 4M
bulk_insert_buffer_size = 64M
myisam_sort_buffer_size = 128M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1
lock_wait_timeout = 3600
explicit_defaults_for_timestamp = 1
innodb_thread_concurrency = 0
innodb_sync_spin_loops = 100
innodb_spin_wait_delay = 30

transaction_isolation = REPEATABLE-READ
#innodb_additional_mem_pool_size = 16M
innodb_buffer_pool_size = 1434M
innodb_buffer_pool_instances = 4
innodb_buffer_pool_load_at_startup = 1
innodb_buffer_pool_dump_at_shutdown = 1
innodb_data_file_path = ibdata1:1G:autoextend
innodb_flush_log_at_trx_commit = 1
innodb_log_buffer_size = 32M
innodb_log_file_size = 2G
innodb_log_files_in_group = 2
innodb_max_undo_log_size = 4G
innodb_undo_directory = /data/mysql/undolog
innodb_undo_tablespaces = 95

# 根据您的服务器IOPS能力适当调整
# 一般配普通SSD盘的话，可以调整到 10000 - 20000
# 配置高端PCIe SSD卡的话，则可以调整的更高，比如 50000 - 80000
innodb_io_capacity = 4000
innodb_io_capacity_max = 8000
innodb_flush_sync = 0
innodb_flush_neighbors = 0
innodb_write_io_threads = 8
innodb_read_io_threads = 8
innodb_purge_threads = 4
innodb_page_cleaners = 4
innodb_open_files = 65535
innodb_max_dirty_pages_pct = 50
innodb_flush_method = O_DIRECT
innodb_lru_scan_depth = 4000
innodb_checksum_algorithm = crc32
innodb_lock_wait_timeout = 10
innodb_rollback_on_timeout = 1
innodb_print_all_deadlocks = 1
innodb_file_per_table = 1
innodb_online_alter_log_max_size = 4G
internal_tmp_disk_storage_engine = InnoDB
innodb_stats_on_metadata = 0

# some var for MySQL 5.7
innodb_checksums = 1
#innodb_file_format = Barracuda
#innodb_file_format_max = Barracuda
query_cache_size = 0
query_cache_type = 0
innodb_undo_logs = 128

innodb_status_file = 1
# 注意: 开启 innodb_status_output & innodb_status_output_locks 后, 可能会导致log-error文件增长较快
innodb_status_output = 0
innodb_status_output_locks = 0

#performance_schema
performance_schema = 1
performance_schema_instrument = '%memory%=on'
performance_schema_instrument = '%lock%=on'

#innodb monitor
innodb_monitor_enable="module_innodb"
innodb_monitor_enable="module_server"
innodb_monitor_enable="module_dml"
innodb_monitor_enable="module_ddl"
innodb_monitor_enable="module_trx"
innodb_monitor_enable="module_os"
innodb_monitor_enable="module_purge"
innodb_monitor_enable="module_log"
innodb_monitor_enable="module_lock"
innodb_monitor_enable="module_buffer"
innodb_monitor_enable="module_index"
innodb_monitor_enable="module_ibuf_system"
innodb_monitor_enable="module_buffer_page"
innodb_monitor_enable="module_adaptive_hash"

[mysqldump]
quick
max_allowed_packet = 32M


```

