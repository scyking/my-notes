# linux setcap命令

标签（空格分隔）： linux

---

[toc]

## 概述

> Linux是一种安全操作系统，它给普通用户尽可能低的权限，而把全部的系统权限赋予一个单一的帐户（`root`）。
> 从2.1版开始，内核开发人员在Linux内核中加入了能力（`capability`）的概念。其目标是消除需要执行某些操作的程序对root帐户的依赖。

## 权限说明

> `Capabilities`的主要思想在于分割root用户的特权，即将root的特权分割成不同的能力，每种能力代表一定的特权操作。

|Capabilities|说明|
|---|---|
|`CAP_CHOWN`|修改文件属主的权限
|`CAP_DAC_OVERRIDE`|忽略文件的DAC访问限制
|`CAP_DAC_READ_SEARCH`|忽略文件读及目录搜索的DAC访问限制
|`CAP_FOWNER`|忽略文件属主ID必须和进程用户ID相匹配的限制
|`CAP_FSETID`|允许设置文件的setuid位
|`CAP_KILL`|允许对不属于自己的进程发送信号
|`CAP_SETGID`|允许改变进程的组ID
|`CAP_SETUID`|允许改变进程的用户ID
|`CAP_SETPCAP`|允许向其他进程转移能力以及删除其他进程的能力
|`CAP_LINUX_IMMUTABLE`|允许修改文件的IMMUTABLE和APPEND属性标志
|`CAP_NET_BIND_SERVICE`|允许绑定到小于`1024`的端口
|`CAP_NET_BROADCAST`|允许网络广播和多播访问
|`CAP_NET_ADMIN`|允许执行网络管理任务
|`CAP_NET_RAW`|允许使用原始套接字
|`CAP_IPC_LOCK`|允许锁定共享内存片段
|`CAP_IPC_OWNER`|忽略IPC所有权检查
|`CAP_SYS_MODULE`|允许插入和删除内核模块
|`CAP_SYS_RAWIO`|允许直接访问`/devport`,`/dev/mem`,`/dev/kmem`及原始块设备
|`CAP_SYS_CHROOT`|允许使用chroot()系统调用
|`CAP_SYS_PTRACE`|允许跟踪任何进程
|`CAP_SYS_PACCT`|允许执行进程的BSD式审计
|`CAP_SYS_ADMIN`|允许执行系统管理任务，如加载或卸载文件系统、设置磁盘配额等
|`CAP_SYS_BOOT`|允许重新启动系统
|`CAP_SYS_NICE`|允许提升优先级及设置其他进程的优先级
|`CAP_SYS_RESOURCE`|忽略资源限制
|`CAP_SYS_TIME`|允许改变系统时钟
|`CAP_SYS_TTY_CONFIG`|允许配置TTY设备
|`CAP_MKNOD`|允许使用`mknod()`系统调用
|`CAP_LEASE`|允许修改文件锁的`FL_LEASE`标志

### 进程能力集

- `cap_effective`：`pE`，表示进程当前可用的能力集。
- `cap_inheritable`：`pI`，表示进程可以传递给其子进程的能力集。
- `cap_permitted`：`pP`，表示进程所拥有的最大能力集。

### 可执行文件能力集

- `cap_effective`：`fE`，表示文件开始运行时可以使用的能力。
- `cap_allowed`：`fI`，表示程序运行时可从原进程的`cap_inheritable`中集成的能力集。
- `cap_forced`：`fP`，表示运行文件时必须拥有才能完成其服务的能力集。

## 使用举例

```bash
usage: setcap [-q] [-v] (-r|-|<caps>) <filename> [ ... (-r|-|<capsN>) <filenameN> ]

 Note <filename> must be a regular (non-symlink) file.
```

### 允许绑定1024以内端口

```bash
setcap cap_net_bind_service=+eip /home/nginx/sbin/nginx
```

### 清除附加权限

```bash
setcap -r nginx
```
