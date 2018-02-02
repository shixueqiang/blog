---
title: TeamTalk——ubuntu服务端部署
id: 64
categories:
  - 未分类
date: 2016-09-13 14:52:30
tags:
---

<span style="font-size: large;">这是第二次部署了，第一次在双系统Ubuntu上部署，这次准备在虚拟机上部署，所用版本Ubuntu 15.10 64位(14.04版本编译出很多问题，所以直接用最新版),这里要感谢蓝狐的教程</span>[<span style="font-size: large;">http://www.bluefoxah.org/teamtalk/new_tt_deploy.html</span>](http://www.bluefoxah.org/teamtalk/new_tt_deploy.html)

<span style="font-size: large;">一、准本工作</span>

<span style="font-size: large;">1、更新操作系统(非必须)
</span>

sudo apt-get update

<span style="font-size: large;">2、删除已安装的软件</span>

mysql、nginx、php最好都用新版。
2.1安装mysql

mysql版本5.6以上

sudo apt-get autoremove --purge mysql-server-5.5

sudo apt-get autoremove --purge mysql-client-5.5

官网下载mysql5.6

![](http://write.blog.csdn.net/postedit/50157731)

安装：

sudo dpkg -i mysql-common_5.6.27-1ubuntu14.04_i386.deb mysql-community-client_5.6.27-1ubuntu14.04_i386.deb mysql-community-        server_5.6.27-1ubuntu14.04_i386.deb

提示缺少 libaio1，安装libaio1

sudo apt-get install libaio1;

重新安装成功

或者 sudo apt-get install mysql-server mysql-client libmysqlclient-dev

导入sql脚本，进入TeamTalk-master/auto_setup/mariadb/conf/

source /自己的目录/TeamTalk-master/auto_setup/mariadb/conf/ttopen.sql;

2.2 安装php

sudo apt-get install php5-fpm; sudo apt-get install php5-mysql;

2.3 安装nginx

如果先前安装了Apache2 先卸载，卸载方法同上

官网下载最新为1.9.7，解压

./configure --prefix=/usr/local/nginx    安装在/usr/local下

提示缺少依赖包pcre

sudo apt-get install libpcre3 libpcre3.dev

重新configure提示缺少zlib

sudo apt-get install zlib1g zlib1g.dev

安装依赖 ssl

sudo apt-get install libssl-dev

重新configure，通过，编译安装nginx

sudo make

sudo make install

启动   sudo  /usr/local/nginx/sbin/nginx

2.4安装redis

官网下载    3.0.5

sudo make

sudo make install

启动  ./redis-server    为了省事我这里不写启动脚本了

2.5 安装PB

protobuf-2.6.1(teamtalk server源码目录下有)，安装

./configure --prefix=/usr/local/protobuf

sudo make

sudo make install

<span style="font-size: large;">3编译源码</span>

3.1准备工作
进入  server/src目录

sudo sh make_log4cxx.sh

报错，手动安装log4cxx依赖apr，apr-util

官网下载http://apr.apache.org/download.cgi

安装apr

./configure --prefix=/usr/apr

sudo make

sudo make install

安转apr-util
<div class="line number1 index0 alt2">`   .``/configure``--prefix=``/usr/apr-util``--with-apr=``/usr/apr`</div>
sudo make

sudo make install

重新 sudo sh make_log4cxx.sh

还报错打开  make_log4cxx.sh

在build_log4cxx（）里修改apr与apr-util路径为刚才的安装路径

重新 sudo sh make_log4cxx.sh，通过。

sudo sh make_hredis.sh

3.2 编译server

sudo sh build_ubuntu.sh version 1.0.0

缺少uuid的话安装

sudo apt-get install uuid-dev

编译成功后会在server目录下生成im-server-1.0.0压缩包

<span style="font-size: large;">  4、部署</span>

解压im-server-1.0.0 为了方便看日志将lib包下的log4cxx.properties复制到各个server目录下

设置临时环境变量 export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:自己的路径/im-server-1.0.0/lib

各个server的配置文件按蓝狐的教程就行

4.1db_proxy_server

server启动顺序会有一定影响所以先启动db_proxy_server

进入db_proxy_server

../daeml db_proxy_server

查看是否启动

ps -ef|grep server

发现没有启动成功，去看log日志提示无法链接到192.168.1.94的数据库上

检查mysql的配置文件my.cnf发现bind-address监听的是127.0.0.1修改成本机ip，重启mysql

我用的root用户，所以还得为root开启远程访问权限

登录mysql，执行

grant all on mysql.* to 'root'@'%' identified by '你的root密码';

flush privileges;//提交修改

退出mysql使用本机ip访问mysql

mysql -h 192.168.0.94 -u root -p

如果能访问说明设置成功

重新启动db_proxy_server成功

4.2route_server

../daeml route_server

4.3file_server

../daeml file_server

4.4msfs

进入msfs目录

拷贝msfs.conf.example一份，重命名为msfs.conf

打开msfs.conf修改BaseDir=自己的目录/files，主要存放小文件照片，表情等

../daeml msfs

4.5login_server

../daeml login_server

4.6msg_server

../daeml msg_server

4.7http_msg_server

../daeml http_msg_server

4.8push_server

../daeml push_server

<span style="font-size: large;"> 5.配置php服务器</span>

<span style="font-size: large;">   </span><span style="font-size: large;">下面是我的nginx服务器的配置，nginx.conf</span>

&nbsp;
<pre class="html">user  www-data;
worker_processes  auto;

error_log  /自己的目录/logs/error.log crit;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

pid        /usr/local/nginx/logs/nginx.pid;

worker_rlimit_nofile 51200;
events {
    use epoll;
    worker_connections  51200;
    multi_accept on;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    server_names_hash_bucket_size 128;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_max_body_size 50m;
    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout 60;
    tcp_nodelay on;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 256k;
    gzip on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types       text/plain application/x-javascript text/css application/xml;
    gzip_vary on;
    gzip_proxied        expired no-cache no-store private auth;
    gzip_disable        "MSIE [1-6]\.";
    server_tokens off;
    log_format  access  '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" $http_x_forwarded_for';

    server {
        listen       80;
        server_name  192.168.214.130;
	root   /自己的目录/www/teamtalk;
        location ~ \.php($|/){
            root   /自己的目录/www/teamtalk;
            index  index.html index.htm index.php default.html default.htm default.php;
            fastcgi_pass   unix:/var/run/php5-fpm.sock;//这里写自己php-fpm.conf  里listen的位置
                fastcgi_index  index.php;
                fastcgi_split_path_info ^(.+\.php)(.*)$;
                fastcgi_param   PATH_INFO $fastcgi_path_info;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                include        fastcgi_params;
        }

         location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
                    {
                            expires      30d;
                    }

            location ~ .*\.(js|css)?$
                    {
                            expires      12h;
                    }
            if (!-e $request_filename) {
                rewrite ^/(.*)$ /index.php/$1 last;
                break;
          }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
</pre>
&nbsp;

修改完后记得重启nginx。

编译部署完成，如果所有server都启动了还是不能发消息的话，重新启动所有server再试一遍。

编译好的文件[http://download.csdn.net/detail/s569646547/9326617](http://download.csdn.net/detail/s569646547/9326617)

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;