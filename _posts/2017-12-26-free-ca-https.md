---
layout: post
title: "申请免费ca证书搭建个人HTTPS站点"
date: 2017-12-26
description: "腾讯云免费证书申请构建HTTPS站点"
tag: HTTP/HTTPS 
---   


**看着别人的网站在浏览器地址栏显示"安全", 自己的网站显示警告, 甚不爽, 决定自己也搞一个HTTPS, 采坑之旅由此开始**

最终完成好的效果如下:

![HTTPS站点](/images/https-safe.png)

### **搭建好个人HTTPS网站需要经历:**
* 1.HTTPS和证书前置知识
* 2.成为阿里云/腾讯云开发者
* 3.提前购买好域名备案通过
* 4.个人云服务器和web服务
* 5.腾讯云上申请免费ca证书
* 6.域名添加TXT记录
* 7.nginx配置SSL证书(基本的vim操作)

下面我们开始采坑之旅...

### 1.HTTPS和证书前置知识
* **HTTPS协议**: 
在HTTP协议的基础上, 在传输层加密, 一旦建立SSL连接, 中间过程的流量都是加密的, 可以一定程度上保证你的信息不被窃取, 篡改
* **证书**: 
证书本质上网站加密解密的密钥, 加密过程采用非对称加密 RSA密钥算法, 生成两个密钥, 一个是公钥进行加密, 另一个是私钥, 用于解密; 网站用一个密钥加密, 然后浏览器请求证书颁发机构的密钥用于解密, 这样就形成了加密与解密的闭环. 所以我们要构建自己的HTTPS网站, 必须去申请一个证书. 腾讯云上面可以申请免费的ca证书, 证书一般有效期一年.


### 2.成为阿里云/腾讯云开发者
你要构建自己的HTTPS站点, 你首先需要去注册成为腾讯云或者阿里云的开发者, 通过实名认证. 在腾讯云上可以申请免费的ca证书, 阿里云上目前好像没有免费证书了.

### 3.提前购买好域名备案通过
你可以在腾讯云或者阿里云上注册购买相关域名, 然后把你要使用的域名先在备案系统备案, 例如阿里云的备案地址: https://beian.aliyun.com/, 备案根据要求需要提交很多信息, 你照着做就是了, 值得注意的是 2018.1.1日起, 备案要求更高了, 其中顶级域名 .cc暂时不支持在北京备案了.备案成功后, 意味着你可以把域名解析到国内的服务器上了.

### 4.个人云服务器和web服务
你要备案成功, 需要先购买阿里云,腾讯云的备案服务号才可以, 所以你必须先购买服务器才能申请到备案服务号备案成功.购买服务器之后你可以 按照你的需求, 搭建 nodejs/python等web服务, 用 nginx做代理.你不妨提前去了解一下, nginx基本操作和vim, 以便你在服务器的道路上走的更顺利.

### 5/6.腾讯云上申请免费ca证书和添加域名解析**
搭建HTTPS站点的前提是你的HTTPS站点可以跑起来, 如果这个要求你都达不到, 请先搭建你的web服务, 让它跑起来.
现在我们可以去腾讯云申请免费的ca证书了,进入 https://console.cloud.tencent.com/ssl, 然后选择 "申请证书"/"申请免费版DVSSL证书", 然后可能出现下面的表单需要填写:
![HTTPS站点](/images/apply-ca.png)
, 你填写好你的域名,邮箱,密码等相关信息

然后他会要求你选择, 添加DNS记录的方式, 你选择 "手动DNS验证"
![HTTPS站点](/images/dns-check.png)
然后最关键的, 腾讯云会给你 一个 TXT 域名解析记录, 如下图:
![HTTPS站点](/images/dns-txt.jpg)
此时, 你需要在域名控制台添加解析, 我现在去阿里云域名控制台给这个域名添加一个TXT解析
如下图:
![HTTPS站点](/images/aliyun-dns.jpg)
然后你可以回到腾讯云测试此条添加的解析是否生效, 成功后下载证书到本地, 你会得到 
![HTTPS站点](/images/download-ca.png)
到这里你已经完成了60%的任务了, 剩下的工作就是参见腾讯云nginx配置证书的文档了

### * 7.nginx配置SSL证书(基本的vim操作)
下载证书后可以参见腾讯云开发者文档: https://cloud.tencent.com/document/product/400/4143
修改 nginx.conf文件, nginx配置时需要监听 80 端口和 443端口(HTTPS用)
我的配置如下(仅供参考):

```
user www;
worker_processes auto;

error_log /data/wwwlogs/error_nginx.log crit;
pid /var/run/nginx.pid;
worker_rlimit_nofile 51200;

events {
  use epoll;
  worker_connections 51200;
  multi_accept on;
}

http {
  include mime.types;
  default_type application/octet-stream;
  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 1024m;
  client_body_buffer_size 10m;
  sendfile on;
  tcp_nopush on;
  keepalive_timeout 120;
  server_tokens off;
  tcp_nodelay on;
  limit_req_zone $binary_remote_addr zone=allips:10m rate=2r/s; #每个ip每秒1个请求, allips是变量名
  limit_conn_zone $binary_remote_addr zone=perip:10m; # ip连接数处理, one是变量名

  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 4 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;
  fastcgi_intercept_errors on;

  #Gzip Compression
  gzip on;
  gzip_buffers 16 8k;
  gzip_comp_level 6;
  gzip_http_version 1.1;
  gzip_min_length 256;
  gzip_proxied any;
  gzip_vary on;
  gzip_types
    text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
    text/javascript application/javascript application/x-javascript
    text/x-json application/json application/x-web-app-manifest+json
    text/css text/plain text/x-component
    font/opentype application/x-font-ttf application/vnd.ms-fontobject
    image/x-icon;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";

  #If you have a lot of static files to serve through Nginx then caching of the files' metadata (not the actual files' contents) can s
ave some latency.
  open_file_cache max=1000 inactive=20s;
  open_file_cache_valid 30s;
  open_file_cache_min_uses 2;
  open_file_cache_errors on;

  server {
    listen 80;
    server_name xxxxxxxx.com;
    rewrite ^(.*)$  https://$host$1 permanent;
    charset utf-8;
    client_max_body_size 75M;
    access_log /data/wwwlogs/access_nginx.log combined;
    root /data/wwwroot/default;
    index index.html index.htm index.php;
    location /nginx_status {
      stub_status on;
      access_log off;
      allow 127.0.0.1;
      deny all;
    }
    location /static/ {
      root  /data/rms;
      expires 30d;
    }
    
    location / {
        include    uwsgi_params; 
        uwsgi_pass 127.0.0.1:8001;
        limit_req zone=allips burst=5 nodelay; # 每秒请求数限制
        limit_conn perip 2;  #连接数限制
        limit_rate 500k; 
        # proxy_set_header   Host             $host;
        # proxy_set_header   X-Real-IP        $remote_addr;
        # proxy_set_header   X-Forwarded-URI  $uri;
        # proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        # client_max_body_size    50m;
    }
  }

    # default ############################
  server {
    listen 443;
    server_name xxxxxxxx.com; #填写绑定证书的域名
    ssl on;
    ssl_certificate 1_xxxxxxxx.com_bundle.crt;
    ssl_certificate_key 2_xxxxxxxx.com.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; #按照这个套件配置
    ssl_prefer_server_ciphers on;
    location /static/ {
      root  /data/rms;
      expires 30d;
    }
    location / {
      include    uwsgi_params; # the uwsgi_params file you installed
      uwsgi_pass 127.0.0.1:8001;
      limit_req zone=allips burst=5 nodelay; # 每秒请求数限制
      limit_conn perip 2;  #连接数限制
      limit_rate 500k; 
      # proxy_set_header   Host             $host;
      # proxy_set_header   X-Real-IP        $remote_addr;
      # proxy_set_header   X-Forwarded-URI  $uri;
      # proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
      # client_max_body_size    50m;
    }
  }
  include vhost/*.conf;
}

```
修改完之后重新加载 一下nginx, 执行 nginx -s reload 即可...

到这里正常情况下, 你的网站就可以用上HTTPS协议啦, 被浏览器标识为 "安全" ,是不是很 happy? 

不说了, 本次表演完毕^_^,  我去学习了...


转载请注明原地址，我的新版博客：[首发博客](http://iceyangcc.github.io) 谢谢！
