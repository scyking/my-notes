# docker 网络介绍

标签（空格分隔）： docker

---

[toc]
> [docker network](https://docs.docker.com/engine/reference/commandline/network/)

## docker 网络核心
> docker 网络的核心使用的是Linux的虚拟化技术，在容器内和`docker0`内分别建立一个虚拟网卡，再使用`evth-pair`技术进行连接。

### docker0
> docker使用的是Linux的桥接，在宿主机中是一个docker容器的网桥docker0。
> 每启动一个docker容器，docker就会给docker容器分配一个ip，只要安装了docker，就会有一个网卡`docker0`。

### `evth-pair`技术
> docker使用的是桥接模式，使用的技术是`evth-pair`技术。
> `evth-pair` 是一对的虚拟设备接口，它们都是成对出现的。
> `evth-pair`充当一个桥梁，可以连接各种虚拟网络设备。一端连接着协议，一端彼此相连，因此可以彼此通信。

![示例](https://img-blog.csdnimg.cn/2020101421471461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTg1ODAw,size_16,color_FFFFFF,t_70#pic_center)

## 自定义网络

### 查看docker网络列表
> 使用 `docker network ls` 命令。

```
[root@localhost ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
b47982b38d80        bridge              bridge              local
28a1b197470b        host                host                local
1a3c604475c8        none                null                local
```

- `bridge`：桥接，即docker0（默认）
- `none`：不配置网络
- `host`：和宿主机共享网络

### 创建docker网络
> 使用 `docker network create` 命令。

```
[root@localhost ~]# docker network create --help

Usage:  docker network create [OPTIONS] NETWORK

Create a network

Options:
      --attachable           Enable manual container attachment
      --aux-address map      Auxiliary IPv4 or IPv6 addresses used by Network driver (default map[])
      --config-from string   The network from which to copy the configuration
      --config-only          Create a configuration only network
  -d, --driver string        Driver to manage the Network (default "bridge")
      --gateway strings      IPv4 or IPv6 Gateway for the master subnet
      --ingress              Create swarm routing-mesh network
      --internal             Restrict external access to the network
      --ip-range strings     Allocate container ip from a sub-range
      --ipam-driver string   IP Address Management Driver (default "default")
      --ipam-opt map         Set IPAM driver specific options (default map[])
      --ipv6                 Enable IPv6 networking
      --label list           Set metadata on a network
  -o, --opt map              Set driver specific options (default map[])
      --scope string         Control the network's scope
      --subnet strings       Subnet in CIDR format that represents a network segment
```

```
## 创建
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
```

### 查看自定义网络
> 使用 `docker network inspect` 命令。

```
[root@localhost ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "c67a854474e656283c7632d0a988142f602c2c53e47cd4a83e507998aa3c42be",
        "Created": "2021-07-01T15:57:56.438714839+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

### 启动容器指定网络
> 在启动命令`docker run`中，使用`--network`参数指定连接网络。（默认连接`bridge`网络）

### 网络连通
> 使用`docker network connect`命令，将容器连接到指定网络中。

```
[root@localhost ~]# docker network connect --help

Usage:  docker network connect [OPTIONS] NETWORK CONTAINER

Connect a container to a network

Options:
      --alias strings           Add network-scoped alias for the container
      --driver-opt strings      driver options for the network
      --ip string               IPv4 address (e.g., 172.30.100.104)
      --ip6 string              IPv6 address (e.g., 2001:db8::33)
      --link list               Add link to another container
      --link-local-ip strings   Add a link-local address for the container
```

- `NETWORK` 网络名
- `CONTAINER` 容器名