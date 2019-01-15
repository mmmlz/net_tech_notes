# MySQL

## 概述

- 数据模型：
    - 层次模型：它将数据组织成一对多关系的结构，层次结构采用关键字来访问其中每一层次的每一部分。例如：人员组织架构
    - 网状模型：它用连接指令或指针来确定数据间的显式连接关系，是具有多对多类型的数据组织方式。多对多。例如：选课
    - 关系模型：它以记录组或数据表的形式组织数据，以便于利用各种地理实体与属性之间的关系进行存储和变换，不分层也无指针，是建立空间数据和属性数据之间关系的一种非常有效的数据组织方法。例如：学生、课程、老师的关系。
- 数据分类，是对存储形式的一种数据类型分析
    - 结构化数据：可以存在excel中的数据
    - 半结构化数据：邮件、报表等
    - 非结构化数据：音频、视频等
- SQL接口：Structured Query Language
    - 类似于OS的shell接口
    - 分为两类：
        - DDL，Data Definition Language：`CREATE, ALTER, DROP, SHOW`
        - DML，Data Manipulation Language：`INSERT DELETE UPDATE SELECT`
    - 代码类型：
        - 存储过程：procedure
        - 存储函数：function
        - 触发器：trigger
        - 时间调度器：event scheduler
    - 用户和权限：
        - 用户：认证
        - 权限：管理类、程序类、数据库、表、字段
- 事务（Transaction）：组织多个操作为一个整体，要么全部都执行，要么全部都不执行；
    - ACID，是指数据库管理系统（DBMS）在写入或更新数据过程中，為保證事务（transaction）是正確可靠的，所必須具備的四个特性：
        - 原子性（atomicity，或稱不可分割性）
        - 一致性（consistency）
        - 隔离性（isolation，又称独立性）
        - 持久性（durability）

## MySQL

- MySQL
    - 衍生版
        - MariaDB
        - Percona
        - AliDB
        - TiDB
    - 原生版
        - Community
        - Enterprise

- 安装和使用MariaDB：
    - 安装方式：
        1. rpm包：由OS的发行商、程序官方提供；
        2. 源码包；
        3. 通用二进制格式的程序包；

- MariaDB程序的组成：
    - Clinet：client --> mysql protocol --> server
        - mysql：CLI交互式客户端程序
        - mysqldump：备份工具
        - mysqladmin：管理工具
        - mysqlbinlog：
    - Server：
        - mysqld
        - mysqld_safe：建议运行服务端程序
        - mysqld_multi：多实例

- 三类套接字地址：
    - IPv4 TCP 3306
    - IPv6 TCP 3306
    - Unix Socket：`/var/lib/mysql.sock`和`/tmp/mysql.sock`

- 配置文件：ini风格，用一个文件为多个程序提供配置；
    - [mysql]
    - [mysqld]
    - [mysqld_safe]
    - [server]
    - [client]
    - [mysqldump]
    - ...
    - mysql的各类程序启动都读取不只一个配置文件，按顺序读取，且最后读取的为最终生效，使用命令`my_print_defaults`查看：
        ```
            Default options are read from the following files in the given order:
            /etc/mysql/my.cnf /etc/my.cnf ~/.my.cnf
            /etc/my.cnf + /etc/my.cnf.d/*
        ```

- 常用命令：`mysql [OPINTIONS] [database]`
    - 常用选项：
        - `-uUSERNAME, --user=name`：用户名，默认为root；
        - `-hHost, --host=name`：mysql服务器，默认为localhost；客户端连接服务端，服务器会反解客户的IP为主机名，关闭此功能`skip_name_resolve=ON`
        - `-pPASSWORD, --password[=PASSWORD]`：用户的密码，默认为空
        - `-P, --port=#`：mysql服务器监听的端口，默认为3306端口
        - `-S, --socket=name`：套接字文件路径
        - `-D, --database=name`：登录时，使用的默认库
        - `-e, --execute='CMD'`：登录数据库时，执行的命令
    - 注意：
        - mysql的用户帐号由两部分组成：`'USERNAME'@'HOST'`；其中HOST用于权限此用户可通过哪些远程主机链接当前的mysql服务；
        - HOST的表示方式，支持使用通配符；
            - `%`：匹配任意长度的任意字符，如：`172.16.%.% == 172.16.0.0/16`
            - `_`：匹配任意单个字符
    - 命令
        - 客户端命令
            - `use, (\u) Use another database. Takes database name as argument.`
            - `exit, (\q) Exit mysql. Same as quit.`
            - `delimiter（界定符）, (\d) Set statement delimiter.`
            - `go, (\g) Send command to mysql server.`
            - `ego, (\G) Send command to mysql server, display result vertically.`
            - `status, (\s) Get status information from the server.`
            - `clear, (\c) Clear the current input statement.`
            - `system, (\!) Execute a system shell command.`
            - `source, (\.) Execute an SQL script file. Takes a file name as an argument.`
        - 服务端命令
            - 获取帮助：`help contents`
            - Account Management
            - Administration
            - Data Definition
            - Data Manipulation
            - Data Types

- 数据类型：
    - 字符型
        - 定长字符
            - CHAR(#)：不区分字符大小写
            - BINARY(#)：区分字符大小写
        - 变长字符：多占一个或两个字符空间
            - VARCHAR(#)
            - VARBINARY(#)
        - 对象存储
            - TEXT：检索时，不区分大小写
            - BLOB：Binary Large OBject，区分大小写
        - 内置类型：SET, ENUM
    - 数值型
        - 精确数值
            - INT(TINIINT, SMALLINT, MEDIUMINT, INT, BIGINT)
            - DECIMAL
        - 近似数值
            - FLOAT
            - DOUBLE
    - 时间日期型
        - DATE
        - TIME
        - DATETIME
        - TIMESTAMP
        - YEAR(2), YEAR(4)
    - 注：字符集
        - `SHOW COLLATION;`：查看各个字符集下的排序规则
        - `SHOW CHARACTER SET;`：查看所有支持的字符集

- 字段数据修饰符：
    - NOT NULL
    - NULL
    - AUTO_INCREMENT：自加
    - DEFAULT value
    - PRIMARY KEY
    - UNIQUE KEY，可以为空

- DDL，Data Definition Language：`CREATE, ALTER, DROP, SHOW`
    - 数据库管理
        - CREATE：即在`/var/lib/mysql/`下的文件夹
            ```
                CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name;
                    [create_specification] ...

                create_specification:
                    [DEFAULT] CHARACTER SET [=] charset_name
                    | [DEFAULT] COLLATE [=] collation_name
            ```
        - ALTER：
            ```
                ALTER {DATABASE | SCHEMA} [db_name]
                    alter_specification ...
                alter_specification:
                    [DEFAULT] CHARACTER SET [=] charset_name
                  | [DEFAULT] COLLATE [=] collation_name
            ```
        - DROP：`DROP {DATABASE | SCHEMA} [IF EXISTS] db_name`
        - SHOW：`SHOW {DATABASES | SCHEMAS} [LIKE 'pattern' | WHERE expr]`
            - 查看数据库支持的所有存储引擎类型：`SHOW ENGINES;`
            
    - 表管理
        - CREATE：
            ```
                CREATE [TEMPORARY] TABLE [IF NOT EXISTS] [db_name.]tbl_name
                    (create_definition,...)
                    [table_options]
                create_definition:
                    字段：col_name data_type
                    键：
                        PRIMARY KEY [index_type] (index_col_name,...)
                        UNIQUE [INDEX|KEY] (index_col_name,...)
                        FOREIGN KEY (index_col_name,...)
                    索引：[INDEX|KEY] [index_name] (index_col_name,...)
                table_option:
                    ENGINE [=] engine_name
                    | [DEFAULT] CHARACTER SET [=] charset_name
                    | [DEFAULT] COLLATE [=] collation_name
                复制表结构：CREATE TABLE tbl_name LIKE db_name.tbl_name
                复制表数据：CREATE TABLE tbl_name select_statement
            ```
        - ALTER：
            ```
                ALTER [ONLINE | OFFLINE] [IGNORE] TABLE tbl_name
                    [alter_specification [, alter_specification] ...]
                    [partition_options]

                alter_specification:
                    字段：
                        添加：ADD [COLUMN] col_name column_definition [FIRST | AFTER col_name]
                        删除：DROP [COLUMN] col_name
                        修改：
                            CHANGE [COLUMN] old_col_name new_col_name column_definition [FIRST|AFTER col_name]
                            MODIFY [COLUMN] col_name column_definition [FIRST | AFTER col_name]
                    键：
                        添加：ADD [PRIMARY|UNIQUE|FOREIGN] KEY (index_col_name,...)
                        删除：DROP [PRIMARY KEY|FOREIGN KEY fk_symbol]
                    索引：
                        添加：ADD {INDEX|KEY} [index_name] (index_col_name,...)
                        删除：DROP {INDEX|KEY} index_name
                    表选项：ENGINE [=] engine_name
            ```
        - DROP：`DROP [TEMPORARY] TABLE [IF EXISTS] tbl_name [, tbl_name, [...]]`
        - SHOW：`SHOW [FULL] TABLES [{FROM | IN} db_name] [LIKE 'pattern' | WHERE expr]`
            - 查看某表的存储引擎类型：`SHOW TABLES STATUS [LIKE 'table_name']|[WHERE expr]`
            - 查看表上的索引信息：`SHOW INDEXES FROM table_name`

- DML，Data Manipulation Language：`INSERT DELETE UPDATE SELECT`

# END

- clock
    - 22.3 62 min
    - 22.5 8 min