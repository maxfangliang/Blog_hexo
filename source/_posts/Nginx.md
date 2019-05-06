---
title: Nginx入门
date: 2019-05-06 14:08:14
categories: 
- Lunix
- Nginx
tags: [Lunix,Nginx]
toc: true
---


[资料来源](http://jspang.com/post/nginx.html)

> Nginx是一款轻量级的HTTP服务器，采用事件驱动的异步非阻塞处理方式框架，这让其具有极好的IO性能，时常用于服务器的反向代理和负载均衡。

**Nginx的优点**

- 支持海量高并发：采用IO多路复用epoll。官方测试Nginx能够支持5万并发链接，实例生产环境中可以支持2-4万并发连接数。
- 内存消耗少：在主流的服务器中Nginx目前是内存消耗最小的，比如我们用Nginx + PHP，在3万并发链接下，开启10个Nginx进程消耗为150M内存。
- 免费使用可以商业化：Ngnix为开源软件，采用的是2-clause BSD-like协议，开源免费使用，并且可以用于商业。
- 配置文件简单：网络和程序配置通俗易懂，即使非专业运维也能看懂。
- 反向代理功能，负载均衡功能。

<!--more-->

**Nginx版本说明**

- Mainline version：开发版
- Stable version：稳定版，长期更新版本。用于生产环境
- legacy version：历史版本


## Yum安装Nginx

查看`yum`是否存在：
```
yum list | grep nginx
```

如果出现类似下面的内容，就说明`yum`源是存在的。

![image](https://note.youdao.com/yws/api/personal/file/173DE36FB4FC4FE6B1184E6DB1550E3B?method=download&shareKey=448dfdf8d204c2ab4c54a63f28222697)

如果不存在`yum`，或者版本不是想要的，可以自行配置`yum`源。具体google

**安装Nginx**

安装命令

```
yum install nginx
```

检测Nginx版本

```
nginx -v
```


## Nginx 基本配置

**查看Nginx的安装目录**

在使用`yum`安装完`Nginx`后，需要查看安装到哪个位置。
```
rpm -ql nginx
```

`rpm`是`linux`的`rpm`包管理工具，`-q`代表询问模式，`-l`代表返回列表，这样我们就可以找到`nginx`的所有安装位置。


*Nginx.conf文件解读*

`nginx.conf`文件是`Nginx`总配置文件。

进入`etc/nginx`目录下，然后用`vim`打开


```
cd /etc/nginx   // 服务器可能存在多个nginx安装目录，不一定在这个目录
vim nginx.conf
```

下面是文件的详细注释:

```
#运行用户，默认即是nginx，可以不进行设置
user  nginx;
#Nginx进程，一般设置为和CPU核数一样
worker_processes  1;   
#错误日志存放目录
error_log  /var/log/nginx/error.log warn;
#进程pid存放位置
pid        /var/run/nginx.pid;
events {
    worker_connections  1024; # 单个后台进程的最大并发数
}
http {
    include       /etc/nginx/mime.types;   #文件扩展名与类型映射表
    default_type  application/octet-stream;  #默认文件类型
    #设置日志模式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;   #nginx访问日志存放位置
    sendfile        on;   #开启高效传输模式
    #tcp_nopush     on;    #减少网络报文段的数量
    keepalive_timeout  65;  #保持连接的时间，也叫超时时间
    #gzip  on;  #开启gzip压缩
    include /etc/nginx/conf.d/*.conf; #包含的子配置项位置和文件
```

*default.conf 配置项*

当前目录下有一个子文件的配置项，打开这个`include`子文件配置项

进入`conf.d`目录，然后用`vim default.conf`进行查看

```
server {
    listen       80;   #配置监听端口
    server_name  localhost;  //配置域名
    #charset koi8-r;     
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;     #服务默认启动目录
        index  index.html index.htm;    #默认访问文件
    }
    #error_page  404              /404.html;   # 配置404页面
    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;   #错误状态码的显示页面，配置后需要重启
    location = /50x.html {
        root   /usr/share/nginx/html;
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

```

明白了这些配置，就知道服务器目录放在`/usr/share/nginx/html`下，可以使用命令进入查看一下目录下的文件

```
cd /urs/share/nginx/html
ls
```

可以看到目录下有两个文件，`50x.html`和`index.html`。可以使用`vim`进行编辑。

到这里可以预想到，`Nginx`服务器已经可以为`html`提供服务了。

*阿里云的安全组配置*

如果使用的是阿里云，记得到`ESC`实例打开一下端口。

![image](https://note.youdao.com/yws/api/personal/file/75BE3B541935423A9F63BD256DCC7108?method=download&shareKey=c0deebe2f859d779bb19d3690ab0b52d)



## Nginx 服务启动、停止、重启

*启动Nginx服务*

默认的情况下，Nginx是不会自动启动的，需要手动进行启动。

*Nginx直接启动*

在`CentOS7.4`版本里，可以直接使用`nginx`启动服务。

```
nginx
```

*使用systemctl命令启动*

这种方法无论启动什么服务，都是一样的，只需要换一下服务的名字

```
systemctl start nginx.service
```

`Linux`命令查询服务的运行情况

```
ps aux | grep nginx
```

启动成功出现如下图片类似结果：

![image](https://note.youdao.com/yws/api/personal/file/4E2A6979C0B24C2BAA1C3EDEC7DBE417?method=download&shareKey=21fc870d120fedba81c90ade11449f4e)


*停止Nginx服务的四种方法*

- 立即停止服务
```
nginx -s stop       // 直接停止进程
```

- 从容停止服务
```
nginx -s quit       // 需要进程完成当前工作后停止
```

- killall 方法杀死进程
```
killall nginx
```

- systemctl 停止
```
systemctl stop nginx.service
```


*重启Nginx服务*

```
systemctl restart nginx.service
```


*重新载入配置文件*

在重新编写或修改`Nginx`的配置文件后，重新载入
```
nginx -s reload
```


*查看端口号*

在默认情况下，`Nginx`启动后会监听80端口，从而提供HTTP访问，如果80端口已经被占有则会启动失败。

```
netstat -tlnp           // 查看端口号的占用情况
```



## 自定义错误页和访问设置


*多错误指向一个页面*

在`/etc/nginx/conf.d/default.conf`可以看到下面的代码
```
error_page  500 502 503 504  /50x.html;
```

`error_page`指令用于自定义错误页面，500、502、503、504这戏就是HTTP中最常见的错误代码，`50x.html`用于表示当发生上述指定的任意一个错误时，都用网站根目录的`/50x.html`文件进行处理。


*单独为错误置顶处理方式*

有些时候要把这些错误页面单独的表现出来，带给用户更好的体验，所有要为每个错误设置不同的页面

```
error_page  404 /404_error.html;
```

然后在网站目录下新建一个`404_error.html`文件
```
<html>
<meta charset="UTF-8">
<body>
<h1>404页面没有找到!</h1>
</body>
</html>
```


*把错误码换成一个地址*

处理错误的时候，不仅可以只使用本服务器的资源，还可以使用外部资源。比如将配置文件设置成这样
```
error_page 404 https://www.google.com/
```
没有找到文件，直接跳转到google




## Nginx 访问权限

*简单实现访问控制*

有时候服务器只允许特定主机访问，比如内部`OA`系统，或者应用的管理后台系统，更或者是某些应用接口，直接在`default.conf`里进行配置
```
location / {
    deny    100.20.0.100        // deny 禁止访问
    allow   172.16.17.100       // allow 允许访问
}
```
配置完成，重启一下服务器就可以实现限制和运行访问了。


*指令优先级*

```
location / {
    allow   45.76.200.230;
    deny    all;
}
```
上面的配置表示只允许`45.76.200.230`进行访问，其他IP是禁止访问的。但是如果把`deny all`指令移动到`45.76.200.230`之前，会发现所有的IP的都不允许访问。*这说明在同一个块下的两个权限指令，先出现的设置会覆盖后出现的设置（谁先触发，谁起作用）*


*复杂访问控制权限匹配*

在实际开发中，访问权限的控制需求更加复杂。例如，对于网站下的`img`(图片目录)是运行所有用户访问，但是对于网站下的`admin`目录则只允许公司内部固定IP访问。仅靠`deny`和`allow`这两个指令是无法实现的。

需要`location`块来完成相关的匹配：
```
location =/img {
    allow   all;
}

location =/admin {
    deny    all;
}
```

`=`号代表精确匹配，使用了`=`后是根据其后的模式进行精确匹配。


*使用正则表达式设置访问权限*

只有精确匹配有时完不成需求，比如要禁止访问所有的`php`页面，可以使用正则
```
location ~\.php$ {
    deny    all;
}
```



## Nginx 设置虚拟主机

> 虚拟主机是指在一台物理主机服务器上划分出多个磁盘空间，每个磁盘空间都是一个虚拟主机，每台虚拟主机都可以对外提供Web服务，并且互不干扰。在外界看来，虚拟主机就是一台独立的服务器主机，这意味着用户能够利用虚拟主机把多个不同域名的网站部署在同一台服务器上，而不必再为简历一个网站单独购买一台服务器，既解决了维护服务器技术的难题，同时又极大地节省了服务器硬件成本和相关的维护费用。

配置虚拟主机可以基于端口号、基于IP和基于域名。


*基于端口号配置虚拟主机*

基于端口号来配置虚拟主机，算是`Nginx`中最简单的一种方式。原理就是`Nginx`监听多个端口，根据不同的端口号，来区分不同网站。

可以直接配置在主文件里`/etc/nginx/nginx.conf`文件里（根据自己的nginx目录），也可以配置在子配置文件里`/etc/nginx/conf.d/default.conf`。只要在`conf.d`文件夹下就可以了。

修改配置文件中的`server`选项，这时候就会有两个`server`。
```
server {
    listen  8001;
    server_name localhost;
    root    /usr/share/nginx/html/html8001;
    index   index.html;
}
```

编写`/usr/share/nginx/html/html8001/`目录下的`index.html`查看结果。
```
<h1>hello port 8001</h1>
```
最后在浏览器中分别访问地址和带端口的地址，看到的结果是不同的。


*基于IP的虚拟主机*

基于IP和基于端口的配置几乎一样，只是把`server_name`选项，配置成IP就可以了。

比如上门的配置，可以修改为：
```
server {
    listen  80;
    server_name 47.99.99.138;
    root    /usr/share/nginx/html/html8001;
    index   index.html;
}
```
这种演示需要多个IP的支持，由于阿里云ECS只提供一个IP，无法进行演示。



## Nginx 使用域名设置虚拟主机

在真实的上线环境中，一个网站是需要域名和公网IP才可以访问的。

先要对域名进行解析，这样域名才能正确定位到需要的IP上。这里新建两个解析，分别是：
- fangliang.pro：映射到默认的Nginx首页位置
- maxfangliang.pro：映射到原来的8001端口位置


*配置以域名为划分的虚拟主机*

修改`etc/nginx/conf.d`目录下的`default.conf`文件，把原来的80端口虚拟主机改为以域名划分的虚拟主机。
```
server {
    listen      80;
    server_name fangliang.pro;
}
```

修改同目录下的`8001.conf`文件，如下
```
server {
    listen      8001;
    server_name maxfangliang.pro;
    location / {
        root /usr/share/nginx/html/html8001;
        index index.html index.htm;
    }
}
```
重启`Nginx`，就可以通过域名访问这两个网页。

域名设置虚拟主机主要是配置文件的`server_name`项，需要域名解析的配合。



## Nginx 反向代理的设置

> 现在的web模式基本都是标准的CS结构，即Client端到Server端。代理就是在Client端和Server端之间增加一个提供特定功能的服务器，这个服务器就是代理服务器。

### 正向代理

翻墙工具就是一个典型的正向代理工具。它会把国内不让访问的服务器的网页请求，代理到一个可以访问该网站的代理服务器上来，一般叫做`proxy`服务器，再转发给客户。

![image](https://note.youdao.com/yws/api/personal/file/8343D7F0C5174D0FADB9B58CFA303580?method=download&shareKey=920ce2eac939e4631db6344dabf8c8d1)

原理就是代理服务器有访问目标服务器的权限，可以通过访问代理服务器，代理服务区访问真实服务区，把要访问的内容呈现出来。


### 反向代理

反向代理跟正向代理正好相反（目前基本所有的大型网站的页面都用了反向代理），客户端发送的请求，想要访问`server`服务器上的内容。发送的内容被发送到代理服务器上，代理服务器再把请求发送到设置好的内容服务器上，用户想获得的内容就在这些设置好的服务器上。

![image](https://note.youdao.com/yws/api/personal/file/CD856C75B081440D858C0E126B7E3AF7?method=download&shareKey=a7d8230ef0b3fcbe930f463388647a1a)

这里的`proxy`服务器代理的并不是客户端，而是服务器，即向外部客户端提供了一个统一的代理入口，客户端的请求都要经过这个`proxy`服务器。具体访问哪个服务器`server`是由`Nginx`来控制的。

一般代理指代理的客户端，反向代理是代理的服务器。


*反向代理的用途和好处*

- 安全性：正向代理的客户端能够在隐藏自身信息的同时访问任意网站，这个给网络安全代理了极大的威胁。因此，我们必须把服务器保护起来，使用反向代理客户端用户只能通过外来网来访问代理服务器，并且用户并不知道自己访问的真实服务器是那一台，可以很好的提供安全保护。
- 功能性：反向代理的主要用途是为多个服务器提供负债均衡、缓存等功能。负载均衡就是一个网站的内容被部署在若干服务器上，可以把这些机子看成一个集群，那Nginx可以将接收到的客户端请求“均匀地”分配到这个集群中所有的服务器上，从而实现服务器压力的平均分配，也叫负载均衡。
- 


*最简单的反向代理*

现在想访问`fangliang.pro`然后反向代理到`maxfangliang.pro`（示例）。到`etc/nginx/con.d/8001.conf`进行配置。

```
server {
    listen      80;
    server_name fangliang.pro;
    location / {
        proxy_pass maxfangliang.pro;
    }
}
```

一般反向代理的都是IP，但是这里代理域名也是可以的。


*其它反向代理指令*

- proxy_set_header：在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息。
- proxy_connect_timeout：配置Nginx与后端代理服务器尝试建立连接的超时时间。
- proxy_read_timeout：配置Nginx向后端服务器组发出read请求后，等待相应的超时时间。
- proxy_send_timeout：配置Nginx向后端服务器组发出write请求后，等待相应的超时时间。
- proxy_redirect：用于修改后端服务器返回的响应头中的Location和Refresh。



## Nginx 适配PC或移动设备

### $http_user_agent的使用

`Nginx`通过内置变量`$http_user_agent`，可以获取到请求客户端的`userAgent`，就可以判断用户目前处于移动端还是PC端，进而展示不同的页面给用户。

操作步骤：

1. 在`/usr/share/nginx/`目录下新建两个文件，分别为：PC和mobile目录
    ```
    cd /usr/share/nginx
    mkdir PC
    mkdir mobile
    ```
    
2. 在PC和mobile目录下，新建两个`index.html`文件，分别写上对应的内容

3. 进入`etc/nginx/conf.d`目录下，修改`8001.conf`文件，改为下面的内容：

    ```
    server {
        listen      80;
        server_name fangliang.pro;
        location / {
            root /usr/share/nginx/PC;
            if ($http_user_agent ~* '(Android|webOS|iPhone|iPod|BlackBerry)') {
                root /usr/share/nginx/mobile;
            }
            index index.html;
        }
    }
    ```


## Nginx的Gzip压缩配置

> Gzip是网页的一种网页压缩技术，经过Gzip压缩后，页面大小可以变为原来的30%甚至更小。更小的网页会让用户体验更好，速度更快。Gzip网页压缩的实现需要浏览器和服务器的支持。

![image](https://note.youdao.com/yws/api/personal/file/A27C4BCD3C2B49459D3D1483BE702240?method=download&shareKey=1120050fa8da538762ebcb40702a1ec1)

> 从上图可以清楚的明白，gzip是需要服务器和浏览器同事支持的。当浏览器支持gzip压缩时，会在请求消息中包含Accept-Encoding:gzip,这样Nginx就会向浏览器发送听过gzip后的内容，同时在相应信息头中加入Content-Encoding:gzip，声明这是gzip后的内容，告知浏览器要先解压后才能解析输出。


### Gzip的配置项

`Nginx`提供了专门的`Gzip`模块，并且模块中的指令非常丰富。

- gzip：该指令用于开启或关闭`gzip`模块。
- giip_buffers：设置系统获取几个单位的缓存用于存储`gzip`的压缩结果数据流。
- gzip_comp_level：`gzip`压缩比，压缩级别是1-9，1的压缩级别最低，9的压缩级别最高。压缩级别越高压缩率越大，压缩时间越长。
- gzip_disable：可以通过该指令对一些特定的`User-Agent`不使用压缩功能。
- gzip_min_length：设置允许压缩的页面最小字节数，页面字节数从相应消息头的`Content-length`中进行获取。
- gzip_http_version：识别`HTPP`协议版本，其值可以是1.1或1.0。
- gzip_proxied：用于设置启用或禁用从代理服务器上收到响应内容`gzip`压缩。
- gzip_vary：用于在响应消息头中添加`Vary：Accept-Encoding`，使代理服务器根据请求头中的`Accept-Encoding`识别是否启用`gzip`压缩。


### Gzip最简单的配置

```
http {
    ...
    gzip on;
    gzip_types text/plain application/javascript text/css;
    ...
}
```

`gzip on`是启动Gzip模块，下面的一行是用于在客户端访问网页时，对文本、JavaScript和CSS文件进行压缩输出。

配置号后，就可以重启`Nginx`服务，让`Gzip`生效。

打开开发者工具，在标签中选择`Headers`，查看HTTP响应头信息。可以清楚的看见`Content-Encoding`为`gzip`类型。

![image](https://note.youdao.com/yws/api/personal/file/48C8B35D28A54D9FAFFE2D0F90E68BD4?method=download&shareKey=8220b4d677cd03d55061c335496e5e2c)





## Nginx 指令
```
# cat default.conf              进入 default.conf 编辑界面
```