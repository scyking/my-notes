# VMware

标签（空格分隔）： 计算机网络

---

[toc]

## 对比

> 仅对比在默认配置下的情况。

|对比项|Bridged|NAT|Host-Only|
|---|---|---|---|
|虚拟机是否可以联网|是|是|否|
|其他物理机是否可以访问虚拟机|是|否|否|
|虚拟机是否可以访问其他物理机|是|是|否|
|虚拟机是否占用网络IP|是|否|否|

## Bridged（桥接模式）

> 将主机网卡与虚拟机的网卡利用虚拟网桥进行通信。
> 在桥接的作用下，类似于把物理主机虚拟为一个交换机，所有桥接设置的虚拟机连接到这个交换机的一个接口上，物理主机也同样插在这个交换机当中，所以所有桥接下的网卡与网卡都是交换模式的，相互可以访问而不干扰。
> 在桥接模式下，虚拟机ip地址需要与主机在同一个网段，如果需要联网，则网关与DNS需要与主机网卡一致。

![桥接模式](https://img-blog.csdnimg.cn/20181123230954320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nX3hpbnhpdQ==,size_16,color_FFFFFF,t_70)

## NAT（网络地址转换模式）

> 使用NAT模式，就是让虚拟机借助NAT(网络地址转换)功能，通过物理机来访问网络。
> 此模式下，如果物理机可以访问互联网，那么虚拟机也可以。默认情况下 和物理机同一网络中的其它机器不能访问虚拟机，但虚拟机可以访问其它物理机。但可以通过主机的端口转发让局域网中的物理机和虚拟机进行互相通信。

![NAT](https://img-blog.csdnimg.cn/20181123231231450.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nX3hpbnhpdQ==,size_16,color_FFFFFF,t_70)

## Host-Only（仅主机模式）

> 在一些网络环境中，由于安全，调试等原因，可能需要讲虚拟机和真实的物理环境隔离开来，那么此模式是首选。此模式下的所有虚拟机可以相互访问，但和真实的物理网络环境是隔离开的，此模式下的IP信息，是由host-only虚拟网络的DHCP服务器来分配的。

![Host-Only](https://img-blog.csdnimg.cn/20181123231409121.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nX3hpbnhpdQ==,size_16,color_FFFFFF,t_70)
