---
layout: post
title: Automatic deployment Tengine
categories: [linux,tengine]
tags: [linux,tengine]
sidebar: []
---

执行脚本之前创建以下配置信息

结构如下：
    --- tengine
        --- config
            nginx.conf
            server.forward
            tengine
            tengine.service
        --- tengine-auto.sh

### tengine/config/nginx.conf
```sh
user apache apache;

worker_processes auto;
# worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;

#error_log /data/logs/nginx/error.log crit;
pid /var/run/nginx.pid;

#google_perftools_profiles /var/tmp/tcmalloc;

worker_rlimit_nofile 30720;

events {
	use epoll;
	multi_accept on;
	worker_connections 10240;
}

http {
	server_tokens off;
	autoindex off;
	access_log off;
	include mime.types;
	default_type application/octet-stream;
	

	#charset gb2312;
	server_names_hash_bucket_size 128;
	client_header_buffer_size 32k;
	large_client_header_buffers 4 32k;
	client_max_body_size 10m;
	client_body_buffer_size 256k;
   	fastcgi_buffer_size 128k;
   	fastcgi_buffers 32 256k;
	fastcgi_busy_buffers_size 256K;
  	fastcgi_temp_file_write_size 256k;
	
	#fastcgi_cache_path   /dev/shm/nginx_cache  levels=1:2 keys_zone=cfcache:10m inactive=50m;
	#fastcgi_cache_key "$request_method://$host$request_uri";
	#fastcgi_cache_methods GET HEAD;
	#fastcgi_cache   cfcache;
	#fastcgi_cache_valid   any 1d;
	#fastcgi_cache_min_uses  1;
	#fastcgi_cache_use_stale error  timeout invalid_header http_500;
	#fastcgi_ignore_client_abort on;
	
	sendfile on;
	tcp_nopush on;
	keepalive_timeout 15;
	tcp_nodelay on;
	client_header_timeout 10;
	client_body_timeout 10;
	reset_timedout_connection on;
	send_timeout 10;

	gzip on;
	gzip_disable "msie6";
	gzip_proxied any;
	gzip_min_length 1000;
	gzip_http_version 1.0;
	gzip_comp_level 4;
	gzip_types text/plain application/json application/x-javascript text/css application/xml;
	gzip_vary on;

	#limit_zone crawler $binary_remote_addr 10m;
	limit_req_zone $binary_remote_addr zone=one:3m rate=1r/s;
    	limit_req_zone $binary_remote_addr $uri zone=two:30m rate=50r/s;
    	limit_req_zone $binary_remote_addr $request_uri zone=three:3m rate=1r/s;


	proxy_connect_timeout 600;
	proxy_read_timeout 600;
	proxy_send_timeout 600;
	proxy_buffer_size 128k;
	proxy_buffers 64 128k;
	#proxy_buffers 4 128k;
	proxy_busy_buffers_size 256k;
	proxy_temp_file_write_size 256k;
	proxy_headers_hash_max_size 1024;
	proxy_headers_hash_bucket_size 128;

	proxy_redirect off;
	proxy_set_header Host $host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header REMOTE-HOST $remote_addr;
	proxy_set_header X-Forwarded-For $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

	proxy_temp_path /usr/local/tengine/nginx_temp;
	proxy_cache_path /usr/local/tengine/nginx_cache levels=1:2 keys_zone=cache_one:2048m inactive=30m max_size=60g;


	# backend apache server address pool
	#include SET/*.conf;

	log_format main '$remote_addr - $remote_user [$time_local] "$request"'
		'$status $body_bytes_sent "$http_referer"'
		'"$http_user_agent" $http_x_forwarded_for $request_body - $request_time s  - upstream = $upstream_response_time s';

	server {
		server_name _;     
		return 404;
	}

	# web page cache and proxy setting
	include ./sites-enable/*.*;
}
```

### tengine/config/server.forward
```sh
upstream forward_pool {
  server 127.0.0.1:8001;
}

server {
        listen          19988;
        server_name     127.0.0.1;

        error_log /data/logs/tengine/sever_forward_error.log;
        access_log /data/logs/tengine/serve_forward_access.log main;

        location / {
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://forward_pool;
        }
}
```

### tengine/config/tengine
```sh
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# this script create it by jackbillow at 2007.10.15.
# it is v.0.0.2 version.
# if you find any errors on this scripts,please contact jackbillow.
# and send mail to jackbillow at gmail dot com.
#
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
#              It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /var/run/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf

nginxd=/usr/local/tengine/sbin/nginx
nginx_config=/usr/local/tengine/conf/nginx.conf
nginx_pid=/usr/local/tengine/logs/nginx.pid

RETVAL=0
prog="nginx"

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
#[ ${NETWORKING} = "no" ] && exit 0

[ -x $nginxd ] || exit 0


# Start nginx daemons functions.
start() {

if [ -e $nginx_pid ];then
echo "nginx already running...."
exit 1
fi

echo -n $"Starting $prog: "
daemon $nginxd -c ${nginx_config}
RETVAL=$?
echo
[ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
return $RETVAL

}


# Stop nginx daemons functions.
stop() {
       echo -n $"Stopping $prog: "
       killproc $nginxd
       RETVAL=$?
       echo
       [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /var/run/nginx.pid
}


# reload nginx service functions.
reload() {

echo -n $"Reloading $prog: "
#kill -HUP `cat ${nginx_pid}`
killproc $nginxd -HUP
RETVAL=$?
echo

}

# See how we were called.
case "$1" in
start)
       start
       ;;

stop)
       stop
       ;;

reload)
       reload
       ;;

restart)
       stop
       start
       ;;

status)
       status $prog
       RETVAL=$?
       ;;
*)
       echo $"Usage: $prog {start|stop|restart|reload|status|help}"
       exit 1
esac

exit $RETVAL
```

### tengine/config/tengine.service
```sh
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/local/tengine/sbin/nginx -t
ExecStart=/usr/local/tengine/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

```sh
#!/bin/bash
src_path=$(pwd)
cd $src_path
groupadd apache
useradd -g apache -m apache

# install system modules
yum -y install gd-devel openssl-devel readline-deve lncurses-devel readline readline-devel libxml2 libxml2-devel libxslt libxslt-devel

# download software package
wget http://tengine.taobao.org/download/tengine-2.2.2.tar.gz
wget https://openssl.org/source/openssl-1.0.2.tar.gz

t_ver="tengine-2.2.2"
o_ver="openssl-1.0.2"
tar zxvf ${t_ver}.tar.gz
tar zxvf ${o_ver}.tar.gz

# install tengine 2.2.x
cd $t_ver
./configure --prefix=/usr/local/$t_ver \
    --user=apache --group=apache \
    --with-pcre-jit \
    --with-mail \
    --with-openssl=../openssl-1.0.2 \
    --with-mail_ssl_module \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-openssl-opt="enable-tlsext" \
    --with-http_auth_request_module \
    --with-http_dav_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_image_filter_module \
    --with-http_stub_status_module \
    --with-http_realip_module \
    --with-http_addition_module \
    --with-http_image_filter_module \
    --with-http_sub_module \
    --with-http_flv_module \
    --with-http_slice_module \
    --with-http_mp4_module \
    --with-http_concat_module \
    --with-http_random_index_module \
    --with-http_secure_link_module \
    --with-http_sysguard_module
make && make install

ln -s /usr/local/$t_ver/ /usr/local/tengine

# make dir
cd $src_path
mkdir -p /data/logs/tengine
mkdir -p /data/www/localhost
mkdir -p /usr/local/$t_ver/conf/sites-enable
rm -f /usr/local/$t_ver/conf/nginx.conf
cp -a config/nginx.conf /usr/local/$t_ver/conf/nginx.conf
cp -a config/service.forward /usr/local/$t_ver/conf/sites-enable/
chown -R apache.apache /data/www/localhost

# copy startup script
cd $src_path
cp config/tengine2 /etc/init.d/
chmod +x /etc/init.d/tengine2
chkconfig --add tengine2
chkconfig tengine2 on

# for systemctl (centos 7)

## cp config/tengine2.service /etc/systemd/system/multi-user.target.wants/

# start and test

/etc/init.d/tengine2 start
/usr/local/$t_ver/sbin/nginx -V

# for systemctl (centos 7)

## systemctl status tengine2

# del

rm -rf tengine-2.2.2 openssl-1.0.2

```
