# nginx 配置详解

标签（空格分隔）： nginx

---

[toc]

## nginx 常用功能

### Http代理

- 正向代理
![正向代理](../md/nginx/代理.jpg)

- 反向代理
![反向代理](../md/nginx/反向代理.jpg)

### 负载均衡

> nginx 内置负载均衡策略有轮询，加权轮询，Ip hash。

- 轮询
![轮询](../md/nginx/轮询.jpg)

- 加权轮询
![加权轮询](../md/nginx/加权轮询.jpg)

- IP Hash
![IP Hash](../md/nginx/ip%20hash.jpg)

### web 缓存

> `Nginx` 可以对不同的文件做不同的缓存处理，配置灵活，并且支持`FastCGI_Cache`，主要用于对 `FastCGI` 的动态程序进行缓存。配合着第三方的 `ngx_cache_purge`，对制定的 `URL` 缓存内容可以的进行增删管理。

## 配置文件

### 文件结构

```cfg
# 全局块
...                             

# events块
events {                        
   ...
}

# http块
http {
    # server块
    server { 
        # location块
        location [PATTERN]{
            ...
        }
    }
}
```

- **全局块**：配置影响nginx全局的指令。
- **events块**：配置影响nginx服务器或与用户的网络连接。
- **http块**：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。
- **server块**：配置虚拟主机的相关参数，一个`http`中可以有多个`server`。
- **location块**：配置请求的路由，以及各种页面的处理情况。

### 常见配置项

- `$remote_addr` 与 `$http_x_forwarded_for`：用以记录客户端的ip地址；
- `$remote_user` ：用来记录客户端用户名称；
- `$time_local` ： 用来记录访问时间与时区；
- `$request` ： 用来记录请求的url与http协议；
- `$status` ： 用来记录请求状态；成功是200；
- `$body_bytes_s ent` ：记录发送给客户端文件主体内容大小；
- `$http_referer` ：用来记录从那个页面链接访问过来的；
- `$http_user_agent` ：记录客户端浏览器的相关信息；

### 配置示例

```cfg
#定义Nginx运行的用户和用户组
user www www;
#
#nginx进程数,建议设置为等于CPU总核心数.
worker_processes 8;
#
#全局错误日志定义类型,[ debug | info | notice | warn | error | crit ]
error_log /var/log/nginx/error.log info;
#
#进程文件
pid /var/run/nginx.pid;
#
#一个nginx进程打开的最多文件描述符数目,理论值应该是最多打开文件数（系统的值ulimit -n）与nginx进程数相除,但是nginx分配请求并不均匀
worker_rlimit_nofile 65535;
#
#工作模式与连接数上限
events
{
    #参考事件模型,use [ kqueue | rtsig | epoll | /dev/poll | select | poll ];
    use epoll;
    #单个进程最大连接数（最大连接数=连接数*进程数）
    worker_connections 65535;
}
#
#设定http服务器
http
{
    include mime.types;                                 #文件扩展名与文件类型映射表
    default_type application/octet-stream;              #默认文件类型
    #charset utf-8;                                     #默认编码
    server_names_hash_bucket_size 128;                  #服务器名字的hash表大小
    client_header_buffer_size 32k;                      #上传文件大小限制
    large_client_header_buffers 4 64k;                  #设定请求缓存
    client_max_body_size 8m;                            #设定请求缓存
    
    # 开启目录列表访问,合适下载服务器,默认关闭.
    autoindex on;                                       # 显示目录
    autoindex_exact_size on;                            # 显示文件大小 默认为on
    autoindex_localtime on;                             # 显示文件时间 默认为off
    
    sendfile on;                                        # 开启高效文件传输模式
    tcp_nopush on;                                      # 防止网络阻塞
    tcp_nodelay on;                                     # 防止网络阻塞
    
    keepalive_timeout 120; # (单位s)设置客户端连接保持活动的超时时间,在超过这个时间后服务器会关闭该链接
    
    # FastCGI相关参数是为了改善网站的性能：减少资源占用,提高访问速度.
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    
    # gzip模块设置
    gzip on;                                            #开启gzip压缩输出
    gzip_min_length 1k;                                 #允许压缩的页面的最小字节数
    gzip_buffers 4 16k;                                 #表示申请4个单位为16k的内存作为压缩结果流缓存
    gzip_http_version 1.1;                              #压缩版本
    gzip_comp_level 2;                                  #压缩等级
    #压缩类型,默认就已经包含text/html
    gzip_types text/plain application/x-javascript text/css application/xml; 
    gzip_vary on;                                       #可以让前端的缓存服务器缓存经过gzip压缩的页面
    
    #开启限制IP连接数的时候需要使用
    #limit_zone crawler $binary_remote_addr 10m;
    
    #虚拟主机的配置
    server
    {
        # 监听端口
        listen 80;
        # 域名可以有多个,用空格隔开
        server_name ably.com;
        # HTTP 自动跳转 HTTPS
        rewrite ^(.*) https://$server_name$1 permanent;
    }
    
    server
    {
        # 监听端口 HTTPS
        listen 443 ssl;
        server_name ably.com;
        # 配置域名证书
        ssl_certificate C:\WebServer\Certs\certificate.crt;
        ssl_certificate_key C:\WebServer\Certs\private.key;
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout 5m;
        ssl_protocols SSLv2 SSLv3 TLSv1;
        ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
        ssl_prefer_server_ciphers on;
        index index.html index.htm index.php;
        root /data/www/;
        location ~ .*\.(php|php5)?$
        {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
        }
        
        # 配置地址拦截转发，解决跨域验证问题
        location /oauth/{
            proxy_pass https://localhost:13580/oauth/;
            proxy_set_header HOST $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        
        # 图片缓存时间设置
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
            expires 10d;
        }
        
        # JS和CSS缓存时间设置
        location ~ .*\.(js|css)?$ {
            expires 1h;
        }
        
        # 日志格式设定
        log_format access '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" $http_x_forwarded_for';
        # 定义本虚拟主机的访问⽇志
        access_log /var/log/nginx/access.log access;
        
        # 设定查看Nginx状态的地址.StubStatus模块能够获取Nginx自上次启动以来的工作状态，此模块非核心模块，需要在Nginx编译安装时手动指定才能使用
        location /NginxStatus {
            stub_status on;
            access_log on;
            auth_basic "NginxStatus";
            auth_basic_user_file conf/htpasswd;
            #htpasswd文件的内容可以用apache提供的htpasswd工具来产生.
        }
    }
}
```