# docker 用户组

---

[toc]

## 概述

### 创建docker用户组原因

> docker守候进程绑定的是 `Socket`，而不是 `TCP` 端口。
> 这个套接字默认的属主是 `root`，其他是用户可以使用 `sudo` 命令来访问这个套接字文件。因此，docker服务进程都是以 `root` 帐号的身份运行的。
> 为了避免每次运行docker命令的时候都需要输入 `sudo`，可以创建一个docker用户组，并把相应的用户添加到这个分组里面。当docker进程启动的时候，会设置该套接字可以被docker这个分组的用户读写。这样只要是在docker这个组里面的用户就可以直接执行docker命令了。

## 使用docker用户组

```bash
# 查询用户组中是否存在docker组
sudo cat /etc/group | grep docker

# 创建docker分组，并将相应的用户添加到该分组
sudo groupadd -g 999 docker 
# -g 999为组ID，也可以不指定
sudo usermod -aG dockerroot <用户名>
sudo usermod -aG docker <用户名>

# 检查是否创建成功
cat /etc/group

# 重启docker
sudo systemctl restart docker
```

## 问题

### 运行docker命令，提示`get ......dial unix /var/run/docker.sock`权限不够

> 修改`/var/run/docker.sock`权限。

```bash
sudo chmod a+rw /var/run/docker.sock
```
