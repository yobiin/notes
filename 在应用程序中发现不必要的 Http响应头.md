## 13、在应用程序中发现不必要的 **Http**响应头

原因：应用返回了Nginx服务版本信息

~~~html
GET /console/login/cas?ticket=ST-1350-PL2Nr3xEVsMkJ4EhtCeDjGaxZm8-xzops_system HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.106 Safari/537.36
Referer: https://192.168.1.24:28443/cas/login
Host: 192.168.1.24:28443
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US
HTTP/1.1 401 
Connection: keep-alive
X-XSS-Protection: 1; mode=block
Server: nginx/1.20.2
Pragma: no-cache
Content-Length: 289
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Content-Language: en-US
Set-Cookie: JSESSIONID=E58D862516D8ACB4B123542474C07CF6; Path=/console; Secure; HttpOnly; SameSite=None
Date: Thu, 23 Dec 2021 01:31:46 GMT
Content-Security-Policy: default-src 'self';script-src 'self' 'unsafe-inline' 'unsafe-eval';img-src 'self' data:;style-src
'self' 'unsafe-inline';font-src 'self' data:
Expires: 0
Content-Type: text/html;charset=UTF-8
~~~

如上面的：Server: nginx/1.20.2

1、下面配置只能隐藏Nginx版本号，如下在http块中添加：

~~~config
server_tokens off;
~~~

2、想要完全移除响应中的Server选项，需要第三方插件配合

- [headers-more-nginx-module-0.33github地址](https://github.com/openresty/headers-more-nginx-module)
- [查看插件支持的nginx版本github地址](https://github.com/openresty/headers-more-nginx-module#compatibility)
- [headers-more-nginx-module下载地址](https://github.com/openresty/headers-more-nginx-module/tags)

下载

```bash
# 举例目录/app/tools
cd /app/tools/
#下载插件
wget https://github.com/openresty/headers-more-nginx-module/archive/v0.33.tar.gz
#解压
tar -zxvf v0.33.tar.gz
```

加载模块

```bash
# 查看安装参数命令(取出：configure arguments:)
/app/nginx/sbin/nginx -V
# 在nginx资源目录编译
cd /app/nginx-1.12.2/
# 将上面取出的configure arguments后面追加 --add-module=/app/tools/headers-more-nginx-module-0.33
./configure --prefix=/app/nginx112 --add-module=/app/tools/headers-more-nginx-module-0.33
# 编辑，切记没有make install
make
# 备份
cp /app/nginx112/sbin/nginx /app/nginx112/sbin/nginx.bak 
# 覆盖(覆盖提示输入y)
cp -f /app/nginx-1.12.2/objs/nginx /app/nginx112/sbin/nginx
```

修改配置

```bash
vim /app/nginx112/conf/nginx.conf
# 添加配置(在http模块)
more_clear_headers 'Server';
```

> 上面配置只是将http响应头中的Server:nginx/1.12.2清楚，详细使用方案可阅读 [参考文档](https://github.com/openresty/headers-more-nginx-module),
> 支持添加·修改·清除响应头的操作，

重启nginx

```bash
/app/nginx112/sbin/nginx -s stop
/app/nginx112/sbin/nginx
```

> 直接使用reload可能会无效