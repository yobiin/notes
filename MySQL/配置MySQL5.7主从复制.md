### 1 环境准备

~~~shell
192.168.1.34 服务器A（主机）
192.168.1.35 服务器B（从机）
~~~

### 2 安装mysql

MySQL版本：

这里采用Server version: 5.7.35 MySQL Community Server (GPL)

我们把安装在“服务器A”的数据库称作“主数据库”、安装在“服务器B”的数据库称作“从数据库”。

#### 2.1 开放端口

确保服务器A与服务器B上的3306端口可以互访。

### 3 设置主库

> 进行下面的配置前，假设你已经在两台服务器AB上安装成功MySQL服务。

192.168.1.34 服务器A（主机）

#### 3.1 修改MySQL配置文件

编辑mysql配置文件/etc/my.cnf，添加如下的内容：

~~~shell
[mysqld]
log-bin=mysql-bin
server-id=1
~~~

保存修改内容并重启MySQL服务。

#### 3.2 创建一个用户用于复制

> 所谓复制用户，即供从数据库使用的，从主数据库拷贝二进制日志信息的用户。

用客户端连接上MySQL服务，执行以下语句创建一个用于复制的用户：

~~~shell
mysql> CREATE USER 'repl'@'192.168.1.%' IDENTIFIED BY 'Abc!@#123';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.1.%';
~~~

#### 3.3 获取二进制文件位置

> 这些信息在后面配置从数据库时用到。

1、阻塞数据库写语句

~~~shell
mysql> FLUSH TABLES WITH READ LOCK;
~~~

2、获取当前二进制文件的名称和位置

~~~shell
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |     1030 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
~~~

#### 3.4 使用mysqldump创建数据快照

> 本指南默认使用[`InnoDB`](https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html)存储引擎。

如果在启用主从复制前，主数据库已存在数据，那么你就需要先把这些数据拷贝到从数据库。下面给出一个例子，将所有数据库备份到一个叫做`dbdump.db`的文件：

~~~shell
shell> mysqldump --all-databases --master-data > dbdump.db
~~~

其中`--master-data`选项会自动追加 [`CHANGE MASTER TO`](https://dev.mysql.com/doc/refman/5.7/en/change-master-to.html)语句，该语句在从数据库启动复制进程时需要到。

> 注意：
>
> 如果你没有使用`--master-data`选项的话，那么你需要在一个单独的会话中锁定所有数据表。详情见 [Section 16.1.2.3, “Obtaining the Replication Source's Binary Log Coordinates”](https://dev.mysql.com/doc/refman/5.7/en/replication-howto-masterstatus.html).

### 4 设置从库

192.168.1.35 服务器B（从机）

#### 4.1 修改MySQL配置文件

编辑mysql配置文件/etc/my.cnf，添加如下的内容：

~~~shell
[mysqld]
server-id=2
skip_slave_start=ON
~~~

这里主要解释一下`skip_slave_start`配置，ON表示数据库启动时不启动从机的复制进程，需要通过手动的方式进行启动。

保存修改内容并重启MySQL服务。

#### 4.2 设置从库对应的主库

这里主要用到[ CHANGE MASTER TO](https://dev.mysql.com/doc/refman/5.7/en/change-master-to.html)语句，其基本语法如下：

~~~shell
mysql> CHANGE MASTER TO
    ->     MASTER_HOST='source_host_name',
    ->     MASTER_USER='replication_user_name',
    ->     MASTER_PASSWORD='replication_password',
    ->     MASTER_LOG_FILE='recorded_log_file_name',
    ->     MASTER_LOG_POS=recorded_log_position;
~~~

我们的例子对应如下（当中的参数见上文[设置主数据库](#3 设置主数据库)一节）：

~~~shell
mysql> CHANGE MASTER TO
    ->     MASTER_HOST='192.168.1.34',
    ->     MASTER_USER='repl',
    ->     MASTER_PASSWORD='Abc!@#123',
    ->     MASTER_LOG_FILE='mysql-bin.000001',
    ->     MASTER_LOG_POS=1030;
~~~

#### 4.3 导入全新的数据的情况

即主库和备库都是新建的，不存在旧库旧表旧数据，且主库已经启用二进制日志，备库已指定主库及当前位置。这时你想要将其它的数据库备份下来导入到当前的主备数据库中，你只需要在主库中执行导入命令即可（不能再备库执行导入语句）：

~~~shell
shell> mysql -h '192.168.1.34' < fulldb.dump
~~~

#### 4.4 主库已存在数据的情况

即主库在启用二进制日志前已存在数据，你在启用从库复制进程前需要把主库的备份快照导入到从库中，然后才能启用从库的复制进程。

1、新建主库快照

~~~shell
shell> mysqldump --all-databases --master-data > dbdump.db
~~~

2、导入到从库中

~~~shell
shell> mysql -h '192.168.1.35' < fulldb.dump
~~~

#### 4.5 启用从库复制进程

~~~shell
mysql> START SLAVE;
~~~

验证主从复制是否启动成功

~~~shell
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.1.34
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 2096
               Relay_Log_File: 192-relay-bin.000007
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
......
~~~

如上，只有`Slave_IO_Running`和`Slave_SQL_Running`同时都是`Yes`的时候表示主从复制配置成功。

### 5 互为主从设置

前面两个章节讲了如何配置主从复制，即服务器A的数据库作为主库，服务器B的数据库作为从库；那么只要按照相反的步骤再设置一遍，即服务器B的数据库作为主库，服务器A的数据库作为从库。这样就可以使得服务器A的数据库和服务器B的数据库互为主从了。

