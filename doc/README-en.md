ngx_php
======
[![Build Status](https://travis-ci.org/rryqszq4/ngx_php.svg?branch=master)](https://travis-ci.org/rryqszq4/ngx_php) 
[![Gitter](https://badges.gitter.im/rryqszq4/ngx_php.svg)](https://gitter.im/rryqszq4/ngx_php?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
[![GitHub release](https://img.shields.io/github/release/rryqszq4/ngx_php.svg)](https://github.com/rryqszq4/ngx_php/releases/latest)
[![license](https://img.shields.io/badge/license-BSD--2--Clause-blue.svg)](https://github.com/rryqszq4/ngx_php/blob/master/LICENSE)

[ngx_php](https://github.com/rryqszq4/ngx_php) - Embedded php script language for nginx-module. Another name is nginx-php5-module.   
[English document](https://github.com/rryqszq4/ngx_php/blob/master/doc/README-en.md) | [中文文档](https://github.com/rryqszq4/ngx_php/blob/master/doc/README-zh.md)  
QQ Group：558795330

Features
--------
* Load php.ini config file
* Global variable support $_GET, $_POST, $_COOKIE, $_SERVER, $_FILES, $_SESSION...
* PHP script code and file execute
* RFC 1867 protocol file upload
* PHP error reporting output
* Support PECL PHP extension
* Support Nginx API for php

Requirement
-----------
- PHP 5.3.* ~ PHP 5.6.*
- nginx-1.7.12 ~ nginx-1.11.8

Installation
-------
**build php**

```sh
$ wget 'http://php.net/distributions/php-5.3.29.tar.gz'
$ tar xf php-5.3.29.tar.gz
$ cd php-5.3.29

$ ./configure --prefix=/path/to/php \
$             --enable-maintainer-zts \
$             --enable-embed
$ make && make install
```

**build ngx_php**

```sh
$ git clone https://github.com/rryqszq4/ngx_php.git

$ wget 'http://nginx.org/download/nginx-1.7.12.tar.gz'
$ tar xf nginx-1.7.12.tar.gz
$ cd nginx-1.7.12

$ export PHP_BIN=/path/to/php/bin
$ export PHP_INC=/path/to/php/include/php
$ export PHP_LIB=/path/to/php/lib

$ ./configure --user=www --group=www \
$             --prefix=/path/to/nginx \
$             --with-ld-opt="-Wl,-rpath,$PHP_LIB" \
$             --add-module=/path/to/ngx_php/dev/ngx_devel_kit \
$             --add-module=/path/to/ngx_php
$ make && make install
```

Synopsis
--------

```nginx
user www www;
worker_processes  4;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    keepalive_timeout  65;
    
    client_max_body_size 10m;   
    client_body_buffer_size 4096k;

    php_ini_path /usr/local/php/etc/php.ini;

    server {
        listen       80;
        server_name  localhost;
    
        location /php {
            content_by_php '
                echo "hello ngx_php";
            ';
        }
    }
}
```

Framework conf
--------------

**yaf & yii :**

```nginx
server {
    listen 80;
    server_name yaf-sample.com;
    access_log  logs/yaf-sample.com.access.log;

    root /home/www/yaf-sample;
    index index.php index.html;
    
    location /favicon.ico {
        log_not_found off;
    }

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        content_by_php '
            header("Content-Type: text/html;charset=UTF-8");
            require_once("/home/www/yaf-sample/index.php");
        ';
    }
}  
```

**wordpress :**

```nginx
server {
    listen 80;
    server_name wordpress-sample.com;
    
    location ~ \.php$ {
        root   /home/www/wordpress;
        content_by_php "
            require_once('/home/www/wordpress'.$_SERVER['DOCUMENT_URI']);
        ";
    }
}
```

Test
----
Using the perl of [Test::Nginx](https://github.com/openresty/test-nginx) module to testing, searching and finding out problem in ngx_php. 

```sh
cd /path/to/ngx_php
export PATH=/path/to/nginx/sbin:$PATH
prove -r t
```
Test result:

```sh
t/001-hello.t ........... ok
t/002-ini.t ............. ok
t/003-_GET.t ............ ok
t/004-_POST.t ........... ok
t/005-_SERVER.t ......... ok
t/006-_COOKIE.t ......... ok
t/007-_FILES.t .......... ok
t/008-error.t ........... ok
t/009-session.t ......... ok
t/100-ngx_socket_tcp.t .. ok
t/200-rewrite_by_php.t .. ok
t/202-access_by_php.t ... ok
All tests successful.
Files=12, Tests=40,  4 wallclock secs ( 0.06 usr  0.01 sys +  1.23 cusr  0.46 csys =  1.76 CPU)
Result: PASS
```

Directives
----------
* [php_ini_path](#php_ini_path)
* [init_by_php](#init_by_php)
* [init_by_php_file](#init_by_php_file)
* [rewrite_by_php](#rewrite_by_php)
* [rewrite_by_php_file](#rewrite_by_php_file)
* [access_by_php](#access_by_php)
* [access_by_php_file](#access_by_php_file)
* [content_by_php](#content_by_php)
* [content_by_php_file](#content_by_php_file)
* [log_by_php](#log_by_php)
* [log_by_php_file](#log_by_php_file)
* [thread_by_php](#thread_by_php)
* [thread_by_php_file](#thread_by_php_file)
* [set_by_php](#set_by_php)
* [set_run_by_php](#set_run_by_php)
* [set_by_php_file](#set_by_php_file)
* [set_run_by_php_file](#set_run_by_php_file)

php_ini_path
------------
**syntax:** *php_ini_path &lt;php.ini file path&gt;*  

**context:** *http*  

**phase:** *loading-config*  

Loading php configuration file in nginx configuration initialization.

```nginx
php_ini_path /usr/local/php/etc/php.ini;
```

init_by_php
-----------
**syntax:** *init_by_php &lt;php script code&gt;*  

**context:** *http*  

**phase:** *loading-config*

In nginx configuration initialization or boot time, run some php scripts.

init_by_php_file
----------------
**syntax:** *init_by_php_file &lt;php script file&gt;*  

**context:** *http*  

**phase:** *loading-config*

In nginx configuration initialization or boot time, run some php script file.

rewrite_by_php
--------------
**syntax:** *rewrite_by_php &lt;php script code&gt;*  

**context:** *http, server, location, location if*  

**phase:** *rewrite*

Use php script redirect in nginx rewrite stage of.

```nginx
location /rewrite_by_php {
        rewrite_by_php "
            echo "rewrite_by_php";
            header('Location: http://www.baidu.com/');
        ";
    }
```

rewrite_by_php_file
-------------------
**syntax:** *rewrite_by_php_file &lt;php script file&gt;*  

**context:** *http, server, location, location if*  

**phase:** *rewrite*

Use php script file, redirect in nginx rewrite stage of.

access_by_php
-------------
**syntax:** *access_by_php &lt;php script code&gt;*  

**context:** *http, server, location, location if*  

**phase:** *access*

Nginx in the access phase, the php script determine access.

access_by_php_file
------------------
**syntax:** *access_by_php_file &lt;php script file&gt;*  

**context:** *http, server, location, location if*  

**phase:** *access*

Nginx in the access phase, the php script file Analyzing access.

content_by_php
--------------
**syntax:** *content_by_php &lt;php script code&gt;*  

**context:** *http, server, location, location if*  

**phase:** *content*

Most central command, run php script nginx stage of content.
```nginx
location /content_by_php {    
    content_by_php "
        header('Content-Type: text/html;charset=UTF-8');
    
        echo phpinfo();
    ";
        
}
```

content_by_php_file
-------------------
**syntax:** *content_by_php_file &lt;php script file&gt;*  

**context:** *http, server, location, location if*  

**phase:** *content*

Most central command, run php script file nginx stage of content.
```nginx
location /content_by_php_file {
        content_by_php_file /home/www/index.php;
}
```

log_by_php
----------
**syntax:** *log_by_php &lt;php script code&gt;*  

**context:** *http, server, location, location if*  

**phase:** *log*

log_by_php_file
---------------
**syntax:** *log_by_php_file &lt;php script file&gt;*  

**context:** *http, server, location, location if*  

**phase:** *log*

thread_by_php
---------------------
**syntax:** *thread_by_php &lt;php script code&gt;*  

**context:** *http, server, location, location if*  

**phase:** *content*  

```nginx
resolver 8.8.8.8;

location = /thread_by_php {
    thread_by_php "
        header('Content-Type: application/x-javascript; charset=GBK');
        $tcpsock = new ngx_socket_tcp();
        $tcpsock->settimeout(30000);
        $tcpsock->connect('hq.sinajs.cn',80);
        $tcpsock->send('GET /list=s_sh000001 HTTP/1.0\r\nHost: hq.sinajs.cn\r\nConnection: close\r\n\r\n');
        $res = $tcpsock->receive();
        $tcpsock->close();
        $res = explode('\r\n',$res);
        var_dump($res[0]);
    ";
}
```

thread_by_php_file
--------------------------
**syntax:** *thread_by_php_file &lt;php script file&gt;*  

**context:** *http, server, location, location if*  

**phase:** *content* 

set_by_php
----------
**syntax:** *set_by_php &lt;php script code&gt;*  

**context:** *server, server if, location, location if*  

**phase:** *content*

set_run_by_php
--------------
**syntax:** *set_run_by_php &lt;php script code&gt;*  

**context:** *server, server if, location, location if*  

**phase:** *content*

set_by_php_file
---------------
**syntax:** *set_by_php_file &lt;php script file&gt;*  

**context:** *server, server if, location, location if*  

**phase:** *content*

set_run_by_php_file
-------------------
**syntax:** *set_run_by_php_file &lt;php script file&gt;*  

**context:** *server, server if, location, location if*  

**phase:** *content*


Nginx API for php
-----------------
* [ngx::_exit](#ngx_exit)
* [ngx_socket_tcp::__construct](#ngx_socket_tcp__construct)
* [ngx_socket_tcp::connect](#ngx_socket_tcpconnect)
* [ngx_socket_tcp::send](#ngx_socket_tcpsend)
* [ngx_socket_tcp::receive](#ngx_socket_tcpreceive)
* [ngx_socket_tcp::close](#ngx_socket_tcpclose)
* [ngx_socket_tcp::settimeout](#ngx_socket_tcpsettimeout)
* [ngx_log::error](#ngx_logerror)
* [ngx_time::sleep](#ngx_timesleep)

ngx::_exit
----------
**syntax:** *ngx::_exit(int $status)*  

**context:** *content_by_php* *thread_by_php*  

```php
echo "start\n";
ngx::_exit(200);
echo "end\n";
```

ngx_socket_tcp::__construct
---------------------------
**syntax:** *ngx_socket_tcp::__construct()*  

**context:** *thread_by_php*  

```php
$tcpsock = new ngx_socket_tcp();
```

ngx_socket_tcp::connect
-----------------------
**syntax:** *ngx_socket_tcp::connect(string $host, int $port)*  

**context:** *thread_by_php*  

resolver 8.8.8.8;

```php
$tcpsock = new ngx_socket_tcp();
$tcpsock->connect('127.0.0.1',11211));
```

ngx_socket_tcp::send
--------------------
**syntax:** *ngx_socket_tcp::send(string $buf)* 

**context:** *thread_by_php*  

```php
$tcpsock = new ngx_socket_tcp();
$tcpsock->connect('127.0.0.1',11211));
$tcpsock->send('stats\r\n');
```

ngx_socket_tcp::receive
-----------------------
**syntax:** *ngx_socket_tcp::receive()* 

**context:** *thread_by_php* 

```php
$tcpsock = new ngx_socket_tcp();
$tcpsock->connect('127.0.0.1',11211));
$tcpsock->send('stats\r\n');
$result = $tcpsock->receive();
var_dump($result);
$tcpsock->close();
```

ngx_socket_tcp::close
---------------------
**syntax:** *ngx_socket_tcp::close()*  

**context:** *thread_by_php*  

ngx_socket_tcp::settimeout
--------------------------
**syntax:** *ngx_socket_tcp::settimeout(int time)*  

**context:** *thread_by_php*  

ngx_log::error
--------------
**syntax:** *ngx_log::error(int level, string log)* 

**context:** *thread_by_php* 

Nginx log of level in php.
* NGX_LOG_STDERR
* NGX_LOG_EMERG
* NGX_LOG_ALERT
* NGX_LOG_CRIT
* NGX_LOG_ERR
* NGX_LOG_WARN
* NGX_LOG_NOTICE
* NGX_LOG_INFO
* NGX_LOG_DEBUG

```php
ngx_log::error(ngx_log::ERR, "test");

/*
 2016/10/06 22:10:19 [error] 51402#0: *1 test while reading response header from upstream, client: 192.168.80.1, 
 server: localhost, request: "GET /_mysql HTTP/1.1", upstream: "127.0.0.1:3306", host: "192.168.80.140"
*/
```

ngx_time::sleep
---------------
**syntax:** *ngx_time::sleep(int seconds)* 

**context:** *thread_by_php*  

```php
echo "sleep_start\n";

ngx_time::sleep(3);

echo "sleep_end\n"; 
```

Question
--------
[issues #6](https://github.com/rryqszq4/ngx_php/issues/6) - Using in php-5.3.29, libxml2 2.7.6 not thread safety. Please disable xml in php install.
```sh
./configure --prefix=/usr/local/php5329 \
            --with-config-file-path=/usr/local/php5329/etc \
            --with-iconv=/usr/local/libiconv \
            --disable-xml \
            --disable-libxml \
            --disable-dom \
            --disable-simplexml \
            --disable-xmlreader \
            --disable-xmlwriter \
            --without-pear \
            --enable-maintainer-zts  \
            --enable-embed
```

Copyright and License
---------------------
Copyright (c) 2016-2017, rryqszq4 <rryqszq@gmail.com>  
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
