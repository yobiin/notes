# 使用Nginx作为HTTP负载均衡器



### 1 介绍

负债均衡是跨多应用实例使用到的一种技术，主要用来优化资源利用、最大化访问吞吐量、减少请求延迟以及确保容错配置。

使用`nginx`作为一个高效的HTTP负载均衡器来分发流量到多个应用服务器，从而提高web应用的性能、可扩展性以及可靠性。

### 2 负载均衡方法

`Nginx`支持以下的负载均衡机制（或者方法）：

* round-robin —— `requests`请求通过循环方式分发到应用服务器。
* least-connected —— 下一个`request`请求分配到连接数最小的应用服务器。
* ip-hash —— 通过哈希函数决定下一个`request`请求应该分发到哪个应用服务器（基于客户端IP地址）。

### 3 默认负载均衡配置

最简单的负载均衡器配置如下所示：

~~~shell
http {
	upstream myapp1 {
		server srv1.example.com;
		server srv2.example.com;
		server srv3.example.com;
	}
	server {
		listen 80;
		
		location / {
			proxy_pass http://myapp1;
		}
	}
}
~~~

在上面的例子中，有三个相同的应用的实例分别运行在srv1-srv3服务器上。当负载均衡方法没有明确指定时默认使用`round-robin`方式分发请求。所有的请求被代理到服务器分组`myapp1`，然后`nginx`使用HTTP负载均衡方法分发请求到目标服务器。

反向代理的实现在`nginx`中包括：HTTP、HTTPS、FastCGI、uwsgi、SCGI、memcached和gRPC的负载均衡。

配置负载均衡时，如果想用HTTPS替换HTTP，那么只需要使用`https`替换`http`协议即可。

为`FastCGI`、`uwsgi`、`SCGI`、`memcached`或者`gRPC`设置负载均衡时，分别使用[fastcgi_pass](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass)、[uwsgi_pass](http://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_pass)、[scgi_pass](http://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_pass)、[memcached_pass](http://nginx.org/en/docs/http/ngx_http_memcached_module.html#memcached_pass)和[grpc_pass](http://nginx.org/en/docs/http/ngx_http_grpc_module.html#grpc_pass)指令即可。

### 4 最小连接负载均衡

另一个负载均衡机制是最小连接数原则。当一些请求需要花费相当长时间才能完成时，最小连接数能更加公平的控制应用实例上的负载。



