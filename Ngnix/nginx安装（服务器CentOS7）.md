### 一、通过yum安装nginx

#### 1、安装yum-utils

```
sudo yum install yum-utils
```

#### 2、配置yum仓库

新建一个文件/etc/yum.repos.d/nginx.repo，然后写入以下内容并保存

```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

#### 3、安装nginx

```
sudo yum install nginx
```

安装过程中提示导入GPG key的时候，检查指纹（fingerprint）是否和`573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62`一致，如果检查一致则接受并继续安装即可。

### 二、配置nginx

> 由于系统需要通过https协议进行访问，因此需要开启nginx的ssl模块。

#### 1、生成证书文件

参考《openssl生成证书》文档，根据步骤生成以下几个文件：

ca.crt  ca.key  server41.cer  server41.crt  server41.csr  server41.key

后面的配置会使用到server41.cer  server41.crt   server41.key等3个文件。

#### 2、修改nginx配置

打开/etc/nginx/conf.d/default.conf文件，将内容修改如下：

~~~shell
upstream srv.cas.com {
    ip_hash;
    server 192.168.1.31:21080;
    server 192.168.1.32:21080;
}

upstream srv.console.com {
    ip_hash;
    server 192.168.1.31:22080;
    server 192.168.1.32:22080;
}

upstream srv.sdh.com {
    ip_hash;
    server 192.168.1.31:23080;
    server 192.168.1.32:23080;
}

upstream srv.cmdb.com {
    ip_hash;
    server 192.168.1.31:24080;
    server 192.168.1.32:24080;
}

upstream srv.datav.com {
    ip_hash;
    server 192.168.1.31:25080;
    server 192.168.1.32:25080;
}

server {

    listen       80;
    listen       443 ssl;
    server_name  localhost;
    ssl_certificate     /root/server41.crt;
    ssl_certificate_key /root/server41.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root /data/www/html;
        index index.html index.htm;
    }
    
    location /cas {
        proxy_pass http://srv.cas.com;
        index  index.html index.htm;
    }
     
    location /console {
        proxy_pass http://srv.console.com;
        index  index.html index.htm;
    }

    location /sdh {
        proxy_pass http://srv.sdh.com;
        index  index.html index.htm;
    }

    location /cmdb {
        proxy_pass http://srv.cmdb.com;
        index  index.html index.htm;
    }

    location /datav {
        proxy_pass http://srv.datav.com;
        index  index.html index.htm;
    }

    location /images/ {
        root  /data/www/images;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /data/www/html;
    }
}
~~~

#### 3、启动nginx

执行以下命令即可：

~~~shell
nginx
~~~

更多关于`nginx`的操作请参考《Nginx初学者指南》以及《使用Nginx作为HTTP负载均衡器》文档说明。

### 三、将证书导入到Java的信任库

这里默认你已经安装了应用，然后将上一步生成的server41.cer文件拷贝到所有的应用服务器，然后把它导入到Java的证书信任库中。

不同版本Java的证书库位置，有所不同：

JDK1.8

~~~shell
./[JDK1.8_HOME]/jre/lib/security/cacerts
~~~

JDK11

~~~shell
./[JDK11_HOME]/lib/security/cacerts
~~~

导入命令如下：

~~~shell
./java/jdk1.8.0_202/bin/keytool -import  -keystore ./java/jdk-11/lib/security/cacerts -file /root/server41.cer
~~~

~~~shell
./java/jdk1.8.0_202/bin/keytool -import  -keystore ./java/jdk-11/lib/security/cacerts -file /root/server41.cer
~~~

> 注意：导入证书后需要重启Tomcat应用。

### 四、nginx高可用配置

这里采用keepalived + nginx方式达到高可用效果。

#### 1、nginx安装配置

测试环境中共有两台nginx服务：

~~~shell
192.168.1.30 nginx1
192.168.1.33 nginx2
~~~

以上两台服务器按照前面几个章节给出的步骤，分别安装并配置启动`nginx`，其中证书环节只需要生成一次即可。

#### 2、keepalived安装配置

分别在`192.168.1.30`,`192.168.1.33`两台服务器上安装keepalived，安装步骤请参考《通过Keepalived实现MySQL双主高可用》文档说明。

keepalived配置文件如下：

~~~shell
! Configuration File for keepalived

global_defs {
   notification_email {
     yobin20@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id 192.168.1.30
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_script chk_nginx {
   script "/etc/keepalived/bin/chk_nginx.sh"
}

vrrp_instance VI_1 {
    state BACKUP
    interface eno16780032
    virtual_router_id 61
    priority 100
    advert_int 1
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_nginx
    }
    virtual_ipaddress {
        192.168.1.41/24 dev eno16780032 label eno16780032:1
    }
}
~~~

和《通过Keepalived实现MySQL双主高可用》中配置MySQL高可用有以下几点不同：

1、这里的虚拟IP是：192.168.1.41

2、virtual_router_id 设置为 61，需要和MySQL的Keepalived的配置区别开来（标识同一局域网内两组不同的Keepalived）

3、nginx监听脚本chk_nginx.sh内容如下：

~~~
#!/bin/bash
ps aux | grep nginx | egrep '(master|worker)'
~~~

#### 3、测试高可用配置

> 开放端口：
>
> 其中`nginx`使用到443端口，`keepalived`需要在两台服务器防火墙开放`224.0.0.18`网段的`vrrp`组播策略，具体参考《通过Keepalived实现MySQL双主高可用》文档说明。

启动`nginx`和`keepalived`。

> 注意：以下的步骤可能由于配置先后或者软件启动先后不一样而有所不同，可根据实际情况做修改。

查看当前虚拟IP所在服务器，如下：

~~~shell
[root@192 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    ......
2: eno16780032: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:49:c2:d4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.33/24 brd 192.168.1.255 scope global eno16780032
       valid_lft forever preferred_lft forever
    inet 192.168.1.41/24 scope global secondary eno16780032:1
       valid_lft forever preferred_lft forever
    ......
~~~

如上可知，虚拟IP`192.168.1.41`当前在`192.168.1.33`服务器上。

① 停止`192.168.1.33`服务器上的`nginx`

~~~shell
nginx -s stop
~~~

再查看可知，此时虚拟IP`192.168.1.41`已经不在`192.168.1.33`这台服务器上了

~~~shell
[root@192 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
    ......
2: eno16780032: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:49:c2:d4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.33/24 brd 192.168.1.255 scope global eno16780032
       valid_lft forever preferred_lft forever
    ......
~~~

事实上，如果Keepalived配置是正确的且组播策略是开放了的，这是的虚拟IP应该是漂移到`192.168.1.30`这台服务器了

~~~shell
[root@192 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
    ......
2: eno16780032: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:d3:6c:d5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.30/24 brd 192.168.1.255 scope global eno16780032
       valid_lft forever preferred_lft forever
    inet 192.168.1.41/24 scope global secondary eno16780032:1
       valid_lft forever preferred_lft forever
    ......
~~~

② 重启`192.168.1.33`服务器上的`nginx`然后停止`192.168.1.30`服务器上的`nginx`，这是你会发现虚拟IP又漂移回`192.168.1.33`这台服务器上了。

至此nginx安装配置以及高可用配置完成。