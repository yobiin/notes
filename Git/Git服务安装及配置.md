### 1、Git安装

```
[root@VM-0-8-centos ~]# yum install git
```

### 2、新建Git用户

```
[root@VM-0-8-centos ~]# groupadd git
[root@VM-0-8-centos ~]# useradd git -g git
```

### 3、初始化一个Git仓库

```
[root@VM-0-8-centos ~]# mkdir -p /home/git/gitrepo
[root@VM-0-8-centos ~]# cd /home/git/gitrepo
[root@VM-0-8-centos ~]# git init --bare test.git
Initialized empty Git repository in /home/git/gitrepo/test.git/
$ chown -R git:git /home/git/gitrepo
```

### 4、克隆仓库

以下命令在window终端测试

```
PS E:\Workspace\GitTest> git clone git@42.14.83.95:/home/git/gitrepo/test.git
Cloning into 'test'...
git@42.194.183.95's password:【输入git密码】
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (3/3), 111.68 KiB | 37.23 MiB/s, done.
```

git为上面在服务器中新建的用户，42.14.83.95是服务器的IP地址，/home/git/gitrepo/test.git为前面新建的仓库。

至此，Git服务已经安装完成。

### 5、配置SSH访问

​	如果使用用户名密码操作服务器上的Git仓库，每次执行命令都需要输入密码，极其麻烦，且用户名密码容易泄露不安全。所以为了避免用户名密码带来的重复繁琐的操作和安全风险，使用SSH证书的方式操作Git服务器上的仓库是更好的一种习惯。

#### 生成SSH公钥和私钥

命令如下

```
PS C:\Users\ybliang> ssh-keygen -t rsa -C "yobin20@163.com"
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\ybliang/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\ybliang/.ssh/id_rsa.
Your public key has been saved in C:\Users\ybliang/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:KPBPIQ/gN7mn9LWHuFzXyTQwAKGo9nxAUuRup9QDcY4 yobin20@163.com
The key's randomart image is:
+---[RSA 2048]----+
|  o+ .oo.        |
| .oo=o   .       |
| .+EB..   o      |
| .++o* o   o     |
|.. ==+= S   o    |
|. =.+B.o o + o   |
|   +..+ + o +    |
|    .. o o       |
|      o          |
+----[SHA256]-----+
```

参数 -t rsa 表示使用rsa算法进行加密，执行后，会在C:\Users\ybliang\\.ssh目录下生成id_rsa(私钥)和id_rsa.pub(公钥)文件，如下所示：

```
PS C:\Users\ybliang\.ssh> dir

    目录: C:\Users\ybliang\.ssh

Mode                LastWriteTime         Length Name

----                -------------         ------ ----

-a----        2020/11/9     23:53           1675 id_rsa
-a----        2020/11/9     23:53            398 id_rsa.pub
```

#### 将公钥拷贝到Git服务器

查看C:\Users\ybliang\.ssh\id_rsa.pub公钥文件，如下所示：

```
PS C:\Users\ybliang\.ssh> cat .\id_rsa.pub

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCTZfxIEYDK35+MwbgsGV1TwdwcT3ST/U2Usek/veWaoIUJfZN5YGcRNhTevr6LlHuIOWPyY67AdgoyvO48ozba0QaF6by3dqKJ3MeipiJvOiuvz5Ubk2WEEN41D8sxxmWowWn1kQlyb/8Tf38fMPtdDyuV0YLhxcvgBUtSdHpBoPgSA4XCY1enlFsJJ0M4WFhYtUsOS7EhvAm37LbdArqToVH606Zr6cKovdet+fj+T1NKsgHBaL95mxJHNvc/sZpu3G+yLIXUxSbD2cqAfrxK1R1OIrwy3I6LyEdBs/xgwK8sL+D41suFl/QGqOagfmVvBXlu38xXK8HGxo0rYkvr yobin20@163.com
```

将控制台输出的字符串导入到Git服务器的/home/git/.ssh/authorized_keys文件中，一行一个，如果没有该文件则新建，如下所示：

```
[root@VM-0-8-centos git]# cd /home/git/
[root@VM-0-8-centos git]# mkdir .ssh
[root@VM-0-8-centos git]# touch .ssh/authorized_keys
[root@VM-0-8-centos git]# echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCTZfxIEYDK35+MwbgsGV1TwdwcT3ST/U2Usek/veWaoIUJfZN5YGcRNhTevr6LlHuIOWPyY67AdgoyvO48ozba0QaF6by3dqKJ3MeipiJvOiuvz5Ubk2WEEN41D8sxxmWowWn1kQlyb/8Tf38fMPtdDyuV0YLhxcvgBUtSdHpBoPgSA4XCY1enlFsJJ0M4WFhYtUsOS7EhvAm37LbdArqToVH606Zr6cKovdet+fj+T1NKsgHBaL95mxJHNvc/sZpu3G+yLIXUxSbD2cqAfrxK1R1OIrwy3I6LyEdBs/xgwK8sL+D41suFl/QGqOagfmVvBXlu38xXK8HGxo0rYkvr yobin20@163.com" >> .ssh/authorized_keys
[root@VM-0-8-centos git]# chown -R git:git .ssh
```

至此，Git的SSH认证登录已经配置完毕，现在可以不用每次都操作都要输入烦人的密码啦(*^▽^*)

### 6、配置http访问

> 想要通过http的方式访问GIT服务器中的仓库，需要通过web容器来启用GIT的http协议，其中web容器可以是Apache、lighthttpd或者nginx。下面将选用 nginx + fcgiwraper 组合来支持GIT服务器的http访问。

#### 安装nginx

##### 1、安装yum-utils

```
sudo yum install yum-utils
```

##### 2、配置yum仓库

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

##### 3、安装nginx

```
sudo yum install nginx
```

安装过程中提示导入GPG key的时候，检查指纹（fingerprint）是否和`573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62`一致，如果检查一致则接受并继续安装即可。

##### 4、生成外部访问账号密码

> 这里使用到htpasswd工具，而httpd包中包含了该工具

安装httpd

```
sudo yum install httpd
```

生成账号

```
sudo htpasswd -c /etc/nginx/passwd.db [账号名称]
# 按照提示输入两次密码即可
New password:
Re-type new password:
```

记住/etc/nginx/passwd.db文件位置，后面的配置中需要

##### 5、配置nginx

> 这里使用80端口作为nginx提供给外部的访问接口，9090作为GIT的http的访问端口，然后通过nginx的反向代理功能，将80端口对git目录的访问重定向到本地的9090端口

给出配置如下（注意配置更改完后需要执行 “nginx -s reload” 命令使得更改生效）

/etc/nginx/nginx.conf

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

/etc/nginx/conf.d/default.conf

```
server {
    listen       80;
    server_name  42.194.183.95;

    root /usr/share/nginx/html;

    location /git {
        proxy_pass http://localhost:9090/;
    }
}
```

/etc/nginx/conf.d/git.conf

```
server {

    listen       9090;
    server_name  localhost;

    root /home/git/gitrepo;

    location ~ /.*\.git/(HEAD|info/refs|objects/info/.*|git-(upload|receive)-pack)$ {
        client_max_body_size 100m;
        auth_basic "Git User Authentication";
        auth_basic_user_file /etc/nginx/passwd.db;
        fastcgi_pass  unix:/var/run/fcgiwrap.socket;
        fastcgi_connect_timeout 24h;
        fastcgi_read_timeout 24h;
        fastcgi_send_timeout 24h;
        fastcgi_param SCRIPT_FILENAME     /usr/libexec/git-core/git-http-backend;
        fastcgi_param PATH_INFO           $uri;
        fastcgi_param GIT_HTTP_EXPORT_ALL "";
        fastcgi_param GIT_PROJECT_ROOT    /home/git/gitrepo;
        fastcgi_param REMOTE_USER $remote_user;
        include fastcgi_params;

    }
}
```

注意/usr/libexec/git-core/git-http-backend很重要，是GIT提供http服务的关键，通常安装了git-core模块就能找到该文件。

#### 安装fcgiwraper和spawn-fcgi

```
sudo yum install fcgiwrap spawn-fcgi
```

##### 1、配置spawn-fcgi

> spawn-fcgi 的作用是用来管理fcgiwrap进程，主要跟nginx有交互的还是fcgiwrap模块

```
sudo vim /etc/sysconfig/spawn-fcgi

# 编辑内容如下

FCGI_SOCKET=/var/run/fcgiwrap.socket
FCGI_PROGRAM=/usr/sbin/fcgiwrap
FCGI_USER=nginx
FCGI_GROUP=nginx
FCGI_EXTRA_OPTIONS="-M 0700"
OPTIONS="-u $FCGI_USER -g $FCGI_GROUP -s $FCGI_SOCKET -S $FCGI_EXTRA_OPTIONS -F 1 -P /var/run/spawn-fcgi.pid -- $FCGI_PROGRAM"
```

> 其中/usr/sbin/fcgiwrap是fcgiwrap的可执行文件路径，nginx是运行nginx的用户以及用户组（通常在root下启动的nginx，其主进程的启动用户是root，工作进程的启动用户是nginx）

##### 2、启动spawn-fcgi

```
sudo systemctl start spawn-fcgi.service
sudo systemctl enable spawn-fcgi.service
```

> systemctl配置开机启动出错，如下提示
>
> spawn-fcgi.service is not a native service, redirecting to /sbin/chkconfig.
> Executing /sbin/chkconfig spawn-fcgi on

则按照提示如下执行即可

```
sudo /sbin/chkconfig spawn-fcgi on
```

#### 修改GIT仓库目录的访问权限

- 1、将nginx用户添加到git用户组

  ```
  sudo gpasswd -d nginx git
  ```

- 2、修改/home/git/gitrepo目录权限

  ```
  # 允许git组用户可读可写可执行
  sudo chmod g+rwx -R /home/git/gitrepo
  ```

#### 测试http配置

```
PS D:\文档资料\test> git clone http://42.194.183.95/git/test.git
# 此处需要输入上面使用htpasswd创建的账号密码
Cloning into 'test'...
remote: Counting objects: 9, done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 9 (delta 2), reused 0 (delta 0)
Unpacking objects: 100% (9/9), 112.13 KiB | 3.20 MiB/s, done.
```

如果输入结果如上，则表示GIT服务的http协议已经配置成功。

#### 记住密码

设置记住密码（默认15分钟）：

git config --global credential.helper cache
如果想自己设置时间，可以这样做：

git config credential.helper 'cache --timeout=3600'
这样就设置一个小时之后失效

长期存储密码：

git config --global credential.helper store
增加远程地址的时候带上密码也是可以的。(推荐)

http://yourname:password@git.oschina.net/name/project.git
补充：使用客户端也可以存储密码的。

如果你正在使用ssh而且想体验https带来的高速，那么你可以这样做： 切换到项目目录下 ：

cd projectfile/
移除远程ssh方式的仓库地址

git remote rm origin
增加https远程仓库地址

git remote add origin http://yourname:password@git.oschina.net/name/project.git

#### 清除密码

清除缓存的用户名和密码：git credential-manager uninstall