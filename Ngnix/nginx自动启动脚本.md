创建自动输入密码启动脚本
vi /usr/sbin/nginxdstart 

#!/usr/bin/expect -f
  set timeout 10
  log_user 0
  spawn /usr/sbin/nginx -c /etc/nginx/nginx.conf
  expect {
  "*phrase" { send "root.501\r"; exp_continue}
}

创建自动输入密码停止脚本
vi /usr/sbin/nginxdstop


#!/usr/bin/expect -f
  set timeout 10
  log_user 0
  spawn /usr/sbin/nginx -s stop
  expect {
  "*phrase" { send "root.501\r"; exp_continue}
}



修改启动服务
vi /usr/lib/systemd/system/nginx.service
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
#ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecStart=/usr/sbin/nginxdstart
ExecReload=/bin/sh -c "/bin/kill -s HUP $(/bin/cat /var/run/nginx.pid)"
#ExecStop=/bin/sh -c "/bin/kill -s TERM $(/bin/cat /var/run/nginx.pid)"
ExecStop=/usr/sbin/nginxdstop

[Install]
WantedBy=multi-user.target