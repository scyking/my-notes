# nginx 介绍

标签（空格分隔）： nginx

---

[toc]

## 概述

> Nginx是一个开源且高性能、可靠的Http中间件、代理服务。

## 优点

- IO多路复⽤`epoll`
- 轻量级
    1. 功能模块少，仅保留了Http需要的模块
    2. 代码模块化，适合二次开发
- CPU亲和
    把CPU核心和Nginx工作进程绑定，把每个worker进程固定在一个CPU上执行，减少切换CPU的cache miss，从而提高性能。

## windows上使用
    
### 命令

```
nginx -v        # 查看版本号
nginx -t        # 检查配置文件
start nginx     # 启动
nginx -s stop   # 快速停止或关闭
nginx -s quit   # 正常停止或关闭
nginx -s reload # 重载配置文件
```

### 注意事项

> 不要直接双击nginx.exe，这样会导致修改配置后重启、停止nginx无效，需要手动关闭任务管理器内的所有nginx进程。

### 错误

- 执行命令报错`bash: nginx: command not found`
解决：使用windows自带的命令行工具`cmd`。

## 配置
> http区域内科配置多个server，每个server对应一个虚拟主机或者域名。

### 基本配置

```
user                                # 设置nginx服务的系统使⽤⽤⼾
worker_processes                    # ⼯作进程数 ⼀般情况与CPU核数保持⼀致
error_log                           # nginx的错误⽇志
pid                                 # nginx启动时的pid
events {
    worker_connections              # 每个进程允许最⼤连接数
    use #nginx使⽤的内核模型
}
```

### http配置

```
http {
    sendfile on                             # ⾼效传输⽂件的模式 ⼀定要开启
    keepalive_timeout 65                    # 客⼾端服务端请求超时时间
    log_format main XXX                     # 定义⽇志格式 代号为main
    access_log /usr/local/access.log main   # ⽇志保存地址 格式代码 main
}
```

### server配置

```
server {
    listen 80                       # 监听端⼝;
    server_name localhost           # 地址
    location / {                    # 访问⾸⻚路径
        root /xxx/xxx/index.html    # 默认⽬录
        index index.html index.htm  # 默认⽂件
    }
    error_page 500 504 /50x.html    # 当出现以上状态码时从新定义到50x.html
    location = /50x.html {          # 当访问50x.html时
        root /xxx/xxx/html          # 50x.html ⻚⾯所在位置
    }
}
```

## 应用场景

### 静态资源web服务

1. 浏览器端渲染：HTML,CSS,JS
1. 图⽚：JPEG,GIF,PNG
1. 视频：FLV,MPEG
1. ⽂件：TXT，任意下载⽂件

### 浏览器缓存
> HTTP协议定义的缓存机制（如：Expires; Cache-control等 ）
> 减少服务端的消耗，降低延迟

### 跨站访问

```
location ~ .*\.(html|htm)$ {
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
    #Access-Control-Allow-Credentials true #允许cookie跨域
}
```

### 防盗链

```
location ~ .*\.(jpg|gif)$ {
    #valid_referers 表示我们允许哪些 $http_referer 来访问
    #none 表示没有带 $http_referer，如第一次访问时 $http_referer 为空
    #blocked 表示 $http_referer 不是标准的地址，非正常域名等
    #只允许此ip
    valid_referers none blocked 127.xxx.xxx.xx
    #不满足情况下变量值为1
    if ($invalid_referer) {                     
        return 403;
    }
}
```

PS:`$http_referer` ，表⽰当前请求上⼀次⻚⾯访问的地址。

### http代理服务

1. 正向代理，代理的对象是客户端
2. 反向代理，代理的对象是服务端

## 常见问题

### 常见错误码

- 413，上传文件过大
- 503，后端服务无响应
- 504，后端服务执行超时

### 同server_name多个虚拟主机优先级
> 优先选择最新读取到的配置⽂件，当多个⽂件是通过include时，⽂件排序越靠前，越早被读取。

### location匹配优先级

```
=       # 进行普通字符精确匹配，完全匹配
^~      # 进行普通字符匹配，当前表示前缀匹配
~\~*    # 表示执行一个正则匹配()

#当程序使用精确匹配时，一但匹配成功，将停止其他匹配
#当正则匹配成功时，会继续接下来的匹配，寻找是否还有更精准的匹配
```

### try_files的使用

> 按顺序检查文件是否存在。

```
location / {
    try_files $uri $uri/ /index.php;
}
#先查找$uri下是否有文件存在，若存在直接返回给用户
#若$url下没有文件存在，再次访问$uri/的路径是否有文件存在
#还是没有文件存在，交给index.php处理
例：
location / {
    root /test/index.html
    try_files $uri @test
}
location @test {
    proxy_pass http://127.0.0.1：9090;
}
#访问 / 时，查看 /test/index.html 文件是否存在
#若不存在，让9090端口的程序去处理这个请求
```

### `alias`和`root`区别

```
location /request_path/image/ {
    root /local_path/image/;
}
#当我们访问 http://xxx.com/request_path/image/cat.png时
#将访问 http://xxx.com/request_path/image/local_path/image/cat.png 下的文件

location /request_path/image/ {
    alias /local_path/image/;
}
#当我们访问 http://xxx.com/request_path/image/cat.png时
#将访问 http://xxx.com/local_path/image/cat.png 下的文件
```

### 获取用户真实IP

> 当一个请求通过多个代理服务器时，用户的ip将会被代理服务器IP覆盖。

```
#在第一个代理服务器中设置
set x_real_ip=$remote_addr
#最后一个代理服务器中获取
$x_real_ip=IP1
```
