## 配置MySQL5.7主从复制

### 1 准备两台服务器

~~~shell
192.168.1.34 服务器A（主机）
192.168.1.35 服务器B（从机）
~~~

### 2 安装MySQL服务

MySQL版本：

这里采用Server version: 5.7.35 MySQL Community Server (GPL)

我们把安装在“服务器A”的数据库称作“主数据库”、安装在“服务器B”的数据库称作“从数据库”。

### 3 设置主数据库

> 进行下面的配置前，假设你已经在两台服务器AB上安装成功MySQL服务。

#### 3.1 开启二进制日志

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

如果在启用主从复制前，主数据库已存在数据，那么你就需要先把这些数据拷贝到从数据库中。下面给出一个例子，将所有数据库备份到一个叫做`dbdump.db`的文件：

~~~shell
shell> mysqldump --all-databases --master-data > dbdump.db
~~~

其中`--master-data`选项会自动追加 [`CHANGE MASTER TO`](https://dev.mysql.com/doc/refman/5.7/en/change-master-to.html)语句，该语句在从数据库启动复制进程时需要到。

> 注意：
>
> 如果你没有使用`--master-data`选项的话，那么你需要在一个单独的会话中锁定所有数据表。详情见 [Section 16.1.2.3, “Obtaining the Replication Source's Binary Log Coordinates”](https://dev.mysql.com/doc/refman/5.7/en/replication-howto-masterstatus.html).

### 4 设置从数据库



