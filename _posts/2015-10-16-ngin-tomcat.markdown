---
layout: post
title:  "Nginx 与 tomcat 反向代理配置"
date:   2015-10-16 11:02:15
categories: 服务器
tags: Nginx tomcat 反向代理
---
由于本人资金有限，所以经常需要一个服务器中同时运行多个网址，但是问题来了，域名解析是只能80端口进入的，不支持其他端口，而且也不能带后缀，所以如何让多个网址都能进行域名解析成了一个问题，后面百度发现有反向代理这个功能，但是一般Nginx服务器的方向代理教程都是教你如何指向一个虚拟目录的，这个对于用tomcat的人来说我不知道到底能不能行，反正我是没有成功配置过，不过后来发现也可以直接指向网路地址，废话少说，上配置
##### 首先说明，本配置基于Nginx 1.8.0，其他版本的情况不清楚，需要自己测试
首先下载Nginx 1.8.0，下载地址[Nginx] 下载地址，下载好后解压（windows上面是直接解压，其他系统的不清楚，不过估计也差不多），然后进入目录conf，打开nginx.conf，进行编辑。

现在的情况是tomcat占用8080端口，web程序路径为localhost:8080/blog,而我们的目标就是要利用Nginx的反向代理，使用localhost直接访问到localhost:8080/blog,当然如果有域名和服务器也可以用你的域名和服务器测试。
主要配置如下（nginx.conf）
{% highlight ruby linenos%}
server
 {
     listen 80;
     server_name localhost;
     location /blog/ {
   proxy_redirect off;
   proxy_set_header Host $host;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $http_x_forwarded_for; 
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_pass http://localhost:8080/blog/;
            sub_filter_once off;
     }
     location / {
   proxy_redirect off;
   proxy_set_header Host $host;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $http_x_forwarded_for; 
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_pass http://localhost:8080/blog/;
            sub_filter_once off;
     }
   access_log logs/blog.log;
 }
{% endhighlight %}
server代表着一个监听服务，listen代表着的监听端口，这里是80（http默认端口），server_name代表着要被监听的名字，我们这里是localhost（域名），例如我的域名是www.raye.wang就需要填写:

```
server_name www.raye.wang;
```
##### 注意

##### 一个很现实的情况是，别人有可能是通过www.raye.wang 来访问，也有可能是通过raye.wang 来访问，所以最好建立2个service，当然如果有其他更好的办法也请不啬赐教
location /blog/ 这个配置是为了防止web程序指定了完整目录而配置的，结果就是会将localhost/blog/继续转发成为localhost:8080/blog,而不是localhost:8080/blog/blog/,当然这个可以根据具体情况配置，简单来说真正的匹配规则就是server_name+location,之所以要把/blog/放前面是为了防止其他先匹配到，比如location / ,Nginx一旦匹配上就不会再匹配下一个了。

proxy_pass 很显然就是跳转的地址了，这个就必须特别了说明了，下面的location很明显是匹配所有localhost/的了，一般这个都是放在最后面的。

access_log就是记录日志的配置，这个无需多说，至于其他配置，我也没有具体搞清楚。

好了现在运行Nginx.exe之后就能进行反向代理了，注意，这个运行后只会闪烁一个黑框，如果任务管理器里面发现2个Nginx线程，说明运行成功了。
##### 百说不如一试，自己配置试试吧
另外附上我的nginx.conf
{% highlight ruby linenos%}
worker_processes  1;


events {
    worker_connections  1024;
}

http {
     include       mime.types;
    default_type  application/octet-stream;
    sendfile on;
    keepalive_timeout 65;
    gzip on;
    client_max_body_size 50m;
    client_body_buffer_size 256k;
    client_header_timeout 3m;
    client_body_timeout 3m;
    send_timeout 3m;
    proxy_connect_timeout 300s;
    proxy_read_timeout 300s;
    proxy_send_timeout 300s;
    proxy_buffer_size 64k;
    proxy_buffers 4 32k;
    proxy_busy_buffers_size 64k;
    proxy_temp_file_write_size 64k;
    proxy_ignore_client_abort on;

 server
 {
     listen 80;
     server_name localhost;
     location /blog/ {
   proxy_redirect off;
   proxy_set_header Host $host;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $http_x_forwarded_for; 
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_pass http://localhost:8080/blog/;
            sub_filter_once off;
     }
     location / {
   proxy_redirect off;
   proxy_set_header Host $host;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $http_x_forwarded_for; 
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_pass http://localhost:8080/blog/;
            sub_filter_once off;
     }
     access_log logs/blog.log;
 }
}
{% endhighlight %}
[Nginx]:http://nginx.org/en/download.html
