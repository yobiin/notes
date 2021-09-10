## 1 环境准备

系统版本：CentOS Linux release 7.9.2009

MySQL版本：Server version: 5.7.35-log MySQL Community Server (GPL)

192.168.1.34 服务器（数据库）A
192.168.1.35 服务器（数据库）B

其中，数据库A和数据库B互为主从，配置方式参考[配置MySQL5.7主从复制](配置MySQL5.7主从复制.md)。

Keepalived 配置如下：

VIP 192.168.1.140

MASTER 192.168.1.34 服务器A

BACKUP 192.168.1.35 服务器B

## 2 Keepalived安装

这里通过`yum`命令方式在两台服务器上安装`keepalived`程序，执行安装命令：

~~~shell
shell> sudo yum install keepalived -y
~~~

`keepalived`版本信息：

~~~shell
shell> keepalived -v
Keepalived v1.3.5 (03/19,2017), git commit v1.3.5-6-g6fa32f2
Copyright(C) 2001-2017 Alexandre Cassen, <acassen@gmail.com>
Build options:  PIPE2 LIBNL3 RTA_ENCAP RTA_EXPIRES RTA_PREF RTA_VIA FRA_OIFNAME FRA_SUPPRESS_PREFIXLEN FRA_TUN_ID RTAX_CC_ALGO RTAX_QUICKACK LIBIPTC LIBIPSET_DYNAMIC LVS LIBIPVS_NETLINK VRRP VRRP_AUTH VRRP_VMAC SOCK_NONBLOCK SOCK_CLOEXEC FIB_ROUTING INET6_ADDR_GEN_MODE SNMP_V3_FOR_V2 SNMP SNMP_KEEPALIVED SNMP_CHECKER SNMP_RFC SNMP_RFCV2 SNMP_RFCV3 SO_MARK
~~~

源码安装方式请参考

官网：[Keepalived for Linux](https://www.keepalived.org/index.html)

GitHub：[acassen/keepalived: Keepalived (github.com)](https://github.com/acassen/keepalived)

## 3 编写Keepalived检查脚本

`keepalived`检查脚本的作用是用来监测MySQL的运行状态，返回0表示MySQL服务运行正常，返回1表示MySQL服务运行异常。脚本内容如下：

~~~shell
#!/bin/bash
pNo=$(ps -ef | grep -v grep | grep mysqld | wc -l)
listenNo=$(netstat -lnt | grep -v grep | grep 3306 | wc -l)
if [ $pNo -eq 1 -a $listenNo -eq 1 ]; then
    exit 0
else
    exit 1
fi
~~~

这里将脚本内容保存到`/etc/keepalived/bin/mysql_process_monitor.sh`文件中。

## 4 修改Keepalived配置文件

默认情况下`keepalived`的配置文件位于`/etc/keepalived`目录下，文件名为`keepalived.conf`。

192.168.1.34 服务器，`keepalived.conf`修改如下：

~~~shell
! Configuration File for keepalived

global_defs {
   notification_email {
     yobin20@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id 192.168.1.34
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_script mysql_monitor {
   script "/etc/keepalived/bin/mysql_process_monitor.sh"
}

vrrp_instance VI_1 {
    state MASTER
    interface eno16780032
    virtual_router_id 51
    priority 200
    advert_int 1
    # MASTER恢复后不抢占虚拟IP
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        mysql_monitor
    }
    virtual_ipaddress {
        192.168.1.40/24 dev eno16780032 label eno16780032:1
    }
}
~~~

192.168.1.35 服务器，`keepalived.conf`修改如下：

~~~shell
! Configuration File for keepalived

global_defs {
   notification_email {
     yobin20@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id 192.168.1.35
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_script mysql_monitor {
   script "/etc/keepalived/bin/mysql_process_monitor.sh"
}

vrrp_instance VI_1 {
    state BACKUP
    interface eno16780032
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        mysql_monitor
    }
    virtual_ipaddress {
        192.168.1.40/24 dev eno16780032 label eno16780032:1
    }
}
~~~

上述参数说明请参考[Keepalived Documentation](https://www.keepalived.org/manpage.html)。

## 5 添加防火墙规则

启动`keepalived`服务前，先添加两条防火墙策略

~~~shell
firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
firewall-cmd --direct --permanent --add-rule ipv4 filter OUTPUT 0 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
firewall-cmd --reload
~~~

> Keepalived使用vrrp组播，默认地址是224.0.0.18，因此要配置防火墙放过。

## 5 启动Keepalived服务

上述配置完成后，分别在192.168.1.34/35 服务器上执行以下命令：

~~~shell
# 1 将keepalived加入开机启动项
shell> systemctl enable keepalived
# 2 启动keepalived服务
shell> systemctl start keepalived
~~~

由于上述配置时，我们给`keepalived`MASTER设置了一个参数项`nopreempt`表示当MASTER重启后不抢占VIP。因此第一个启动`keeaplived`服务的服务器将持有VIP，使用`ip addr`命令查看如下：

~~~shell
shell> ip addr
......
2: eno16780032: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:70:a1:dd brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.35/24 brd 192.168.1.255 scope global eno16780032
       valid_lft forever preferred_lft forever
    inet 192.168.1.40/24 scope global secondary eno16780032:1
       valid_lft forever preferred_lft forever
    ......
~~~

如上，可以看到当前VIP`192.168.1.40`已生效，通过MySQL客户端使用VIP连接数据库，如下所示：

~~~shell
shell> mysql -h'192.168.1.40' -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 5.7.35-log MySQL Community Server (GPL)
Copyright (c) 2000, 2021, Oracle and/or its affiliates.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> 
~~~

如果像上面一样打印出信息则表示`keepalived`基本配置成功。

## 6 测试高可用性

MySQL双主热备配合`keepalived`，使得192.168.1.34/35中工作的数据库发生崩溃后，能够迅速切换到另外一台备用数据库，从而最大限度减小对业务的影响。

`keepalived`通过VIP漂移的方式，当检测到工作的MySQL数据库进程或者端口不可用的时候，就会把VIP切换并绑定到另一台可用的服务器上。

1 登录到当前VIP所在服务器，然后关闭MySQL服务：

~~~shell
shell> systemctl stop mysqld
# 查看ip地址，发现已经没有了VIP`192.168.1.40`
shell> ip addr
......
2: eno16780032: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:95:77:a7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.34/24 brd 192.168.1.255 scope global eno16780032
       valid_lft forever preferred_lft forever
    ......
~~~

2 登录到另外一台服务器，然后使用`ip addr`查看：

~~~shell
shell> ip addr
......
2: eno16780032: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:70:a1:dd brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.35/24 brd 192.168.1.255 scope global eno16780032
       valid_lft forever preferred_lft forever
    inet 192.168.1.40/24 scope global secondary eno16780032:1
       valid_lft forever preferred_lft forever
   ......
~~~

如果像上面一样发现VIP`192.168.1.40`则证明`keepalived`已经可以正常工作。

3 查看/var/log/messages日志可以看到`keepalived`打印的切换过程日志

~~~shell
Sep  8 15:41:12 192 Keepalived_vrrp[15938]: VRRP_Instance(VI_1) Transition to MASTER STATE
Sep  8 15:41:13 192 Keepalived_vrrp[15938]: VRRP_Instance(VI_1) Entering MASTER STATE
Sep  8 15:41:13 192 Keepalived_vrrp[15938]: VRRP_Instance(VI_1) setting protocol iptable drop rule
Sep  8 15:41:13 192 Keepalived_vrrp[15938]: VRRP_Instance(VI_1) setting protocol VIPs.
Sep  8 15:41:13 192 Keepalived_vrrp[15938]: Sending gratuitous ARP on eno16780032 for 192.168.1.40
Sep  8 15:41:13 192 Keepalived_vrrp[15938]: VRRP_Instance(VI_1) Sending/queueing gratuitous ARPs on eno16780032 for 192.168.1.40
~~~

