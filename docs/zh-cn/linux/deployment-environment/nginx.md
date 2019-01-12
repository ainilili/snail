此前因为需要部署静态资源服务器，所以就找了个vps部署一下Nginx，顺带将vsftpd配置好了，下面就给大家讲一下如何在CentOS上部署Nginx及vsftpd！

如果大家不知道Nginx和vsftpd的用处，请自行百度，这里就不过多介绍，废话少说，进入正题。

## 一、在CentOS上下载Nginx和vsftpd

常用的Nginx下载方式有两种，一种是使用CentOS上自带的yum下载，第二种是从网上下载后在通过make进行编译安装，下面我将会向大家介绍一下这两种安装方式

### 1、yum安装方式

首先我们更新一下yum软件库

```
yum update

```

然后我们搜索一下yum库关于nginx的rpm包

```
yum list | grep nginx

```

可以看到下面的列表

```
[root@VM_239_130_centos html]# yum list | grep nginx
nginx-filesystem.noarch                     1.10.2-1.el6                 @epel  
collectd-nginx.x86_64                       4.10.9-4.el6                 epel   
munin-nginx.noarch                          2.0.33-1.el6                 epel   
nginx.x86_64                                1.10.2-1.el6                 epel   
nginx-all-modules.noarch                    1.10.2-1.el6                 epel   
nginx-mod-http-geoip.x86_64                 1.10.2-1.el6                 epel   
nginx-mod-http-image-filter.x86_64          1.10.2-1.el6                 epel   
nginx-mod-http-perl.x86_64                  1.10.2-1.el6                 epel   
nginx-mod-http-xslt-filter.x86_64           1.10.2-1.el6                 epel   
nginx-mod-mail.x86_64                       1.10.2-1.el6                 epel   
nginx-mod-stream.x86_64                     1.10.2-1.el6                 epel   
pcp-pmda-nginx.x86_64                       3.10.9-9.el6                 os

```

接下来我们选择使用yum安装`nginx.x86_64 1.10.2-1.el6 epel`

```
yum install nginx

```

中间会提示我们一次是否确认安装

```
=======================================================================================================================
 Package                                   Arch                 Version                       Repository          Size
=======================================================================================================================
Installing:
 nginx                                     x86_64               1.10.2-1.el6                  epel               462 k
Installing for dependencies:
 nginx-all-modules                         noarch               1.10.2-1.el6                  epel               7.7 k
 nginx-mod-http-geoip                      x86_64               1.10.2-1.el6                  epel                14 k
 nginx-mod-http-image-filter               x86_64               1.10.2-1.el6                  epel                16 k
 nginx-mod-http-perl                       x86_64               1.10.2-1.el6                  epel                26 k
 nginx-mod-http-xslt-filter                x86_64               1.10.2-1.el6                  epel                16 k
 nginx-mod-mail                            x86_64               1.10.2-1.el6                  epel                43 k
 nginx-mod-stream                          x86_64               1.10.2-1.el6                  epel                36 k

Transaction Summary
=======================================================================================================================
Install       8 Package(s)

Total download size: 620 k
Installed size: 1.6 M
Is this ok [y/N]:

```

输入`y`继续

```
Installed:
  nginx.x86_64 0:1.10.2-1.el6                                                                                          

Dependency Installed:
  nginx-all-modules.noarch 0:1.10.2-1.el6                       nginx-mod-http-geoip.x86_64 0:1.10.2-1.el6            
  nginx-mod-http-image-filter.x86_64 0:1.10.2-1.el6             nginx-mod-http-perl.x86_64 0:1.10.2-1.el6             
  nginx-mod-http-xslt-filter.x86_64 0:1.10.2-1.el6              nginx-mod-mail.x86_64 0:1.10.2-1.el6                  
  nginx-mod-stream.x86_64 0:1.10.2-1.el6                       

Complete!

```

安装完毕！！接下来我们来看一下nginx的文件分布

```
whereis nginx

```

```
[root@VM_239_130_centos html]# whereis nginx
nginx: /usr/sbin/nginx /etc/nginx /usr/lib64/nginx /usr/local/nginx /usr/share/nginx /usr/share/man/man3/nginx.3pm.gz /usr/share/man/man8/nginx.8.gz

```

其中三个文件（夹）比较重要：

| 路径 | 作用 |
| --- | --- |
| /usr/sbin/nginx | nginx启动路径 |
| /etc/nginx | 存放nginx的配置文件 |
| /usr/share/nginx | 默认的nginx资源库 |

我们首先进入`/etc/nginx/`中看一下nginx到底有哪些配置文件

```
[root@VM_239_130_centos html]# cd /etc/nginx
[root@VM_239_130_centos nginx]# ls
conf.d        fastcgi.conf.default    koi-utf     mime.types.default  scgi_params          uwsgi_params.default
default.d     fastcgi_params          koi-win     nginx.conf          scgi_params.default  win-utf
fastcgi.conf  fastcgi_params.default  mime.types  nginx.conf.default  uwsgi_params

```

哇，看到这么多配置文件是不是吓了一跳，其实我们只需要在意nginx.conf就行了，其他的涉及到了再百度，接下来我们进入nginx.conf(这里我们使用vim，系统没有vim的小伙伴可以使用`yum install vim`进行下载安装)

```
vim nginx.conf

```

```
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '  #日志格式
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;                                                  #操作成功记录日志

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
}

```

上面的配置不要动，我们跟踪到这一行`include /etc/nginx/conf.d/*.conf`主要作用就是加载更多的配置文件，我们退出vim编辑`ESC+:q`进入到conf.d文件夹来看一下

```
[root@VM_239_130_centos nginx]# cd /etc/nginx/conf.d
[root@VM_239_130_centos conf.d]# ls
default.conf  default.conf.rpmsave  ssl.conf  virtual.conf

```

然后进入default.conf

```
vim default.conf

```

```
#
# The default server
#

server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  _;
    root         /usr/share/nginx/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }

}

```

我们发现这个文件配置的是server{}，server的配置就是我们nginx的核心配置，这里先不过多的讲，下面会详细介绍server的配置，但是此时如果我们运行nginx的话将会报错

```
 Address family not supported by protocol

```

我们需要将上面的

```
 listen       [::]:80 default_server;

```

注释掉

```
   # listen       [::]:80 default_server;

```

退出vim编辑，使用`/usr/sbin/nginx`启动nginx

```
[root@VM_239_130_centos conf.d]# /usr/sbin/nginx

```

关闭nginx

```
pkill -9 nginx

```

### 2、编译安装

这里借鉴了腾讯云论坛上的一个帖子： [CentOS 7中Nginx1.9.5编译安装教程systemctl启动](http://bbs.qcloud.com/thread-10429-1-1.html)

<font color="#555555">先安装gcc 等</font>

```
yum -y install gcc gcc-c++ wget

```

<font>.然后装一些库</font>

```
yum -y install gcc wget automake autoconf libtool libxml2-devel libxslt-devel perl-devel perl-ExtUtils-Embed pcre-devel openssl-devel

```

<font color="#555555">进入默认的软件目录</font>

```
cd /usr/local/src/

```

<font color="#555555">下载 nginx软件</font>

```
wget http://nginx.org/download/nginx-1.9.5.tar.gz

```

<span style="color: rgb(68, 68, 68); font-size: 14px;">如果这个下载太慢可以在这里下载</span><font>[http://nginx.org/download/nginx-1.9.5.tar.gz](http://nginx.org/download/nginx-1.9.5.tar.gz) 下载完后yum -y intall lrzsz 装好上传工具</font>

<font>然后用rz上传到服务器</font> <font color="#555555">然后解压文件.</font>

```
tar zxvf nginx-1.9.5.tar.gz

```

<font color="#555555">进入 nginx1.9.5的源码 如果想改版本号 可以进入源码目录</font><font color="#555555">src/core/nginx.h</font><font color="#555555">更改</font>

```
cd nginx-1.9.5/

```

<font color="#555555">创建一个nginx目录用来存放运行的临时文件夹</font>

```
mkdir -p /var/cache/nginx

```

<font color="#555555">开始configure</font>

```
./configure \
--prefix=/usr/local/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--user=nobody \
--group=nobody \
--with-pcre \
--with-http_v2_module \
--with-http_ssl_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_sub_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_stub_status_module \
--with-http_auth_request_module \
--with-mail \
--with-mail_ssl_module \
--with-file-aio \
--with-ipv6 \
--with-http_v2_module \
--with-threads \
--with-stream \
--with-stream_ssl_module

```

<span style="color: rgb(68, 68, 68); font-size: 14px;">接着 编译</span>

```
make

```

<span style="color: rgb(68, 68, 68); font-size: 14px;">安装</span>

```
make install

```

<font color="#555555">启动nginx</font>

```
/usr/sbin/nginx

```

<font color="#555555">用ps aux来查看nginx是否启动</font>

```
ps aux|grep nginx

```

<span style="font-size: 12px;">复制代码</span>

<span style="color: rgb(68, 68, 68); font-size: 14px;">然后配置服务</span>

```
vim /usr/lib/systemd/system/nginx.service

```

<font color="#555555">按i输入以下内容</font>

```
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target

```

<font color="#555555">编辑好后保存</font><font color="#555555">然后开启开机启动</font>

```
systemctl enable nginx.service

```

<font color="#555555">用命令关掉nginx</font>

```
pkill -9 nginx

```

<font color="#555555">后面可以用systemctl来操作nginx.service</font>

```
systemctl start nginx.service

```

这里值得一提的是nginx编译安装后的文件夹和yum安装的文件夹类似，nginx.conf文件都在/etc/nginx下，不过编译安装后的nginx.conf文件内部配置与yum安装略有差异

编译安装后的nginx.conf内部直接配置server，所以编译安装的小伙伴配置server就不用去改/etc/nginx/conf.d下的default.conf文件配置了，直接到nginx.conf文件中改server配置就行了

## 二、Nginx配置详解

1、全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。

2、events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。

3、http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。

4、server块：配置虚拟主机的相关参数，一个http中可以有多个server。

5、location块：配置请求的路由，以及各种页面的处理情况。

我们来到yum安装后的`/etc/nginx/conf.d/default.conf`文件中

```
server {
    listen       80 default_server;
   # listen       [::]:80 default_server;
    server_name  _;
    root         /usr/share/nginx/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }

```

在server块中

| 名称 | 作用 |
| --- | --- |
| listen | nginx监听端口 |
| root | nginx资源路径根目录 |
| location | url访问的本地资源路径配置(支持通配符) |
|  |
| error_page | 跳转报错页面 |

其中nginx会在资源目录中去找默认的index.html页面，我们进入/usr/share/nginx/html中看一下

```
[root@VM_239_130_centos conf.d]# cd /usr/share/nginx/html

[root@VM_239_130_centos html]# ls

404.html  50x.html  index.html  nginx-logo.png  poweredby.png

```

我们在这个文件夹中创建一个test.html

```
[root@VM_239_130_centos html]# touch test.html

[root@VM_239_130_centos html]# ls

404.html  50x.html  hello  index.html  nginx-logo.png  poweredby.png test.html  world

```

html页面内容

```
<html>
        <head>
                <title>test</title>
        </head>
        <body>  
                <h1>hello world</h1>
        </body>
</html>

```

然后我们在浏览器上去访问test.html

成功~

然后我们修改一下default.conf的配置，增加一个location

```
[root@VM_239_130_centos html]# vim /etc/nginx/conf.d/default.conf

```

```
#
# The default server
#

server {
    listen       80 default_server;
   # listen       [::]:80 default_server;
    server_name  _;
    root         /usr/share/nginx/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {

    }

    location /test{
        #root /;
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }

}

```

保存退出然后关闭nginx后重新运行

```
[root@VM_239_130_centos html]# pkill -9 nginx
[root@VM_239_130_centos html]# /usr/sbin/nginx

```
此时访问是找不到网页，这是因为我们新添加了一个location资源路径的配置，他会自动找到root，然后在去找root下面是否有test这个文件夹，有的话就去test文件夹中去找我们访问的test.html，可想而知，我们并没有建立test文件夹

回到 /usr/share/nginx/html，然后建立test文件夹，将test.html移动到test文件夹中

```
[root@VM_239_130_centos html]# mkdir test
[root@VM_239_130_centos html]# mv test.html test/test.html
[root@VM_239_130_centos html]# ls
404.html  50x.html  hello  index.html  nginx-logo.png  poweredby.png  test  world
[root@VM_239_130_centos html]# cd test
[root@VM_239_130_centos test]# ls
test.html

```

这里要注意一下，我增加的location配置是这样的

```
location /test{
        #root /;
    }
```

如果这个root配置不注释的话将会覆盖server块下的root路径哦，到这里nginx基本配置应该就写完了，大家可以去亲自试一试~
