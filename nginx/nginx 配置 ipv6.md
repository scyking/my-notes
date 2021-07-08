# nginx 配置 ipv6

标签（空格分隔）： nginx

---

[toc]

## 添加 ipv6 模块

1. 查看版本信息，执行如下命令：

    ```
    ./nginx -V
    ```

1. 查看`configure arguments`中是否包含ipv6模块`--with-ipv6`

1. 不存在，则添加ipv6模块，重新编译nginx
    - 进入nginx源码所在目录
    - 执行命令配置新的参数

        ```
        ## 按需添加其他模块
        ./configure --prefix=/usr/local/software/nginx/ --with-ipv6
        ```
    - 编译源码：`make`
    > `make install` 会覆盖原来的nginx文件。
    > 编译后的nginx在当前目录的`objs`文件夹内
    - 备份原来的nginx文件
    
        ```
        cp /usr/local/software/nginx/sbin/nginx /usr/local/software/nginx/sbin/nginx.bak
        ```
    - 关闭nginx服务：`nginx -s stop`
    - 替换nginx
    
        ```
        # 进入objs文件夹内，替换nginx
        cp nginx /usr/local/software/nginx/sbin/nginx
        ```
    - 启动nginx
    
## 配置文件

### 同时支持ipv4和ipv6配置

```
listen 80;
listen [::]:80 default_server ipv6only=on;
```

PS:`default_server ipv6only=on`仅需在一个站点的配置文件内添加。

### 检测

- 检测配置是否正确：`nginx -t`
- 检测监听是否成功：`nginx -tuln`
