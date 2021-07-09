# docker 修改镜像源

---

[toc]

## 国内镜像仓库

|仓库|链接|
|---|---|
|Docker 官方中国区|`https://registry.docker-cn.com`|
|网易|`http://hub-mirror.c.163.com`|
|中国科学技术大学|`https://docker.mirrors.ustc.edu.cn`|
|阿里云|`https://<id>.mirror.aliyuncs.com`|

## 添加镜像源

1. 编辑配置文件

    ```bash
    vi /etc/docker/daemon.json
    ```

1. 添加如下配置

    ```json
    {
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
    }
    ```

1. 生效配置

    ```bash
    # 重载配置
    sudo systemctl daemon-reload
    # 重启 docker
    sudo service docker restart
    ```
