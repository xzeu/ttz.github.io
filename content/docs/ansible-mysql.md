---
title: "Ansible Playbook快速部署一主多从MySQL集群"
date: 2023-08-07T13:49:05+08:00
lastmod: 2023-08-07T13:49:05+08:00
draft: false
keywords: [ansible,mysql]
description: ""
tags: [教程笔记,ansible,mysql,linux]
categories: [ansible]
author: "xzeu"

# Uncomment to pin article to front page
# weight: 1
# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: true
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: '<a rel="license noopener" href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0 / 转载文章请保留链接。</a>'
reward: false
mathjax: true

# Uncomment to add to the homepage's dropdown menu; weight = order of article
# menu:
#   main:
#     parent: "docs"
#     weight: 1
---
- [1. 部署目标](#1-部署目标)
- [2. 部署清单目录结构](#2-部署清单目录结构)
- [3. 主机清单](#3-主机清单)
- [4. 声明变量](#4-声明变量)
- [5. 交互式配置文件](#5-交互式配置文件)
- [6. 部署清单](#6-部署清单)
- [7. MySQL配置文件](#7-mysql配置文件)
- [8. 部署](#8-部署)
- [9. 验证](#9-验证)
  - [9.1. 登录节点查看状态](#91-登录节点查看状态)
  - [9.2. 主节点建库，从节点查看同步正常](#92-主节点建库从节点查看同步正常)

<!--more-->

## 1. 部署目标

> 1、快速部署一套一主两从的mysql集群
> 2、部署过程中支持交互式定义安装目录及监听端口号

## 2. 部署清单目录结构

```bash
root@master:/opt/mysql# tree .
.
├── group_vars
│   └── all.yml
├── hosts
├── mysql.yml
└── roles
    └── mysql
        ├── files
        │   └── mysql-5.7.33-linux-glibc2.12-x86_64.tar.gz
        ├── tasks
        │   └── main.yml
        └── templates
            ├── my.cnf.j2
            └── mysqld.service.j2

6 directories, 7 files
```

## 3. 主机清单

定义了需要部署mysql的主机组、IP及设定mysql主机角色

```shell
# 主机清单
root@master:/opt/mysql# cat hosts 
[mysql]
192.168.16.140 slave=true
192.168.16.150 slave=true
192.168.16.129 master=true
```

## 4. 声明变量

声明变量的好处在于用户可以按需改这一个文件，而不需要挨个儿修改部署清单，同时可以在交互式中进行个性化安装设置。

```shell
# 声明变量
root@master:/opt/mysql# cat group_vars/all.yml 
mysql_pkg: mysql-5.7.33-linux-glibc2.12-x86_64.tar.gz
mysql_version: mysql-5.7.33-linux-glibc2.12-x86_64
mysql_install_path: /opt/soft
mysql_link: mysql
mysql_sock: /tmp/mysql.sock
mysql_port: 3306
mysql_root_passwd: Root_123^
repl_user: repl
repl_passwd: Repl_123^
user: mysql
group: mysql
master_ip: 192.168.16.129 
```

## 5. 交互式配置文件

```shell
root@master:/opt/mysql# cat mysql.yml 
---
- hosts: mysql
  gather_facts: false
  vars_prompt:
  - name: mysql_install_path
    prompt: 请输入mysql安装目录
    default: "/opt/soft"
    private: no
  - name: mysql_port
    prompt: 请输入mysql服务的端口
    default: 3306
    private: no
  - name: mysql_sock
    prompt: 请输入mysql服务的socket文件
    default: "/tmp/mysql.sock"
    private: no
  - name: mysql_root_passwd
    prompt: 请输入mysql的root密码
    default: Root_123^ 
    private: yes
  - name: repl_user
    prompt: 请输入复制用户名
    default: repl
    private: no
  - name: repl_passwd
    prompt: 请输入复制用户密码
    default: Repl_123^
    private: yes
  - name: user
    prompt: 请输入mysql服务的启动用户
    default: mysql
    private: no
  roles:
    - mysql
```

## 6. 部署清单

其实就是把手动部署MySQL集群的步骤通过Ansible的相关模块、playbook语法及条件判断进行组合，继而实现自动化部署的过程。原理很简单，但是其中涉及的关于操作MySQL的模块需要着重研究，还有关于template模板的用法也非常重要，在通过Ansible playbook部署一些更复杂的系统时，经常会用到使用template模板语法渲染不同的配置，实现更为复杂系统的部署。比如通过Ansible playbook离线部署Kubernetes集群。

```shell
root@master:/opt/mysql# cat roles/mysql/tasks/main.yml
---
- name: "1、创建{{ user }}用户"
  user:
    name: "{{ user }}"
    shell: /bin/bash

- name: "2、创建安装目录"
  file:
    path: "{{ mysql_install_path }}"
    state: directory
    owner: "{{ user }}"
    group: "{{ group }}"
    recurse: yes

- name:  "3、解压mysql二进制包"
  unarchive:
    src:  "{{ mysql_pkg }}"
    dest: "{{ mysql_install_path }}"
    owner: "{{ user }}"
    group: "{{ group }}"

- name: "4、创建数据目录"
  file:
    path: "{{ item }}"
    state: directory
    owner: "{{ user }}"
    group: "{{ group }}"
    recurse: yes
  with_items:
    - "{{ mysql_install_path }}/{{ mysql_version }}/data"
    - "{{ mysql_install_path }}/{{ mysql_version }}/undolog"

- name: "5、修改权限"
  command: chown -R "{{ user }}:{{ group }}" "{{ mysql_install_path }}"

- name: "6、创建链接文件"
  file:
    src: "{{ mysql_install_path }}/{{ mysql_version }}"
    dest: "{{ mysql_install_path }}/{{ mysql_link }}"
    owner: "{{ user }}"
    group: "{{ group }}"
    state: link

- name: "7、生成配置文件"
  template:
    src: my.cnf.j2
    dest: /etc/my.cnf

- name: "8、数据库初始化"
  shell: ./mysqld --initialize --user={{ user }} --basedir={{ mysql_install_path }}/{{ mysql_link }} --datadir={{ mysql_install_path }}/{{ mysql_link }}/data
  args:
    chdir: "{{ mysql_install_path }}/{{ mysql_link }}/bin"

- name: "9、注册初始密码"
  shell: cat error.log |grep localhost|grep "temporary password"|awk '{print $NF}'
  register: mysql_init_passwd
  args:  
    chdir: "{{ mysql_install_path }}/{{ mysql_link }}/data"

- name: "10、打印初始密码"
  debug: 
    msg: "{{ mysql_init_passwd.stdout }}"

- name: "11、配置systemd守护进程"
  template:
    src: mysqld.service.j2
    dest: /usr/lib/systemd/system/mysqld.service

- name: "12、启动mysqld服务"
  systemd:
    name: mysqld
    state: started
    daemon_reload: yes
    enabled: yes

- name: "13、修改初始密码"
  shell: ./mysqladmin -u root -p"{{ mysql_init_passwd.stdout }}" password "{{ mysql_root_passwd }}"
  args:
    chdir: "{{ mysql_install_path }}/{{ mysql_link }}/bin"

- name: "14、创建{{ repl_user }}同步用户"
  mysql_user: 
    login_host: localhost
    login_port: "{{ mysql_port }}"
    login_user: root
    login_unix_socket: "{{ mysql_sock }}" 
    login_password: "{{ mysql_root_passwd }}"
    name: "{{ repl_user }}"
    password: "{{ repl_passwd }}"
    priv: "*.*:ALL"
    state: present 
    host: "%"
  when: master is defined

- name: "15、从库配置从主库同步"
  mysql_replication:
    login_unix_socket: "{{ mysql_sock }}"
    login_host: localhost
    login_port: "{{ mysql_port }}"
    login_user: root   
    login_password: "{{ mysql_root_passwd }}"
    master_host: "{{ master_ip }}" 
    master_user: "{{ repl_user }}" 
    master_password: "{{ repl_passwd }}"
    master_port: "{{ mysql_port }}"
    master_auto_position: 1
    mode: changemaster
  when: slave is defined

- name: "16、Start Slave"
  mysql_replication: 
    login_unix_socket: "{{ mysql_sock }}"
    login_user: root 
    login_host: localhost
    login_port: "{{ mysql_port }}"
    login_password: "{{ mysql_root_passwd }}"
    mode: startslave
  when: slave is defined

- name: "17、注册复制状态"
  mysql_replication:
    login_host: localhost
    login_user: root
    login_port: "{{ mysql_port }}"
    login_password: "{{ mysql_root_passwd }}"
    login_unix_socket: "{{ mysql_sock }}"
    mode: getslave
  when: slave is defined
  register: info

- name: "18、打印复制状态信息"
  debug:
    msg: "Slave_IO_Running={{ info.Slave_IO_Running }}       Slave_SQL_Running={{ info.Slave_SQL_Running }}"
  when: slave is defined
```

## 7. MySQL配置文件

MySQL配置文件中的系统参数可以根据实际按需修改，以下配置只供参考，着重看一下文件中有标注的地方。

```shell
root@master:/opt/mysql# cat roles/mysql/templates/my.cnf.j2
[client]
port = {{ mysql_port }}
socket = {{ mysql_sock }}
default-character-set=utf8mb4

[mysqldump]
single-transaction

[mysqld]
port = {{ mysql_port }}
socket = {{ mysql_sock }}
character-set-server=utf8mb4
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
log_bin_trust_function_creators=1
innodb_flush_log_at_trx_commit=1
sync_binlog=1

gtid-mode = on
enforce_gtid_consistency
log-bin = on
log-slave-updates = on

#rpl_semi_sync_master_enabled=1
#rpl_semi_sync_master_timeout=1000
#rpl_semi_sync_slave_enabled=1


master_info_repository = TABLE
relay_log_info_repository = TABLE


replicate-ignore-table=mysql.failover_console

datadir={{ mysql_install_path }}/{{ mysql_link }}/data

# 设置主节点server-id=1，模式为读写；从节点server-id=2,模式为只读
{% if master is defined %}
server-id=1
#read-only=0                           
{% else %}
server-id=2
#read-only=1
{% endif %}


#relay_log_purge=0
log_timestamps=SYSTEM
lower_case_table_names=1
log_slave_updates=on

skip-name-resolve
#skip-networking
back_log = 600

slave_parallel_workers = 16
slave-parallel-type = LOGICAL_CLOCK
master_info_repository = TABLE
relay_log_info_repository = TABLE
relay_log_recovery = ON
slave_preserve_commit_order = 1

innodb_undo_directory={{ mysql_install_path }}/{{ mysql_link }}/undolog
innodb_undo_tablespaces=4
innodb_undo_logs=128
innodb_max_undo_log_size=512M
innodb_purge_rseg_truncate_frequency
innodb_undo_log_truncate=1

max_connections = 4000
max_connect_errors = 6000
open_files_limit = 1024
table_open_cache = 4096
table_open_cache_instances = 64
max_allowed_packet = 128M
binlog_cache_size = 32M
max_heap_table_size = 128M
tmp_table_size = 32M
read_buffer_size = 8M  
read_rnd_buffer_size = 8M  
sort_buffer_size = 8M  
join_buffer_size = 8M  
key_buffer_size = 8M  
thread_cache_size = 64
query_cache_type = 0
query_cache_size = 0
#query_cache_size = 16M  
#query_cache_limit = 8M
ft_min_word_len = 4
log_bin = mysql-bin
binlog_format = row
expire_logs_days = 15
log_error ={{ mysql_install_path }}/{{ mysql_link }}/data/error.log
slow_query_log = 1
long_query_time = 3
performance_schema = 0
explicit_defaults_for_timestamp
#lower_case_table_names = 1
skip-external-locking
default_storage_engine = InnoDB
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1
innodb_stats_persistent_sample_pages = 64
innodb_open_files = 10000
innodb_buffer_pool_size = 2G
innodb_write_io_threads = 24
innodb_read_io_threads = 24
innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_log_buffer_size = 64M
innodb_sort_buffer_size = 64M
innodb_log_file_size = 512M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 75
innodb_lock_wait_timeout = 120
#log_warnings=1
log_error_verbosity=1
#local-infile=0
#connection-control-failed-connections-threshold=10
#connection-control-min-connection-delay=10800
wait_timeout = 3600
interactive_timeout = 3600
innodb_temp_data_file_path = ibtmp1:200M:autoextend:max:5G
```

## 8. 部署

```shell
# 部署命令
root@master:/opt/mysql# ansible-playbook -i hosts mysql.yml
```

交互式配置，不输入即保持默认配置，如默认端口为3306，我自定义为33306，部署路径我自定义为/opt/software，其他的保持默认直接按回车

```shell
root@master:/opt/mysql# ansible-playbook -i hosts mysql.yml
请输入mysql安装目录 [/opt/soft]: /opt/software
请输入mysql服务的端口 [3306]: 33306
请输入mysql服务的socket文件 [/tmp/mysql.sock]:
请输入mysql的root密码 [Root 123~]:
请输入复制用户名[repl]:
请输入复制用广密码 [Rep] 123~]:
请输入mysql服务的启动用户 [mysql]:
```

获取到的初始密码

```shell
TASK [mysql :9、注册初始密码]  ****************************
changed: [192,168,16,129]
changed: [192.168.16.150]
changed: [192.168.16.140]

TASK [mysql : 10、打印初始密码]  ****************************
ok: [192.168.16.140] => {
    "msg":"EZfCR--5B2ui"
}
ok: [192.168.16.150] => {
    "msg":"9frskdQpr3#n"
}
ok: [192.168.16.129] => {
    "msg":"Vih.A%,px7Sp"
}
```

部署完成后的截图：

```shell
TASK [mysgl : 16、Start Slave] ************************************************
skipping: [192.168.16.129]
changed: [192.168,16,150]
changed: [192.168.16.140]

TASK [mysql : 17、注册复制状态] ************************************************
skipping: [192.168.16.129]
ok: [192.168.16.150]
ok: [192.168.16.140]

TASK [mysql :18、打印复制状态信息] ************************************************
ok: [192.168.16.140] => {
    "msg" : "Slave_IO_Running=Yes   Slave_SQL_Running=Yes"
}
ok: [192.168.16.150] => {
    "msg" : "Slave_IO_Running=Yes   Slave_SQL_Running=Yes"
}
Skipping : [192.168.16.129]

PLAY RECAP ************************************************
192.168.16.129  : ok=14 changed=12  unreachable=0   failed=0 rescued=0  ignored=0
192.168.16.140  : ok=14 changed=12  unreachable=0   failed=0 rescued=0  ignored=0
192.168.16.150  : ok=14 changed=12  unreachable=0   failed=0 rescued=0  ignored=0
```

可以看到，一主两从已成功部署完成，获取到的复制状态信息正常。

## 9. 验证

### 9.1. 登录节点查看状态

```shell
# 主节点
root@ubuntu:/opt/software# mysql -u root -pRoot_123^ -e "show master status;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000002 |      852 |              |                  | 07c31ccf-208c-11ee-abc5-000c29e5c1ce:1-3 |
+------------------+----------+--------------+------------------+------------------------------------------+

# 从节点1
root@work-01:/opt/software# mysql -u root -pRoot_123^ -e "show slave status\G;"
mysql: [Warning] Using a password on the command line interface can be insecure.
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.16.129
                  Master_User: repl
                  Master_Port: 33306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 852
               Relay_Log_File: work-01-relay-bin.000002
                Relay_Log_Pos: 1065
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

# 从节点2
root@work-02:/opt/software#  mysql -u root -pRoot_123^ -e "show slave status\G;"
mysql: [Warning] Using a password on the command line interface can be insecure.
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.16.129
                  Master_User: repl
                  Master_Port: 33306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 852
               Relay_Log_File: work-02-relay-bin.000002
                Relay_Log_Pos: 1065
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

```shell
root@ubuntu: /opt/software# mysql -u root -pRoot_123^ -e "show master status;"
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000002 |      852 |              |                  | 07c31ccf-208c-11ee-abc5-000c29e5c1ce:1-3 |
+------------------+----------+--------------+------------------+------------------------------------------+
```

### 9.2. 主节点建库，从节点查看同步正常

```shell

# 主节点建库
root@ubuntu:/opt/software# mysql -u root -pRoot_123^ -e "create database db_demo;"
mysql: [Warning] Using a password on the command line interface can be insecure.
# 从节点1查看
root@work-01:/opt/software# mysql -u root -pRoot_123^ -e "show databases;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db_demo            |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
# 从节点2查看
root@work-02:/opt/software# mysql -u root -pRoot_123^ -e "show databases;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db_demo            |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```
