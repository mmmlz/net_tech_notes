# MySQL

## 数据的备份

- 时间点恢复：binary logs; 
        
- 备份类型：
    - 备份的数据集的范围：完全备份和部分备份
        - 完全备份：整个数据集；
        - 部分备份：数据集的一部分，比如部分表；
    - 全量备份、增量备份、差异备份：
        - 完全备份
        - 增量备份：仅备份自上一次完全备份或增量备份以来变量的那部数据；
        - 差异备份：仅备份自上一次完全备份以来变量的那部数据；
    - 物理备份、逻辑备份：
        - 物理备份：复制数据文件进行的备份；
        - 逻辑备份：从数据库导出数据另存在一个或多个文件中；
    - 根据数据服务是否在线：
        - 热备：读写操作均可进行的状态下所做的备份；
        - 温备：可读但不可写状态下进行的备份；
        - 冷备：读写操作均不可进行的状态下所做的备份；

- 备份需要考虑因素：
    - 锁定资源多长时间？
    - 备份过程的时长？
    - 备份时的服务器负载？
    - 恢复过程的时长？

- 备份策略：
    - 全量+差异 + binlogs
    - 全量+增量 + binlogs
    - 完全+binlog：mysqldump
    - 备份手段：物理、逻辑
            
- 备份什么？
    - 数据
    - 二进制日志、InnoDB的事务日志；
    - 代码（存储过程、存储函数、触发器、事件调度器）
    - 服务器的配置文件

- 备份工具：
    - mysqldump：mysql服务自带的备份工具；逻辑备份工具；
        - 完全、部分备份；
        - InnoDB：热备；
        - MyISAM：温备；
    - cp/tar
        - lvm2：快照（请求一个全局锁），之后立即释放锁，达到几乎热备的效果；物理备份；
        - 注意：不能仅备份数据文件；要同时备份事务日志；要求数据文件和事务日志位于同一个逻辑卷；
    - xtrabackup：由Percona提供，开源工具，支持对InnoDB做热备，物理备份工具；
        - 完全备份、部分备份；
        - 完全备份、增量备份；
        - 完全备份、差异备份；
    - mysqlhotcopy
    - select：
        - 备份：SELECT cluase INTO OUTFILE 'FILENAME';
        - 恢复：CREATE TABLE
        - 导入：LOAD DATA

- mysqldump：
    - 二次封装工具：mydumper和phpMyAdmin
    - 用法：
        ```
            mysqldump [OPTIONS] database [tables]
                OR     mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]
                OR     mysqldump [OPTIONS] --all-databases [OPTIONS]

            mysqldump mydb：表级别备份
            mysqldump --databases mydb：库级别备份
            e.g.. mysqldump -uUSERNAME -pPASSWORD --single-transaction DB_NAME > FILENAME.sql
        ```
    - MyISAM存储引擎：支持温备，备份时要锁定表；
        - -x, --lock-all-tables：锁定所有库的所有表，读锁；
        - -l, --lock-tables：锁定指定库所有表；
    - InnoDB存储引擎：支持温备和热备；
        - --single-transaction：创建一个事务，基于此快照执行备份；
    - 其它选项：
        - -R, --routines：存储过程和存储函数；
        - --triggers：备份触发器
        - -E, --events：事件调度器
        - --master-data[=#]
            1：记录为CHANGE MASTER TO语句，此语句不被注释；
            2：记录为CHANGE MASTER TO语句，此语句被注释；
        - --flush-logs：锁定表完成后，即进行日志刷新操作；

- 基于lvm2的备份：
    - 0.要求数据文件和事务日志位于同一个逻辑卷；
    - 1.请求锁定所有表：`mysql> FLUSH TABLES WITH READ LOCK;`
    - 2.记录二进制文件事件位置：
        ```
            mysql> FLUSH LOGS;
            mysql> SHOW MASTER STATUS;
            mysql  -e  'SHOW MASTER STATUS;' >> /PATH/TO/SOME_POS_FILE
        ```
    - 3.创建快照卷：`lvcreate  -L # -s -p r - SNAM-NAME /dev/VG-NAME/LV-NAME `
    - 4.释放锁：`mysql> UNLOCK TABLES`
    - 5.挂载快照卷，并执行备份，备份完成后删除快照卷；
    - 6.周期性备份二进制日志； 

- Xtrabackup：
    - [xtrabackup命令用法实战](https://blog.csdn.net/wfs1994/article/details/80399408 "xtrabackup命令用法实战")
    - [innobackupex命令用法实战](https://blog.csdn.net/wfs1994/article/details/80398234 "innobackupex命令用法实战")


## MySQL Replication

### 主从复制

- 参考：[MySql 主从复制及配置实现](https://segmentfault.com/a/1190000008942618 "MySql 主从复制及配置实现")

- Master/Slave
    - Master: write/read
    - Slaves: read

- 主从复制的目的：
    - 冗余：promte（提升为主），异地灾备，人工；工具程序：MHA
    - 负载均衡：转移一部分“读”请求；
    - 支援安全的备份操作；
    - 测试；

- 主/从架构：
    - 异步复制：
    - 半同步复制：
    - 一主多从；
    - 一从一主；
    - 级联复制：一个主，带多个从；找一个代理节点，从主节点那道数据，复制给多个从
    - 循环复制；
    - 双主复制；
    - 一从多主：每个主服务器提供不同的数据库；

- 配置：
    - 时间同步；
    - 复制的开始位置：
        - 从0开始；
        - 从主服务器备份中恢复到从服务器节点后启动的复制，复制的起始点备份操作时主节点所处的日志文件及其事件位置；
    - 主从服务器mysqld程序版本不一致？从服务器的版本号可以高于主的版本号。
    - 主服务器：
        - 配置文件my.cnf
            - server_id=#
            - log_bin=log-bin
        - 启动服务：
        - 授权从节点连接主节点的账户：
            - `mysql> GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'USERNAME'@'HOST' IDENTIFIED BY 'YOUR_PASSWORD';`
            - `mysql> FLUSH PRIVILEGES;`
        - 查看主节点的二进制文件位置，便于从服务器找到日志位置
            - `mysql> SHOW MASTER STATUS;`
    - 从服务器：
        - 配置文件my.cnf
            - server_id=#
            - relay_log=relay-log
            - read_only=ON
        - 连接主节点：
            - `mysql> CHANGE MASTER TO MASTER_HOST='HOST',MASTER_USER='USERNAME',MASTER_PASSWORD='YOUR_PASSWORD',MASTER_LOG_FILE='BINLOG',MASTER_LOG_POS=#;`
        - 启动服务：
            - `mysql> START SLAVE [IO_THREAD|SQL_THREAD];`
            - `mysql> SHOW SLAVE STATUS;`
        - 课外作业：基于SSL的复制的实现；

### 主主复制

- 互为主从：两个节点各自都要开启binlog和relay log；
    - 可能遇到的问题：
        - 1.数据不一致；
        - 2.自动增长id；
            - 定义一个节点使用奇数id
                - auto_increment_offset=1
                - auto_increment_increment=2
            - 另一个节点使用偶数id
                - auto_increment_offset=2
                - auto_increment_increment=2

- 配置：
    - 1、server_id必须要使用不同值；
    - 2、均启用binlog和relay log；
    - 3、存在自动增长id的表，为了使得id不相冲突，需要定义其自动增长方式；
    - 服务启动后执行如下两步：
    - 4、都授权有复制权限的用户账号；
    - 5、各把对方指定为主节点；

- 复制时应该注意的问题：
    - 1、从服务设定为“只读”；
        - 在从服务器启动read_only，但仅对非SUPER权限的用户有效；
        - 阻止所有用户：
            - `mysql> FLUSH TABLES WITH READ LOCK;`
    - 2、尽量确保复制时的事务安全
        - 在master节点启用参数：`sync_binlog = ON`
        - 如果用到的是InnoDB存储引擎：
            - `innodb_flush_logs_at_trx_commit=ON`
            - `innodb_support_xa=ON`
    - 3、从服务器意外中止时尽量避免自动启动复制线程
    - 4、从节点：设置参数
        - `sync_master_info=ON`
        - `sync_relay_log_info=ON`

### 半同步复制

- 异步复制, 全同步复制和半同步复制
    - 异步复制（Asynchronous replication）：MySQL默认的复制即是异步的，主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理，这样就会有一个问题，主如果crash掉了，此时主上已经提交的事务可能并没有传到从上，如果此时，强行将从提升为主，可能导致新主上的数据不完整。
    - 全同步复制（Fully synchronous replication）：指当主库执行完一个事务，所有的从库都执行了该事务才返回给客户端。因为需要等待所有从库执行完该事务才能返回，所以全同步复制的性能必然会收到严重的影响。
    - 半同步复制（Semisynchronous replication）介于异步复制和全同步复制之间，主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待至少一个从库接收到并写到relay log中才返回给客户端。相对于异步复制，半同步复制提高了数据的安全性，同时它也造成了一定程度的延迟，这个延迟最少是一个TCP/IP往返的时间。所以，半同步复制最好在低延时的网络中使用。

- 支持多种插件：/usr/lib64/mysql/plugins/

- 需要安装方可使用：
    - `mysql> INSTALL PLUGIN plugin_name SONAME 'shared_library_name';`

- 半同步复制需要的插件`semisync_master.so`和`semisync_slave.so`

- 主节点：
    ```
        INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
        
            MariaDB [mydb]> SHOW GLOBAL VARIABLES LIKE 'rpl_semi%';
            +------------------------------------+-------+
            | Variable_name                      | Value |
            +------------------------------------+-------+
            | rpl_semi_sync_master_enabled       | OFF   |
            | rpl_semi_sync_master_timeout       | 10000 |
            | rpl_semi_sync_master_trace_level   | 32    |
            | rpl_semi_sync_master_wait_no_slave | ON    |
            +------------------------------------+-------+            
          
        MariaDB [mydb]> SET GLOBAL rpl_semi_sync_master_enabled=ON;    
    ```

- 从节点：
    ```
        INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
        
            MariaDB [mydb]> SHOW GLOBAL VARIABLES LIKE 'rpl_semi%';                        
            +---------------------------------+-------+
            | Variable_name                   | Value |
            +---------------------------------+-------+
            | rpl_semi_sync_slave_enabled     | OFF   |
            | rpl_semi_sync_slave_trace_level | 32    |
            +---------------------------------+-------+            
        
        MariaDB [mydb]> STOP SLAVE IO_THREAD;
        MariaDB [mydb]> SHOW GLOBAL VARIABLES LIKE 'rpl_semi%';
        MariaDB [mydb]> START SLAVE IO_THREAD;
    ```

- 主节点判断方法：`MariaDB [mydb]> SELECT @@global.rpl_semi_sync_master_clients；`

### 复制过滤器

- 仅复制有限一个或几个数据库相关的数据，而非所有；由复制过滤器进行；

- 有两种实现思路：
    - (1) 主服务器
        - 主服务器仅向二进制日志中记录有关特定数据库相关的写操作；过滤仅可以到达库级别
        - 问题：其它库的time-point recovery将无从实现；
        - 修改参数：
            - binlog_do_db=
            - binlog_ignore_db=
    - (2) 从服务器
        - 从服务器的SQL THREAD仅重放关注的数据库或表相关的事件，并将其应用于本地；过滤可以到达表级别
        - 问题：浪费网络IO和磁盘IO；
        - 修改参数：
            - Replicate_Do_DB=
            - Replicate_Ignore_DB=
            - Replicate_Do_Table=
            - Replicate_Ignore_Table=
            - Replicate_Wild_Do_Table=
            - Replicate_Wild_Ignore_Table=

- What's more：基于SSL复制的实现


### 复制的监控和维护

- (1) 清理日志：PURGE
    `PURGE { BINARY | MASTER } LOGS { TO 'log_name' | BEFORE datetime_expr };`
- (2) 复制监控
    - MASTER
        ```
            SHOW MASTER STATUS;
            SHOW BINLOG EVENTS;
            SHOW BINARY LOGS;
        ```
    - SLAVE：`SHOW SLAVE STATUS;`
    - 判断从服务器是否落后于主服务器：`Seconds_Behind_Master: 0`
- (3) 如何确定主从节点数据是否一致？
    - 通过表的CHECKSUM检查；
    - 使用percona-tools中pt-table-checksum；
- (4) 主从数据不一致时的修复方法？重新复制；


### 主从复制的读写分离

- [MySQL主从复制与读写分离](https://www.jianshu.com/p/127d6aeab276 "MySQL主从复制与读写分离")


#### ProxySQL

- ProxySQL：http://www.proxysql.com/, ProxySQL is a high performance, high availability, protocol aware proxy for MySQL and forks (like Percona Server and MariaDB).

- 配置示例：
    ```
        datadir="/var/lib/proxysql"
        admin_variables=
            {
                admin_credentials="admin:admin"
                mysql_ifaces="127.0.0.1:6032;/tmp/proxysql_admin.sock"
            }
        mysql_variables=
            {
                threads=4
                max_connections=2048
                default_query_delay=0
                default_query_timeout=36000000
                have_compress=true
                poll_timeout=2000
                interfaces="0.0.0.0:3306;/tmp/mysql.sock"
                default_schema="mydb"
                stacksize=1048576
                server_version="5.5.30"
                connect_timeout_server=3000
                monitor_history=600000
                monitor_connect_interval=60000
                monitor_ping_interval=10000
                monitor_read_only_interval=1500
                monitor_read_only_timeout=500
                ping_interval_server=120000
                ping_timeout_server=500
                commands_stats=true
                sessions_sort=true
                connect_retries_on_failure=10
            }
        mysql_servers =
            (
                {
                    address = "172.18.0.67" # no default, required . If port is 0 , address is interpred as a Unix Socket Domain
                    port = 3306           # no default, required . If port is 0 , address is interpred as a Unix Socket Domain
                    hostgroup = 0           # no default, required
                    status = "ONLINE"     # default: ONLINE
                    weight = 1            # default: 1
                    compression = 0       # default: 0
                },
                {
                    address = "172.18.0.68"
                    port = 3306
                    hostgroup = 1
                    status = "ONLINE"     # default: ONLINE
                    weight = 1            # default: 1
                    compression = 0       # default: 0
                },
                {
                    address = "172.18.0.69"
                    port = 3306
                    hostgroup = 1
                    status = "ONLINE"     # default: ONLINE
                    weight = 1            # default: 1
                    compression = 0       # default: 0
                }
            )
        mysql_users:
            (
                {
                    username = "root"
                    password = "mageedu"
                    default_hostgroup = 0
                    max_connections=1000
                    default_schema="mydb"
                    active = 1
                }
            )
        mysql_query_rules:()
        scheduler=()
        mysql_replication_hostgroups=
            (
                {
                    writer_hostgroup=0
                    reader_hostgroup=1
                    comment='描述信息'
                }
            )
    ```
- 可以查看`monitor`库查看相关配置信息以及使用状态信息；

#### maxscale

- maxscale配置示例：
    ```
        [maxscale]
        threads=auto
        
        [server1]
        type=server
        address=172.18.0.67
        port=3306
        protocol=MySQLBackend
        
        [server2]
        type=server
        address=172.18.0.68
        port=3306
        protocol=MySQLBackend
        
        [server3]
        type=server
        address=172.18.0.69
        port=3306
        protocol=MySQLBackend
        
        [MySQL Monitor]
        type=monitor
        module=mysqlmon
        servers=server1,server2,server3
        user=maxscale
        passwd=201221DC8FC5A49EA50F417A939A1302
        monitor_interval=1000
        
        [Read-Only Service]
        type=service
        router=readconnroute
        servers=server2,server3
        user=maxscale
        passwd=201221DC8FC5A49EA50F417A939A1302
        router_options=slave
        
        [Read-Write Service]
        type=service
        router=readwritesplit
        servers=server1
        user=maxscale
        passwd=201221DC8FC5A49EA50F417A939A1302
        max_slave_connections=100%
        
        [MaxAdmin Service]
        type=service
        router=cli
        
        [Read-Only Listener]
        type=listener
        service=Read-Only Service
        protocol=MySQLClient
        port=4008
        
        [Read-Write Listener]
        type=listener
        service=Read-Write Service
        protocol=MySQLClient
        port=4006
        
        [MaxAdmin Listener]
        type=listener
        service=MaxAdmin Service
        protocol=maxscaled
        port=6602            
    ```


### HMA

[MHA构建MySQL高可用平台最佳实践](https://www.jianshu.com/p/6173dae5ed7a "MHA构建MySQL高可用平台最佳实践")

- 当master出现故障时，通过对比slave之间I/O线程读取masterbinlog的位置，选取最接近的slave做为latestslave。 其它slave通过与latest slave对比生成差异中继日志。在latest slave上应用从master保存的binlog，同时将latest slave提升为master。最后在其它slave上应用相应的差异中继日志并开始从新的master开始复制。

- mysqlrouter：语句透明路由服务；MySQL Router 是轻量级 MySQL 中间件，提供应用与任意 MySQL 服务器后端的透明路由。MySQL Router 可以广泛应用在各种用案例中，比如通过高效路由数据库流量提供高可用性和可伸缩的 MySQL 服务器后端。Oracle 官方出品。
