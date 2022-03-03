## Redis快速开始指南

这是一份快速开始指南，目标是那些没有使用Redis经验的人群，阅读本指南将帮助你：

* 下载、编译安装Redis服务。
* 使用`redis-cli`访问Redis。
* 在你的应用中使用Redis。
* 了解Redis持久化是如何工作的。
* 如何更加合适的安装Redis。
* 找出下一步应该阅读什么内容，以便更进一步了解Redis。

## 安装Redis

比较建议的方式是通过编译源码来安装，而且Redis源码编译除了需要GCC编译器和libc外没有其它多余的依赖，编译前只需要安装好`GCC`编译器和`libc`库即可。

你可以自行访问 [redis.io](https://redis.io/)然后下载最新的Redis源码包，也可以通过 http://download.redis.io/redis-stable.tar.gz下载最新的稳定版Redis。

为了正确编译Redis，请遵循以下的步骤：

~~~shell
# yum install gcc gcc-c++
wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
cd redis-stable
make
~~~

此时此刻，你可以通过`make test`命令来检查上述安装是否正确，当然这是一个可选步骤，你可以选择忽略它。编译完成后，在src目录下你可以看到许多新生成的可执行文件，它们都是Redis服务的一部分：

* `redis-server`是Redis服务器本身。
* `redis-sentinel`是Redis哨兵可执行程序（用于监听及故障转移）。
* `redis-cli`是用于访问Redis的命令行工具。
* `redis-benchmark`是用来测试Redis性能的。
* `redis-check-aof`和·`redis-check-rdb`在数据文件损坏的场景下非常有用。

把`redis-server`和其它可执行文件拷贝到合适的地方是非常有用的，执行以下的命令：

~~~shell
sudo cp src/redis-server /usr/local/bin/
sudo cp src/redis-cli /usr/local/bin/
~~~

下面的文档中，我们假设`/usr/local/bin`目录在你系统的`PATH`环境变量中，这样你就不需要通过全路径的方式来执行二进制文件了。

## 启动Redis

Redis最简单的启动方式只需要执行`redis-server`命令即可，而不需要多余的参数。

~~~shell
[root@192 redis-stable]# redis-server
17605:C 03 Mar 2022 15:20:05.227 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
17605:C 03 Mar 2022 15:20:05.227 # Redis version=6.2.6, bits=64, commit=00000000, modified=0, pid=17605, just started
17605:C 03 Mar 2022 15:20:05.227 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
...更多日志...
~~~

上面的例子中，Redis在启动的时候没有显式的指定配置文件，因此所有的参数都将使用内部默认配置。如果你只是想体验一下或者在开发中使用它，那么这样是非常好的；但是在生产环境中使用时，你应该指定一个配置文件。那么应该如何在启动Redis的时候指定配置文件呢？你可以使用配置文件的全路径作为启动命令的第一个参数，举个例子`redis-server /etc/redis.conf`。Redis源码目录下提供了`redis.conf`文件，你可以使用它作为模板编写自己的配置文件。

## 检查Redis是否正常工作











