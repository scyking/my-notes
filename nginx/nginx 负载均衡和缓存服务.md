# nginx 负载均衡和缓存服务

标签（空格分隔）： nginx

---

[toc]

## 负载均衡

> 将客⼾的请求通过 nginx 分发（反向代理）到⼀组多台不同的服务器上。
> 这⼀组服务器我们称为 **服务池**（`upstream server`），池内的每⼀个服务器称为⼀个**单元**，服务池内将对每⼀个单元进⾏请求轮训，实现负载均衡。

### 配置

```
upstream #自定义组名 {
    server x1.baidu.com;
    server x2.baidu.com;
    server x3.baidu.com
    #down 不参与负载均衡
    #weight=5; 权重，越高分配越多
    #backup; 预留的备份服务器
    #max_fails 允许失败的次数
    #fail_timeout 超过失败次数后，服务暂停时间
    #max_coons 限制最大的接受的连接数
    
    #根据服务器性能不同，配置适合的参数
    #server 106.xx.xx.xxx; 可以是ip、域名
    #server 106.xx.xx.xxx:8080; 可以带端口号
    #server unix:/tmp/xxx; 支持socket⽅式
}
```

### 调度算法

- 轮训：按时间顺序逐⼀分配到不同的后端服务器
- 加权轮训：weight值越⼤，分配到的⼏率越⾼
- `ip_hash`：每个请求按访问IP的hash结果分配，这样来⾃同⼀个IP固定访问⼀个后端服务器
- `least_conn`：最少链接数，哪个机器连接数少就分发给谁
- `url_hash`：按照访问的URL的hash结果来分配请求，每⼀个URL定向到同⼀个后端服务器
- `hash`关键数值：hash⾃定义key

PS：`ip_hash`存在缺陷，当前端服务器再多⼀层时，将获取不到⽤⼾的正确IP，获取的将是前⼀个前端服务器的IP。

## 缓存服务

### 缓存类型

- 服务端缓存：缓存存储在后端服务器，如redis,memcache
- 代理缓存：缓存存储在代理服务器或者中间件上，它的内容是从后端服务器获取的，但是保存在⾃⼰本地
- 客⼾端缓存：缓存在浏览器内的

### nginx代理缓存

> 客⼾端请求nginx，nginx查看本地是否有缓存数据，若有直接返回给客⼾端，若没有再去后端服务器请求。

```
http {
    proxy_cache_path /var/www/cache                         #缓存地址
    levels=1:2                                              #目录分级
    keys_zone=test_cache:10m                                #开启的keys空间名字:空间⼤⼩(1m可以存放8000个key)
    max_size=10g                                            #目录最大大小(超过时，不常⽤的将被删除)
    inactive=60m                                            #60分钟内没有被访问的缓存将清理
    use_temp_path=pff;                                      #是否开启存放临时文件目录，关闭默认存储在缓存地址
    server {
        ...
        location / {
            proxy_cache test_cache;                             #开启缓存对应的名称，在keys_zone命名好
            proxy_cache_valid 200 304 12h;                      #状态码为200 304的缓存12⼩时
            proxy_cache_valid any 10m;                          #其他状态缓存10⼩时
            proxy_cache_key $host$uri$is_args$args;             #设置key值
            add_header Nginx-Cache "$upstream_cache_status";
        }
    }   
}
```

### 分片请求

> 前端发起请求，nginx去获取这个请求⽂件的⼤⼩，若超过我们的定义slice的⼤⼩，会进⾏切⽚，分割成多个⼩的请求去请
求后端，到前端就成为⼀个⼀个独⽴的缓存⽂件。

- 优势：每个⼦请求收到的数据都会形成独⽴⽂件，⼀个请求中断了，其他请求不受影响，原本情况请求中断，再次请求⽂件将
从头开始，⽽开启分⽚请求，就接下去获取未请求的⼩⽂件
- 劣势：当⽂件很⼤或者slice很⼩时，可能会导致⽂件描述符耗尽等情况